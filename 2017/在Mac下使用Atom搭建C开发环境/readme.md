---
title: 在Mac下使用Atom搭建C开发环境
date: 2017-02-25 12:34:56
categories: ["技术"]
tags: ["2017", "教程"]
comments: true
---

在 Mac 下写 C 的时候，如果程序并不复杂，其实蛮不愿意打开 Xcode 这个庞然大物的。为此，特意找时间解决了这个事情：`Atom` + `Atom 插件` = `C 轻量级 IDE`。

下面将会列出所用到的主要插件：

- [autocomplete-clang](https://atom.io/packages/autocomplete-clang) 
  autocomplete for C/C++/ObjC using clang

- [build](https://atom.io/packages/build)
  Build your current project, directly from Atom

- [linter](https://atom.io/packages/linter)
  A Base Linter with Cow Powers to visualize errors and other kind-of messages, easily

- [linter-clang](https://atom.io/packages/linter-clang)
 A linter plugin for [Linter](https://atom.io/packages/linter) provides an interface to clang

- [script](https://atom.io/packages/script)
  Run code in Atom!
  > 在 Mac 下，按 cmd-i ，该插件就会调用 clang 编译并运行当前的 c 程序

安装完毕后，开始你的 C 旅程吧~

