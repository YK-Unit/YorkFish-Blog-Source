---
title: 手Y工程实践之Podfile.lock管理新方案
date: 2021-03-07 10:10:01
categories: ["技术"]
tags: ["2021", "iOS", "工程实践", "教程"]
comments: true
---

## 背景

在iOS工程接入`CocoaPods`做依赖管理后，开发者对`Podfile.lock`的管理主要有以下两种方案：

1. 不把`Podfile.lock`纳入版本管理
2. 把`Podfile.lock`纳入版本管理

这两种方案各有优劣和各有适用场景：

1. 第1种方案不强制团队成员使用统一版本的`CocoaPods`，对团队成员较为友好，但是无法保证团队成员在本地安装的依赖是一致的；其适合个人或者前期对规范不作要求、规模很小的团队
2. 第2种方案能保证团队成员在本地安装的依赖是一致的，但是却要强制团队成员使用统一版本的`CocoaPods`；其适合对规范有要求的团队

在我加入手Y团队之前，手Y团队选用的是第1种方案。基于第1种方案的选择，为了能保证团队成员在本地安装的依赖是一致的，手Y团队又做了以下的解决措施：

> Podfile里的每个库都声明一个具名的固定版本号，如`pod 'yyabtestsdk', '2.1.0-dev.2'`、`pod 'yybaseapisdk', :git=>'https://xxx/yybaseapisdk-ios.git', :tag => '7.46.0-dev.8'`。

随着团队的变大（现在iOS业务端已有40+人），这种方案的弊端逐渐暴露：

- 无法百分百确保编译运行阶段，团队成员的本地安装的依赖是一致的

  这个弊端对应的场景是：有人更新了`Podfile`，安装了`非BreakingChanges`的、新版本的依赖库，并进行了代码推送；其他人拉取代码后，若不手动执行一次`Pod install/update`，其本地安装的依赖是落后的，并且在编译运行阶段，由于代码兼容，若非出现严重bug，其不会发现其本地依赖需要更新。

- 因本地依赖的版本不正确导致编译失败的时机延迟发生到了编译中后期

  这个弊端对应的场景是：有人更新了`Podfile`，安装了`BreakingChanges`的、新版本的依赖库，并进行了代码推送；其他人拉取代码后，若不手动执行一次`Pod install/update`，其本地安装的依赖是落后的，然后其进行编译时，由于代码不兼容，会发现编译失败了——但是这时候编译失败的时机常常是发生在编译中后期——在让开发者至少等待了十几分钟后才抛出编译失败的错误——这相当影响开发者的心情和工作效率。

	> 对于`BreakingChanges`的疑问可看下文的《Q&A》

为了解决这些弊端，有必要考虑重新把`Podfile.lock`纳入版本管理。那有没有一种方案，能同时获得上述第1种方案和第2种方案的收益呢？具体是，希望有一种方案能满足以下的需求：

1. 保证团队成员本地安装的依赖是一致的
2. 允许团队成员不使用统一版本的`CocoaPods`
3. 把因本地依赖的版本不正确导致编译失败的时机从编译中后期提前到编译前，帮助提升开发效率
4. 新方案对当前团队成员是零负担（不强制团队成员做不必要的事、不耗费团队成员不必要的注意力和精力）

为此，我制定了一个新的`Podfile.lock`管理方案，下面将会做具体的介绍。

<!-- more -->

## 手Y工程的Podfile.lock管理新方案

`Podfile.lock`管理新方案借助`Ruby`强大的`Method Swizzling`能力实现，整体技术方案如下：

1. hook `CocoaPods`的创建`Podfile.lock`的方法`write_lockfiles`，将其逻辑变更为：创建`Podfile.lock`后，再基于`Podfile.lock`创建一个移除了`CocoaPods`版本信息的副本`Podfile.lock.dump`；然后人为把`Podfile.lock.dump`纳入版本管理

	> 此举是为了满足需求1和需求2。对于此举的疑问可看下文的《Q&A》。

2. hook `CocoaPods`的创建`Manifest.lock`检测脚本的方法`add_check_manifest_lock_script_phase`，将其逻辑变更为：基于本地的`Manifest.lock`创建副本`Manifest.lock.dump`，并比较`Manifest.lock.dump`是否和`Podfile.lock.dump`一致；若不一致，就使用`Podfile.lock.dump`更新本地的 `Podfile.lock`，并抛出错误信息和中断编译

	> 此举是为了满足需求3。

