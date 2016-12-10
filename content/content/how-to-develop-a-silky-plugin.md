<!--
title: 如何开发一个Silky插件
-->

## Silky的插件机制

Silky在启动的时候，会根据项目中的配置文件加载插件列表（注意：这些插件必需已经被安装）。Silky提供了很多hook，而插件就是通过这些hook注入到Silky中。
Silky在运行的过程中，会调用对应的Hook，待插件处理完毕后，控制权才会转交到Silky。

## 手把手教你一步一步开发插件

下面让我们来开发一个简单的插件，目标是在所有的HTML文件的最后面加上一个时间戳。首先我们约定你的工作目录为`~/silky-learn`，如果你的操作系统是Windows，则约定你的工作目录在`%HOMEPATH%/silky-learn`，下面以*nix为例，Windows请更改对应的命令。

* `mkdir ~/silky-learn && cd ~/silky-learn && mkdir my-plugin`，创建一个名为`my-plugin`的插件
* `cd ~/silky-learn/my-plugin && silky init --plugin`，创建一个示例插件项目
* `cd ~/silky-learn && mkdir silky-test && cd silky-test && silky init -f`，创建一个Silky工作目录
* 用编辑器打开`.silky/config.js`，找到`plugins`键，增加`my-plugin`插件，这时候你的`config.js`文件plugins的配置看起来像这样：

    plugins: {
        "my-plugin": {
            source: '../my-plugin'
        }
    }

* `cd ~/silky-learn/silky-test && silky start`，切换到Silky测试项目，并启动Silky
* 用浏览器打开`http://localhost:14422`，然后查看源码的最后面，是否已经看到了时间戳

如果过程不顺利，请检查：

1. 你的操作系统是否为windows，如果是，请将`~`改成`%HOMEPATH%`，`mkdir`改成`md`
2. 是否已经安装了Silky的最新版本，你需要安装`0.7.4`或以上的版本

## 插件的构成

