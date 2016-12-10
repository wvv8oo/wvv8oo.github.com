
<!--
title: Silky的命令
date: 2018
-->

## 本文目标

* 全面介绍Silky的命令

## 如何使用

* Windows系统中，推荐使用`git bash`或者`power shell`
* *nix系统中，打开终端(Terminal)即可输入命令

## 命令介绍

### silky start

用于启动Silky，如果当前项目不是一个Silky项目，那么启动一个简易的http服务器。 

* `-e`或者`--environment` 用于指定运行环境，默认环境为`development`。例如`silky start -e production`的运行环境为`production`，关于Silky的运行环境，请参考：[Silky的运行环境介绍](running-environment-of-silky.html)
* `-p`或者`--port` 用于指定http服务器的端口，默认端口为`14422`，例如`silky start -p 8000`可以指定silky的端口为`8000`，之后就可以用`http://localhost:8000`访问你的项目了
* `-l`或者`--language`，指定语言，默认为`en`，例如`silky start -l cn`可以指定运行语言为中文。关于多语言，请参考：[在项目中使用多国语言](how-to-use-mutil-language-with-silky.html)
* `-s`或者`--sample`，启动示例项目

### silky build

编译Silky项目，也可以用于编译非Silky项目。

* `-o`或者`--output`，指定输出目录，默认情况下会输出到`./build`
* `-e`或者`--environment`，指定运行环境，build的默认环境为`production`
* `-f`或者`--force`，在非Silky项目中强行编译，如果你要编译的目录是一个非Silky项目，那么需要指定`-f`参数
* `-x`或者`--extra`，扩展参数，此参数提供给插件用，根据具体的插件而定
* `-c`或者`--config`，指定配置文件，默认情况下，配置文件文件为`.silky/config.js`，如果你想单独指定一个配置文件，可以使用此参数

### silky init [boilerplate]

在当前目录初始化silky项目，这将会在当前目录创建一个`.silky`的文件夹，并且会复制默认的配置文件到此目录，但不会删除当前目录的任何文件。如果你不指定boilerplate，使用`silky init`会创建一个默认的项目，你也可以从官方样板库指定一个样板项目，或者使用自己的私有样板库。

#### 从官方样板库中创建项目

Silky官方的样板项目库地址在 [https://github.com/wvv8oo/silky-boilerplate](这里)，官方样板项目库会提供一些常见的项目示例，在官方样板项目库中，每一个文件夹都是一个示例项目。例如你可以通过`silky init default`来安装默认的样板项目。

在官方样板库中，根目录中的每一个文件夹，都是一个样板项目，你只需要通过`sily init folder_name`即可安装相应的样板项目。同样，自己创建私有的样板项目也基于同样的规则。

#### 使用私有样板库

某些情况下，你可能会希望使用私有的样板项目，可以通过如下步骤：

1. 创建一个git的样板仓库，你可以把它放到github，也可以是私有的gitlab，甚至是基于文件系统的本地git。现在我假定你的私用仓库地址是`git@your-git-repos.com:examples/silky-boilerplate.git`
2. 通过命令指定私有的样板库地址，命令：`silky config set boilerplateRepository git@your-git-repos.com:examples/silky-boilerplate.git -g`，注意把示例中的地址换成你真实的仓库地址
3. 假定你创建了一个样板项目，名为my_boilerplate，然后`silky init my_boilerplate`

#### 参数

* `-f`或者`--full`，复制全部的示例项目
* `-p`或者`--plugin`，创建一个插件的示例项目

### silky config

读写silky的配置，包括全局配置和项目配置。

#### 参数

* `-g`或者`--global`，全局设置

* `silky config set [key] [value]`，设置某个键
* `silky config remove [key]`，移除某个键
* `silky config get [key]`，读取某个键

### silky install [plugin...]

安装Silky插件，可以指定一个插件名，也可以指定多个插件，插件名之间用空格隔开。如`silky install sample`，或者`silky install sample blog`
注意此功能需要访问`github.com`，由于众所周知的原因，github.com在某些时候可能会无法访问，所以，你需要保持github.com的畅通。

### silky uninstall [plugin...]

删除Silky插件，可以指定一个插件名，也可以指定多个插件，插件名之间用空格隔开。如`silky uninstall sample`，或者`silky uninstall sample blog`


### silky list

列出已经安装的silky插件