3. 在执行`pod install/update`时，完成上述的hook

为了同时满足需求1和需求2，新方案采取的措施是：基于`Podfile.lock`生成一个不记录`CocoaPods`版本信息的副本`Podfile.lock.dump`，并使用`Podfile.lock.dump`代替`Podfile.lock`纳入版本管理。

技术方案实施如下：

1. 在工程根目录创建hook `CocoaPods`的代码`PodilePatch_HookPod.rb`：

```ruby
class Pod::Installer
  
  def dump_podfile_lock()
    puts "dump Podfile.lock now ..."

    system 'cp Podfile.lock Podfile.lock.dump && sed -i "" "/^COCOAPODS:/"d Podfile.lock.dump'
    
    puts "dump Podfile.lock at path(./Podfile.lock.dump) done!!!"
  end

  # 将来若团队的CocoaPods都升级到v1.10+后，可在新特性 post_integrate hook中执行 dump_podfile_lock
  # modify write_lockfiles method
  old_write_lockfiles = instance_method(:write_lockfiles)
  define_method(:write_lockfiles) do
    old_write_lockfiles.bind(self).()
    dump_podfile_lock()
  end

  class UserProjectIntegrator::TargetIntegrator
    # modify add_check_manifest_lock_script_phase method
    define_method(:add_check_manifest_lock_script_phase) do
        phase_name = CHECK_MANIFEST_PHASE_NAME
        native_targets.each do |native_target|
          phase = UserProjectIntegrator::TargetIntegrator.create_or_update_shell_script_build_phase(native_target, BUILD_PHASE_PREFIX + phase_name)
          native_target.build_phases.unshift(phase).uniq! unless native_target.build_phases.first == phase
          phase.shell_script = <<-SH.strip_heredoc
            Podfile_lock_dump_file="${PODS_PODFILE_DIR_PATH}/Podfile.lock.dump"
            if [ ! -f ${Podfile_lock_dump_file} ]; then
                echo "${Podfile_lock_dump_file} not found" >&2
                exit 0
            fi

            Manifest_lock_dump_file="${PODS_ROOT}/Manifest.lock.dump"
            cp "${PODS_ROOT}/Manifest.lock" ${Manifest_lock_dump_file} && sed -i "" "/^COCOAPODS:/"d ${Manifest_lock_dump_file}
            if [ ! -f ${Manifest_lock_dump_file} ]; then
                echo "${Manifest_lock_dump_file} not found" >&2
                exit 0
            fi

            diff ${Podfile_lock_dump_file} ${Manifest_lock_dump_file} > /dev/null
            if [ $? != 0 ] ; then
                # 使用 Podfile.lock.dump 覆盖本地的 Podfile.lock
                cp ${Podfile_lock_dump_file} "${PODS_PODFILE_DIR_PATH}/Podfile.lock" && echo "COCOAPODS: $(pod --version)">> "${PODS_PODFILE_DIR_PATH}/Podfile.lock"

                # 抛出错误，提醒开发者
                echo "error: The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation." >&2
                echo "错误: 本地安装的Pod库依赖非最新。请执行 'pod install' 或者更新你的 CocoaPods 版本。" >&2

                exit 1
            fi
            # This output is used by Xcode 'outputs' to avoid re-running this script phase.
            echo "SUCCESS" > "${SCRIPT_OUTPUT_FILE_0}"
          SH
          phase.input_paths = %w(${PODS_PODFILE_DIR_PATH}/Podfile.lock ${PODS_ROOT}/Manifest.lock)
          phase.output_paths = [target.check_manifest_lock_script_output_file_path]
        end
    end
  end

end
```

2. 在工程的`Podfile`中引入hook的代码：

```ruby
# hook pod
require_relative 'PodilePatch_HookPod'

target 'Demo' do
  pod 'AFNetworking'
end
```

技术方案实施完毕后，按照业界以往把`Podfile.lock`纳入版本管理后的方式做工程开发即可：`pod install` first always。

## Q&A

### 何为`BreakingChanges`版本的依赖库？

`BreakingChanges`版本的依赖库，其代码是不兼容的，其常见的一个特征就是：API不兼容，比如旧版本的API`f(a,b)`在新版本变成了`f(a,b,c)`。