打开我们刚刚使用`silky init --plugin`所创建的示例插件，发现只有两个文件，这是一个插件的基本文件。package.json描述插件的名称与依赖等信息，与npm的package.json完全是一致的。更多与package.json相关请参考[NPM官方文档](https://docs.npmjs.com/files/package.json)

`index.js`则是插件的入口文件，如果你习惯用`CoffeeScript`，那么入口文件应该是`index.coffee`。特别提醒的是，不要忘记在`package.json`加上依赖。

### 入口文件index.js

#### exports.silkyPlugin

用于标识这是一个Silky插件，必需强制为`true`，即`exports.silkyPlugin = true`

#### exports.registerPlugin

注册插件的方法，Silky在加载插件的时候，会调用这个方法。`registerPlugin`提供`silky`和`options`两个参数。`options`即是用户在`config.js`文件中`plugins`键的配置信息。

假定你的插件名称为`my-first-plugin`，用户在`config.js`中配置为：

	"plugins": {
		"my-first-plugin": {
			"title": "My Title" 
		}
	}

此时，`options`所得到的对象应该是这样：

		{
			"title": "My Title" 
		}
		
在你的插件`registerPlugin`方法中，使用`options.title`所得到的值为`My Title`
		
示例：

    //注册silky插件
    exports.registerPlugin = function(silky, options) {
        //注册handlebars的helper，关于handlebars，请参考：http://handlebarsjs.com/
        silky.registerHandlebarsHelper('plugin_custom_command', function(value, done) {
            //直接返回value，什么也不做，你可以根据需要返回具体的数据
            return value
        });

        //将要响应路由时hook
        silky.registerHook('route:willResponse', function(data) {
            //如果是html文件，则在最后面加上一个时间戳
            if (/\.html$/.test(data.request.url)) {
                var extendText = "<!--" + new Date() + "-->";
                data.content += extendText;
            }
        });

        //编译完成后的的hook
        silky.registerHook('build:didCompile', {
            //这里声明使用异步，如果async:true，那么必需显式调用done()
            async: true
        }, function(data, done) {
            //这里可以做任何你想做的事，比如说合并文件等
            return done(null);
        });
    };

## silky对象

此处介绍注册插件`exports.registerPlugin(silky, options)`所返回的`silky`对象，silky对象提供很多方法或配置，详细如下：

### silky.registerHook(hookName, options, factory)

注册一个hook，`options`为可选参数，你也可以这样`registerHook(hookName, factory)`使用。这是一个很重要的方法，在插件中，你可以将你的代码注入到Silky中，并影响到Silky的运行。

* `hookName` 将要注册的hook名称，更多hook请参考文档中的hooks一节
* `options` 可选，如果你希望异步的话，需要设置`options.async`为`true`
* `factory` 工厂函数，Silky触发hook时，会调用此工厂函数，插件的代码也在此工厂函数中实现。

示例一：

	//同步运行的hook
	silky.registerHook('route:willResponse', function(data) {
	    //如果是html文件，则在最后面加上一个时间戳
	    if (/\.html$/.test(data.request.url)) {
	        var extendText = "<!--" + new Date() + "-->";
	        data.content += extendText;
	    }
	});

#### factory(data, done)

`silky.registerHook`的工厂函数，`data`根据不同的hook会提供不同的数据，如果你希望异步运行，那么在运行结束后需要调用`done(err)`，否则silky会被阻塞。

### silky.registerHandlebarsHelper(name, factory)

注册一个handlebars的指令，适用于`hbs`的模板，建议指令名称以`plugin_`为前缀，避免与silky本身的指令冲突，例如：

    silky.registerHandlebarsHelper('plugin_custom', function(value, done) {
        //直接返回value，什么也不做，你可以根据需要返回具体的数据
        return value
    });
    
然后在hbs模板中就可以使用了，`{{plugin_custom 'test'}}`，这将会输入`test`。更多Handlebars Helper的信息请参考[官方文档](http://handlebarsjs.com/reference.html)。

### silky.registerCompiler(type, factory)

注册一个编译器，如果你希望支持silky目前还不支持的文件类型，则需要在此注册一个编译器。

* `type` 文件类型
* `factory` 处理的工厂函数，调用方式为`factory(source, options, cb)`，参数具体含义可以参考`silky.compiler.execute`

### silky.config

项目的配置文件，其实本质是`your_project/.silky/config.js`与`~/.silky/config.js`合并之后的结果。

### silky.options

silky的选项，特别注意，`silky.options`并不是插件的`options`。`silky.options`包含的信息如下：

	{ 
		//当前运行环境
		env: 'development',
		//当前工作目录
		workbench: '/Volumes/Files/WorkStation/silky-blog',
		//是否为构建模式
		buildMode: false,
		//silky的版本
		version: '0.7.4',
		//当前指定的语言
		language: 'en',
		//当前指定的端口
		port: '',
		//是否指定了debug模式
		debug: false,
		//标识目录名
		identity: '.silky',
		//全局的.silky目录
		globalSilkyIdentityDir: '/Users/wvv8oo/.silky'
	}


### silky.compiler

处理编译相关的对象，包含三个方法。

#### silky.compiler.detectCompiler(type)

根据文件类型，返回编译器的类型，例如：

	//默认情况下，返回hbs，即html的默认编译器是hbs
	compiler.detectCompiler('html')
	
#### silky.compiler.sourceFile(type, source)

根据编译类型`type`与文件路径，返回真实的文件，例如：

````js
//返回结果为：/path/to/main.less
compiler.sourceFile('less', '/path/to/main.css')
````

#### silky.compiler.execute(type, source, options, cb)

执行编译，其中`options`为可选参数，也可以这样`execute(type, source, cb)`使用

* `type` 编译器的类型，例如`less`、`hbs`、`coffee`等
* `source` 要编译的文件绝对路径
*  `options` 选项，此参数可选。比如说在build的时候，你可能想编译后直接保存文件，那么`options`可以这样设置：

````
	{
		save: true,
		//文件保存的位置 
		target: '/path/to/file.html'			
	}

````

* `cb` 处理完成后的回调函数，此回调函数会返回处理的结果。`cb(err, content)`，`err`：错误信息，`content`：编译后的内容

### silky.detectFileType(path)

根据路径或者url，判断文件的类型，例如

* `silky.detectFileType('http://localhost:14422/')`，返回`dir`
* `silky.detectFileType('http://localhost:14422/index.html')`，返回`html`
* `silky.detectFileType('http://localhost:14422/index.htm')`，返回`html`
* `silky.detectFileType('http://localhost:14422/css/main.css')`，返回`css`

### silky.registerPluginDirectory(relativePath)

注册一个插件的数据目录


## Hooks

Silky提供了一系列hook，通过这些hook，插件可以注入到Silky。

### route相关的hook

当使用`silky start`时可能触发的hook，以`route:`为前缀

#### route:initial

此hook在silky的http服务启动前触发，`silky start`后最先触发此hook。`data`为一个空对象

示例：

	silky.registerHook('route:initial', function(){
		//do something
	})

#### route:didRequest

此hook在silky收到http请求，且处理完routers后触发。此时`factory(data, done)`的`data`会提供如下数据：

* `data.request` http请求的request对象
* `data.response` http请求的response对象
* `data.next` express中的`next`，可以跳到下一个路由
* `data.stop` 阻止Silky的处理流程。默认为false，一般在插件直接接管路由的时候，会将`data.stop`置为`true`
* `route` 路由对象
	* `url` 请求的原始url
	* `rule` 匹配到配置文件中的转发规则
	* `type` 当前请求对应的文件类型，如`css`、`html`、`dir`、`js`等
	* `mime` 当前请求的mime类型
	* `compiler` 使用的编译器名称	
* `data.pluginData` 提供给`hbs`模板的数据，默认为`null`，在模板中获取`pluginData`的数据可以使用`$$.plugin.key`。例如：`data.pluginData = {username: 'wvv8oo'}`，那么在模板中应该这样获取数据：`{{$$.plugin.username}}`
* `data.method`: 当前http请求的类型，一般是`post`、`get`、`delete`、`put`之一

下面我们以一个例子来说明，我们要截获`test.html`这个url，并且向浏览器返回一个字符。

    //将要响应路由时hook
    silky.registerHook('route:willResponse', function(data) {
    	//匹配到test.html这个url
        if (/^\/test\.html$/i.test(data.route.url)) {
        	//阻止Silky继续执行
            data.stop = true
            //直接向浏览器输出自定义字符
            data.response.end('my first plugin')
        }
    });

#### route:willPrepareDirectory

此hook在将要准备目录输出的时候触发，用于展示目录列表给浏览器的时候。一般在此时，我们可以过滤掉不想呈现给浏览器的文件，即过滤`data.files`的文件。
此时`factory(data, done)`的`data`会提供如下数据：

* `data.request` http请求的request对象
* `data.response` http请求的response对象
* `data.next` express中的`next`，可以跳到下一个路由
* `data.stop` 阻止Silky的处理流程
* `data.files` 此目录下的所有文件列表
	* `filename` 文件名
	* `url` 链接地址
* `data.directory` 目录的完整路径

#### route:didPrepareDirectory

此hook在目录列表已经准备好，且经过模板渲染为最终页面后触发。此时`factory(data, done)`的`data`会提供如下数据：

* `data.request` http请求的request对象
* `data.response` http请求的response对象
* `data.next` express中的`next`，可以跳到下一个路由
* `data.content` 将要输出的页面内容
* `data.directory` 目录的完整路径

#### route:willResponse

此hook在数据已经处理好(如果是coffee或者less，则已经编译好)，将要向浏览器响应数据的时候触发。此时`factory(data, done)`的`data`会提供如下数据：

* `data.request` http请求的request对象
* `data.response` http请求的response对象
* `data.next` express中的`next`，可以跳到下一个路由
* `data.content` 将要输出的页面内容
* `data.mime` 当前请求的mime类型
* `data.stop` 阻止Silky的处理流程

### build相关的hook

这些hook在构建的过程中会被触发，即当用户运行`silky build`后可能会被触发，以`build:`为前缀

#### build:initial

此hook在构建开始的时候触发，此时data为一个空对象

示例：

	silky.registerHook('route:initial', function(data){
		//do something
	})

#### build:willBuild

此hook在将要构建的时候触发，一般可以通过这个hook更改输出目录。此时`factory(data, done)`的`data`会提供如下数据：

* `data.output` 构建将要输出的绝对路径

#### build:didBuild

此hook在整个项目构建完成后会被触发，如果你希望运行完`silky build`后，将构建后的项目打包并上传到服务器，可以考虑截获这个hook。


#### build:willMake

此hook在将要编译复制整个项目时被触发

*警告：此hook可能会被抛弃*

#### build:didMake 

此hook在编译复制整个项目后将被触发

*警告：此hook可能会被抛弃*

#### build:willProcess

此hook在将要处理文件或者文件夹的时候被触发，此时要处理的可能是文件也可能是文件夹，可能是复制文件，也可能是要编译文件。此时`factory(data, done)`的`data`会提供如下数据：

* `data.stat` 文件或者对象的`fs.Stats`对象，更多关于`fs.Stats`请参考[Node.js官方API](http://www.nodejs.org/api/fs.html#fs_class_fs_stats)
* `data.source` 将要处理的绝对路径
* `data.ignore` 是否忽略此路径的处理，如果一个文件夹被忽略，那么该文件夹内所有的内容都将被忽略
* `data.copy` 是否仅复制
* `data.relativePath` 相对(于当前工作目录而言)路径
* `data.target` 处理后将要保存的目录

#### build:didProcess

此hook在完成对文件或者文件夹的处理后会被触发

*警告：此hook可能会被抛弃*


#### build:willCompile

此hook在将要编译某个具体文件的时候被触发，与`build:willProcess`不同的是，此时已经到具体的文件编译，文件夹以及复制文件不会触发这个hook。

此时`factory(data, done)`的`data`会提供如下数据：

* `data.source` 将要编译的源文件绝对路径
* `data.target` 编译后保存的绝对路径
* `data.type` 文件类型
* `data.pluginData` 插件提供的附加数据，更多请参考`route:didRequest`对`data.pluginData`的描述

#### build:didCompile

此hook在完成编译后会被触发，此时`factory(data, done)`的`data`会提供如下数据：

* `data.source` 将要编译的源文件绝对路径
* `data.target` 编译后保存的绝对路径

*警告：此hook可能会被抛弃*

#### build:willCompress

此hook在将要压缩的时候会被触发，此时`factory(data, done)`的`data`会提供如下数据：

* `data.stat` 将要压缩文件的`fs.Stats`对象，更多关于`fs.Stats`请参考[Node.js官方API](http://www.nodejs.org/api/fs.html#fs_class_fs_stats)
* `data.path` 将要压缩文件的绝对路径
* `data.relativePath` 文件的相对路径
* `data.ignore` 是否忽略此文件

示例：

	silky.registerHook('build:willCompress', function(data){
		//不压缩文件名带有min.js的文件
		data.ignore = /.+min\.js$/i.test(data.relativePath)
	})

#### build:didCompress

此hook在压缩完成后会被触发。

*警告：此hook可能会被抛弃*


## 如何发布我的插件

一般情况下，如果是自用插件，只需要在配置插件的时候设置`source`为本地目录即可，但如果你希望将插件共享给其他使用，那么你可以按如下步骤进行操作：

1. fork官方插件项目 `https://github.com/wvv8oo/silky-plugins`
2. 添加你的插件
3. 向官方插件库提交pull request

注意：官方插件仅收录通用且优秀的插件