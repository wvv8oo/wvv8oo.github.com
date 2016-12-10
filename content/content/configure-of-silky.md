
<!--
title: Silky的配置
-->

## 简述

Silky的配置文件分为两个部分，第一个是项目配置文件，文件位置是`your_project/.silky/config.js`；另一个是全局的配置文件，在*nix下是`~/.silky/config.js`，而在Windows下则是`%HOMEPATH%/.silky/config.js`。Silky的配置文件实际上是一个JSON文件。

## 项目配置文件

### version

Silky配置文件的版本，通常不要去修改，除非你非常了解它的用处

### port

指定当前项目的http监听端口，有三种方式可以指定监听端口，即通过命令行`silky start -p 14422`，通过配置文件的`port`指定，使用默认端口。其优先级分别为**命令行指定端口>配置文件指定>默认端口**

注意：在*nix环境下，指定80端口需要sudo权限

### proxy

代理转发的配置，通过代理，我们可以很容易就解决前端的大麻烦跨域问题。Silky使用了[http-proxy](https://github.com/steve-jansen/json-proxy)来处理代理，更多的配置可以参考http-proxy的配置要求。

例如：

```
proxy:{
	forward: {
		"/api": "http://127.0.0.1:8001/api"
	}
}
```

上面这个配置，将会把所有`/api`下的请求都代理到`http://127.0.0.1:8001/api`去

### routers

前端开发的另一个大问题，就是与后端最终的目录结构可能不一致，一个非常典型的情况就是，后端可以通过`http://example.com`访问`index.html`页，而在前端开发中，必需指定明确的文件名`index.html`。通过路由的功能，我们就可以完全模拟后端的目录结构。

在Angular等单页应用的项目中，我们通常会使用HTML5的pushState来模拟URL。在传统的前端开发中，我们可能一刷新页面就会出现找不到页面的情况，因为模拟的url在后端实际是并不存在的。

`routers`节点可以接受一个数组，如果你想把根目录转到index.html，那么你的配置应该是：

	'routers': [
		 //如果你希望以目录的形式访问，则需要删除此条路由
		 {
		 	//匹配根目录
		    path: /^\/$/, 
		    //转发到index.html
		    to: 'index.html', 
		    //不再响应下一个路由
		    next: false
		 }
	]

在angular项目中，我们可能需要将所有没有扩展名的url都转到`main.html`，那么我们的配置如下：

	'routers': [
		 {
		 	//匹配所有没有扩展名的url
		    'path': /.*(\/[^\.]+(\/)?)$/i,
		    //全部转发到main.html
		    'to': "/main.html",
		    'static': false,
		    'next': false
		 }
	]
	
* `path`：表示将要匹配路由的正则表达式
* `to`：要替换的url，即`url.replace(path, to)`，所以你可以使用`$1`这样的正则捕获组。
* `static`：是否为静态文件，如果是静态文件则会直接返回，将不会做任何编译处理
* `next`：如果已经捕获成功，是否继续匹配后面的路由。如果`next`为true，那么将会捕获最后一条匹配的路由，反之则会捕获最先匹配的路由。

### plugins

`plugins`节点可以配置插件，如果你想使用`blog`插件，那么你的配置应该是：

    plugins: {
        "blog": {}
    }
    
默认情况下，可以使用`"pluginName": {}`即可，如果插件要求要提供其它配置，可以在这里配置插件，具体的参数请参考插件的要求。

某些时候，你可能会使用一些私有的插件，那么此时，你可以利用`source`来指定插件的本地路径，source允许使用绝对路径与相对(相对于当前Silky项目)路径。

例如：

    plugins: {
        "blog": {
        	//强制指定本地的插件源
        	"source": "/path/to/silky-plugins/blog",
        	//禁止插件运行，默认为true
        	enable: false
        }
    }

此时，`blog`插件查找的路径将是`/path/to/silky-plugins/blog`，注意：你必需保证插件所依赖的npm包已经安装。

### build

构建编译项目的配置

#### output

配置编译时的输出目录，默认为`./build`，也可以通过命令行的`-o`参数指定，如：`silky build -o ~/temp/my-project`。优先级为：命令行指定>config.js>默认配置

#### copy

构建时将要复制哪些文件或者目录，可接受的参数为一个数组，数组的值可以是`string`和`RegExp`，如果是`string`则要求完全匹配，否则需要正则匹配。

示例：

````js
build: {
	//复制images-demo目录，复制所有以min.js结尾的文件
	copy: ["images-demo", /^.+min\.js$/i]
}
````

#### ignore

经常有一些辅助性的文件，我们并不想复制到最终的构建目标中去，这时候我们可以通过`ignore`配置要忽略的文件列表。`ignore`可接受的参数为一个数组，数组的值可以是`string`和`RegExp`，如果是`string`则要求完全匹配，否则需要正则匹配。

示例：

````js
build: {
	//忽略template/module下的所有文件，忽略css/module下的所有文件，忽略所有以点开头的文件(在*nix中表示隐藏文件)
	ignore: [/^template\/module$/i, /^css\/module$/i, /^\..+/]
}
````
#### rename

编译后对文件进行重命名，例如在开发的时候，我们的文件可都是以`*.source.js`这种方式命名，在编译后，我们希望命名为`*.js`，那么我们就可以通过`rename`节点进行配置。

示例：

````js
build:{
	rename: [
		{
			source: /source\.(js)$/i, target: '$1', next: false
		},
		{
			//将替换掉template，通常在开发的时候，我们会将所有的模板放到template下，但在产品环境中，我们会把html文件放到根目录下
			source: /^template\/(.+)/i, target: '$1', next: false
		}
	]
}
````

#### compress

配置将要压缩哪些类型的文件，如果你希望具体的某个文件不要压缩，请使用`ignore`

示例：

	build: {
        compress: {
            //将要忽略压缩的文件，支持RegExp和string
            ignore: [],
            //压缩js，包括coffee
            js: true,
            //压缩css，包括less
            css: true,
            //压缩html
            html: false,
            //是否压缩internal的js
            internal: true
        }
    }

## 全局配置文件

全局配置的目的，是为了解决很多项目都存在同一配置的情况。全局配置的文件位置为`$HOME/.silky/config.js`。

示例：

	module.exports = {
	  "lastCheckUpdate": 1423639202173,
	  "custom": {
	  	//强制指定插件目录
	    "globalPluginDirectory": "/Volumes/Files/WorkStation/silky-plugins"
	  }
	}
	
其中`custom`节点中的内容为全局配置，项目配置会覆盖同名的全局配置。