### 为什么不直接把Podfile.lock纳入版本管理，而是使用其副本Pofile.lock.dump？

`Podfile.lock`纳入版本管理后，若团队成员安装的`CocoaPods`版本不一致，那么这个文件会由于记录的`CocoaPods`版本信息不一致，常常引起版本冲突——这导致团队成员每次更新依赖时都可能要解决一次这种冲突，耗费团队成员不必要的精力。

那是否可以在生成`Podfile.lock`时，移除`CocoaPods`版本信息呢？

在表面上，这能满足我们的需求，但是实际上，这样做，会导致在执行`pod install`时，即使本地的依赖已经是最新的，由于`CocoaPods`版本信息缺失，`CocoaPods`都会为重新安装一次所有的依赖，浪费了团队成员的时间。

### 除了上述方案，是否还有其他管理方案吗？

答案是肯定的。这里分享另外两种技术方案。

第1种技术方案：使用[Git Attributes](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E5%B1%9E%E6%80%A7)，自定义`smudge`和`clean`子过滤器，让`git`“忽略”`Podfile.lock`中记录`CocoaPods版本信息`的文本行。具体如下：

1. 工程根目录创建`.gitattributes`，并添加自定义过滤器执行规则：

	```shell
	echo "Podfile.lock filter=ignorePodVersion" > .gitattributes
	```

2. 为`ignorePodVersion`过滤器分别设置`smudge`和`clean`子过滤器：

	```shell
	git config --local filter.ignorePodVersion.smudge 'sed -i "" "/^COCOAPODS:/"d Podfile.lock && echo "COCOAPODS: $(pod --version)" >> Podfile.lock'
	
	git config --local filter.ignorePodVersion.clean 'sed -i "" "/^COCOAPODS:/"d Podfile.lock 
	```

    > - `smudge`子过滤器会在文件被检出时触发
    >  ![ “smudge”过滤器会在文件被检出时触发](https://git-scm.com/book/en/v2/images/smudge.png)
    >
    > - `clean`子过滤器会在文件被暂存时触发
    > 	![“clean”过滤器会在文件被暂存时触发](https://git-scm.com/book/en/v2/images/clean.png)

但是这种方案，会耗费团队成员不必要的注意力：每次`checkout`代码时，`Podfile.lock`都被`git`标记为`modified`，团队成员在终端或者SourceTree提交代码时，由于这个变更不是出自团队成员自身操作导致的，团队成员难免疑惑，继而耗费一些注意力和精力去理解这个事情——而这正是我努力去避免给团队成员的负担之一。

第2种技术方案：使用`bundler`和`Gemfile`。`bundler`相当于`CocoaPods`，`Gemfile`相当于`Podfile`，这2个是`Ruby`开发中用于管理依赖的工具——我们可借助它们帮助统一管理当前项目的`Ruby`工具链版本，从而满足上述的需求1、2、3。具体实施如下：

1. 安装`bundler`，并在工程根目录下创建`Gemfile`：

   ```shell
   sudo gem install bundler
   cd path/to/project_root_dir
   # create Gemfile
   bundle init
   ```

2. 编辑`Gemfile`，并添加项目所需的`Ruby`工具链版本信息：

   ```ruby
   # frozen_string_literal: true
   
   source "https://rubygems.org"
   
   git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }
   
   # 添加所需的 Ruby工具链
   gem 'cocoapods', "1.10.1"
   ```

3. 工程根目录执行`sudo bundle install`，安装项目所需的`Ruby`工具链，并按需把生成的`Gemfile.lock`纳入版本管理

   > 注意：每次更新了所需的`Ruby`工具链版本，都需要执行一次`sudo bundle install`

4. 安装成功后，开发过程中使用`bundle exec pod install/update`代替`pod install/update`为工程安装`pod`依赖

由于需要引入`bundler`和`Gemfile`，以及使用`bundler exec pod install/update`代替`pod install/update`，这斜在我看来不符合我的需求4，故没有采取此方案。

> 注意：一般来说，工具链版本比较稳定，其变更频率远远低于工程代码依赖的版本变化。其他团队在评估引入`bundler`和`Gemfile`给团队成员带来的负担时，应根据自己当前团队的情况（比如人数、技术栈等）进行评估。

