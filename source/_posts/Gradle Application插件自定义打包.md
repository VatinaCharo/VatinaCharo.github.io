---
title: Gradle Application插件自定义打包
tags: [Gradle]
categories: [Code]
category_bar: [Code]
date: 2023-05-17 16:45:16
typora-root-url: ..
---

# Gradle Application插件自定义打包

## 目标需求

1. 启动脚本中添加指定内容
2. zip中添加指定文件

## 实现

### 启动脚本中添加指定内容

在PTAssist项目的开发中，我们基于kotlin语言开发javafx程序，并通过gradle来进行依赖管理，最终我们使用Application插件的distZip进行项目打包，这会自动生成一个目录结构如下所示的zip文件

```text
xxx.zip
    |- xxx
        |- bin
            |- xxx
            |- xxx.bat
        |- lib
            |- aaa.jar
            ...
            |- nnn.jar
```

其中 `bin` 目录下是对应的启动脚本， `lib` 目录下是程序jar包

插件默认提供的启动脚本中没有pause命令，所以如果程序出现什么无法启动的错误，就会出现cmd一闪而过的问题，很难排查错误，也不利于用户使用

于是我们需要在打包的过程中，往启动脚本中注入一行pause

我们发现，可以通过修改application插件提供的startScripts的Task来手动注入一行pause

具体代码如下：

```Groovy
tasks.startScripts {
    doLast {
        def windowsScriptFile = getWindowsScript()
        windowsScriptFile.text = windowsScriptFile.text.replace("\n:end", "pause\n\n:end")
    }
}
```

### zip中添加指定文件

我们的项目需要在打包时加入一个README.md作为用户的使用说明文件

我们这里采用的是修改distribution插件的配置，因为application插件会自动应用此插件，所以其实可以通过distribution插件的配置来控制application插件的打包内容

由于默认使用的是 `Groovy` 的gradle配置文件，所以可以通过 `Groovy` 脚本语言提供的功能来自定义的添加需要被打包进入zip的文件或者文件夹

```Groovy
// 打包时加入README.md

distributions {
    main {
        contents {
            from parent.file('README_Client.md')
            parent.file("assets").listFiles().each {
                from(it) {
                    into "assets"
                }
            }
        }
    }
}
```

这里通过 `parent` 获取到了模块对应的项目根目录，然后 `assets` 存放的是readme文件所需的图片，所以也需要一并打包进去。

做完之后，个人觉得，如果最开始采用的是基于kotlin语言的 `bulid.gradle` ，那么写这些配置可能会简单一些，而且应该会有比较好的补全。（idea默认不会补全build.gradle里面的groovy的语法，有点蛋疼）
