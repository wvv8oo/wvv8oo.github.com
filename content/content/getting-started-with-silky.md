<!--
title: Silky的新手指南
-->

## 依赖条件

* `node.js v0.10` 以上以及`npm`，如果你还没有安装node.js，请参考：[Installing Node.js via package manager](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)

* 如果你使用的是windows系统，建议安装[git-bash](http://www.git-scm.com/downloads)

## 本文目标

1. 学会安装并启动Silky
2. 了解Silky的基本目录结构
3. 学会使用Silky开发简单的项目

## 入门指南

### 安装并创建示例项目

1. `npm install -g silky coffee-script`，安装silky以及全局的coffee-script，*nix下会需要`sudo`权限
2. `cd ~ && mkdir silky-test && cd ~/silky-test` (Windows下可以考虑使用：`cd %HOMEPATH% && mk silky-test && cd %HOMEPATH%/silky-test`)
3. `silky init -f`
4. `silky start`
5. 用浏览器打开`http://localhost:14422/`，即可看到示例项目了

### Silky的目录结构

#### Silky目录结构图

![Silky目录结构](/images/silky-directory-tree.png)

### 目录说明

1. `.silky`：包含所有与Silky相关的文件，并且此目录用于标识一个项目是否为Silky项目
	1. `.silky/data`：包含不同环境的数据文件，默认数据文件放在`normal`文件夹下，数据文件的作用请参考[Silky多环境与数据引用](/post/running-environment-of-silky.html)
	2. `.silky/language`：包含多国语言的数据文件，如果你的项目只有一种语言，可以忽略此目录
	3. `.silky/config.js`：包含所有Silky的配置，是Silky中非常重要的一个文件。关于配置文件，请参考[Silky的配置](/post/configure-of-silky.html)
2. `template`：可选，该文件夹下包含所有的模板文件与HTML文件，默认情况下，Silky的模板采用Handlebars，并要求模板的扩展名为`.hbs`
3. `css`：可选，用于放置Less及css，`css/module`用于放置less的引用文件
4. `js`：可选，用于放置js和coffee文件
5. 其它文件，根据项目实际需要可以随意创建文件夹，如`images`，`docs`等等

### 模板的使用

Silky采用Handlebars作为默认模板，模板扩展名应为`.hbs`，建议将模板文件放在`template`文件夹下。使用模板的目的是为了更好的模块化开发，减少代码量，并能够实现代码重用。

一般来说，你只需要掌握几个模板命令就可以流畅地使用Silky了。

* `import`：导入子模块，例如有一个菜单的模块`template/module/menu.hbs`，那么我们在`index.hbs`中可以用 `{{import "module/menu"}}`进行引用，在引用模块时候，不需要添加扩展名`.hbs`
* `loop`：可以循环某个子模块多次，例如`{{loop "module/cell" 5}}`，可以将`cell`这个模块重复5次


更多关于模板的使用，请参考： [Silky的模板介绍](/post/template-of-silky.html)


##进阶使用


### 兼容已有的项目

某些时候，我们可能已经有一个已经存在的前端项目需要使用Silky，那么我们可以用如下方式实现。

1. `cd your_project_directory`
2. `silky init`可以初始化当前项目为一个Silky项目，且不删除原有的文件。`--force`或者`-f`参数会强制清除当前目录，并复制示例项目到当前项目。


### 任意目录启动Silky

在任意目录使用`silky start`可以启动一个http服务器，等同于`python -m SimpleHTTPServer`的功能。注意：如果目录中包含`.silky`文件夹，那么将会当成是一个Silky的项目

