<!--
title: Silky插件介绍
-->

## 插件的安装与使用

* 从官方库安装插件，使用`silky install plugin_name`即可安装
* 在你的项目中，需要添加这个插件的引用，用编译器打开项目中的`config.js`，找到`plugins`键，增加你将要使用的插件。常规的配置，只需要在`plugins`增加插件名称键即可，如：`"pluginName": {}`
* 如果你希望使用自定义的项目库，可以修改全局配置中的custom.pluginRepository键，也可以使用命令：`silky config set pluginRepository your_repository_address`

示例：

    "plugins": {
        //在当前项目中，增加markdown-blog这个插件
        "markdown-blog": {
            //如果你希望截入本地插件，则可以在source中指定插件的绝对路径
            //"source": "/path/to/markdown-blog"
            //更多的参数配置，请参考具体插件的要求
        }
    }


## 芒果TV专用插件

### honey-preview

构建项目后，将项目发布到预览服务器

    "plugins": {
        "honey": {
            //可选，如果项目名称与目录名称不一致，可以在这里指定项目名
            "projectName": "your_project_name",
            //可选，指定上传的服务器，也可以使用绝对路径，如http://192.168.1.1:8000
            //注意，配置文件指定的优先级低于命令行指定
            "server": "108"
        }
    }

在命令行也可以指定上传的服务器 `silky build -x 108`

### honey

处理honey的路径与合并的问题，以及其它honey相关的处理，常规配置即可。

### bhf

处理bhf的代码合并，常规配置即可。

## 常用插件列表

### markdown-blog

根据markdown自动生成技术文档或者博客，更多请参考[这里](/post/markdown-blog-for-plugin-of-silky.html)