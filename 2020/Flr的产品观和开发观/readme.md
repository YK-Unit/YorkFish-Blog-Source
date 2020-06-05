---
title: Flr的产品观和开发观
date: 2020-06-04 12:34:56
categories: ["人文"]
tags: ["2020", "人文"]
comments: true
---


![Flr](readme/flr.png)

## 前言
[Flr](https://blog.yorkfish.me/2020/Flr%EF%BC%9A%E4%B8%80%E4%B8%AA%E5%87%BA%E8%89%B2%E7%9A%84Flutter%E8%B5%84%E6%BA%90%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7/readme/) 是我主导的和组织团队对外开源的第一款作品——一个用于帮助管理Flutter资源的开发工具系列产品，包括： 

   - [flr-as-plugin]( https://github.com/Fly-Mix/flr-as-plugin)：`Flr`的Android Studio插件版本
   - [flr-vscode-extension](https://github.com/Fly-Mix/flr-vscode-extension)：`Flr`的VSCode插件版本
   - [flr-cli](https://github.com/Fly-Mix/flr-cli)：`Flr`的命令行版本

在这里，主要分享一下`Flr`背后的产品观和开发观。

<!-- more -->

## Flr的产品观

`Flr`的产品观主要体现在“产品矩阵策略”和“产品运营策略”这2点上，下面对这2点分别做具体的分享。

#### 产品矩阵策略

`Flr`的产品矩阵组成如下：

   - [flr-as-plugin]( https://github.com/Fly-Mix/flr-as-plugin)：`Flr`的Android Studio插件版本
   - [flr-vscode-extension](https://github.com/Fly-Mix/flr-vscode-extension)：`Flr`的VSCode插件版本
   - [flr-cli](https://github.com/Fly-Mix/flr-cli)：`Flr`的命令行版本

`Flr`的产品矩阵策略主要是2点：

- 构建开发场景支持全面的产品矩阵

   Flutter的开发场景主要有：
   1. 使用Android Studio进行Flutter开发
   2. 使用VSCode进行Flutter开发
   3. 使用其他文本编辑工具（如Emacs）进行Flutter开发
   4. 使用CI工具对Flutter工程进行自动构建

   > Flutter官方推荐的IDE主要有Android Studio和VSCode两种。
   > 
   > 详细可见[《Flutter编辑工具设定》](https://flutter.cn/docs/get-started/editor)

   对应上述产品矩阵：
   1. `flr-as-plugin`用于满足开发场景一
   2. `flr-vscode-extension`用于满足开发场景二
   3. `flr-cli`主要用于满足开发场景三和开发场景四
   
   通过实施这个策略，`Flr`将能获得以下好处：
   - 支持全面，有助全面占领市场（此处的市场特指Flutter资源管理开发工具这个垂直领域市场）
   - 形成宣传亮点，有助宣传推广产品
   - 形成护城河，直接提升后来挑战者的竞争门槛

- 构建具备容灾能力的产品的矩阵

   此处所述的“灾难”，主要是指Android Studio或者VSCode进行了breaking式的版本升级后，插件因为兼容性问题导致运行失败或者运行异常。
   当发生此种“灾难”后，上述产品矩阵中的`flr-cli`可以作为`flr-as-plugin`和`flr-vscode-extension`的“备胎”，帮助开发者正常完成开发任务，顺利过渡到插件新版本的发布。

#### 产品运营策略

`Flr`的产品运营策略主要是2点：

- 突出`Flr`的全平台支持特性，强调支持“Android Studio”、“VSCode”、“命令行”
  
   注意，此处使用“平台”代替上述的“场景”，一是因为`Flr`的目标用户是开发者，相比“场景”，“平台”与“开发者”具有更深的“亲缘性”，简单来说就是“全平台”比“全场景”更容易获得目标用户的理解；二是因为在字眼上，“平台”这个词比“场景”更有“震撼力”，简单来说就是显得“更牛逼”。

- 突出`Flr`的健壮性，强调支持各种异常场景

   不过，这一点主要体现在`Flr`相关产品的特性说明上，而不是在日常宣传上，因为“事实胜于雄辩”——在目标用户头脑埋下这个种子后，等目标用户在事实中进行发现和获得证明即可。
   
## Flr的开发策略

Flr的开发策略主要有以下几点：

- 构建`Flr`核心逻辑项目——[flr-core](https://github.com/Fly-Mix/flr-core)，以保证`flr-as-plugin`、`flr-vscode-extension`、`flr-cli`在核心逻辑上保持一致，在输出产物上保持一致

   > - “输出产物”包括：文件输出、日志输出、交互输出
   > - 现阶段，保证“文件输出”一致是第一要求
   
- 构建测试用例文档（详细见`flr-core`各分支版本下的`CLD/assets/flr-use-case.xmind`），以保证`flr-as-plugin`、`flr-vscode-extension`、`flr-cli`的健壮性
- 以`flr-cli`的“文件输出”作为测试结果基准，以提升`flr-as-plugin`、`flr-vscode-extension`的测试速度和进一步提升这二者的健壮性
- `flr-cli`作为新特性的实验田，在论证新特性的可行性后，再引进到`flr-as-plugin`、`flr-vscode-extension`
  
   > `flr-cli`采用`Ruby`开发，可快速实现新特性，以察看特性效果

