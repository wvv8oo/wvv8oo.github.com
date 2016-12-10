<!--
title: Silky的模板
-->

## 本文目标

指导用户如何如何使用模板功能

## 综述

模板最主要的目的就是为了代码重用，模块化开发的好处不言而喻。Silky使用Handlebars作为默认的模板引擎，更多的Handlebars相关的文档与介绍，请点击[英文官网](http://handlebarsjs.com/) 或者 [中文教程](http://www.cnblogs.com/iyangyuan/p/3471227.html)

## 新手用户

默认情况下，Silky会创建一个template的文件夹，所以的模板和html文件都应该放在template文件夹中。模板文件应该以`.hbs`为扩展名，需要被引用的模板文件建议放在`template/module`文件夹下。

例如我们现在有一个首页`index.hbs`，一个页头文件`header.hbs`，一个页脚文件`footer.hbs`，那么我们应该将`header.hbs`和`footer.hbs`文件放到`template/module`文件夹下。

在Silky的示例项目中，目录结构如下：

	├── css
	│   ├── main.less
	│   ├── module
	│   │   └── global.less
	│   └── normalize.css
	├── images
	│   └── logo.png
	├── images-demo
	│   └── logo.png
	├── js
	│   ├── main.coffee
	│   └── vendor
	└── template
	    ├── index.hbs
	    └── module
	        ├── cell.hbs
	        ├── footer.hbs
	        ├── head.hbs
	        └── header.hbs


`index.hbs`文件我们可以这样写：

	<!DOCTYPE html>
	<html>
	    {{import "module/head"}}
	    <body>
	        {{import "module/header"}}
	        <!--其它内容-->
	        {{import "module/footer"}}
	    </body>
	</html>

`{{import "module/header"}}`可以引入module文件夹下的header.hbs，当然Silky也支持相对路径，比如说像这样：`{{import "./module/header"}}`

### 模板指令

模板指令也称之为模板表达式，你可以通过调用一些命令来实现某些特殊的功能，例如循环`each`，引用子模板`import`，我们把这些命令称之为指令。一个典型的指令通常是这样的：`{{command 参数1 参数二}}`，其中command为要执行的指令，后面为具体的参数，指令与参数以及参数之前用空格分隔。参数可以是字符，也可能是`.silky/data`下的数据文件。

如果你希望输出HTML内容，那么你需要这样使用：`{{{command '<div>HTML内容</div>'}}}`

### 引用数据

在Silky中，数据和模板是可以分开的，数据被放在`.silky/data`目录下。通常我们将数据放在`.silky/data/normal`文件夹下。假如文件`.silky/data/normal/global.js`的内容如下：

	module.exports = {
	    "title": "Silky",
	    "footer": {
	        "copyright": "Copyright(at)Silky"
	    }
	}
	
那么在`index.hbs`中，我们可以这样引用：

	<!DOCTYPE html>
	<html>
		<title>{{global.title}}</title>
	    <body>
	      
	    </body>
	</html>	
	
最终生成的代码为：

	<!DOCTYPE html>
	<html>
		<title>Silky</title>
	    <body>
	      
	    </body>
	</html>	

当你在模板中使用`{{global.title}}`进行数据引用的时候，Silky会找到`.silky/data/normal/global.js`这个文件，然后将该文件中的`title`节点插入到模板中。

换句话说，如果你再创建一个数据文件`.silky/data/normal/custom.js`，然后在模板中你的引用方式应该是`{{custom.something}}`，如果你不确定引用是否正确，可以使用`{{print custom}}`来打印这个数据。

当然我们还可能会使用到多环境的数据引用，更多请参考：[Silky中的多环境与数据文件的引用](/post/running-environment-of-silky.html)

### js与css文件引用

在Silky中，我们像平时一样引用js/css文件即可，但如果你在项目中使用了coffee或less，引用的文件名同样是`.js`或`.css`，因为Silky会自动处理，如果没有找到`.css`文件，就会查找`.less`文件，对于js的规则也是同样的。

例如有如下引用：

`<link rel="stylesheet" href="/css/main.css" type="text/css" charset="utf-8" />`

Silky会先查找`/css/main.css`文件，如果没有找到此文件，那么Silky会再尝试查找`/css/main.less`并编译输出。

## 进阶用户

### 常用的Handlebars指令

#### #each

循环指令，例如：

数据文件`.silky/data/normal/global.js`

	module.exports = {
	    "projects": [
	    	{project: 'Silky'},
	    	{project: 'charm.js'}
	    ]
	}

模板文件：

	<ul>
		<li class="title">项目列表</li>
		{{#each global.projects}}
			<li>{{project}}</li>
		{{/each}}
	</ul>

输出：

	<ul>
		<li class="title">项目列表</li>
		<li>Silky</li>
		<li>charm.js</li>
	</ul>

#### #if

条件判断语句，例如

	{{#if true}}
		条件为真
	{{/else}}
		条件为假
	{{/if}}

#### unless

只有在条件为false的时候才输出，等同于else，下面两例

使用`if/else`

	{{#if condition}}

	{{/else}}
		条件为假
	{{/if}}

使用`unless`

	{{unless condition}}
		条件为假
	{{/unless}}

#### @index

在使用`#each`进行循环输出的时候，你可以使用`@index`来获取索引


### Silky扩展指令

#### import

导入其它模板，支持相对路径和绝对路径，支持指定数据源，不需要指定`.hbs`扩展名。早前版本的Silky使用了`partial`指令，现在应当使用`import`，`partial`未来将会被抛弃。

使用示例：

* `{{import “module/header”}}`，导入模板，使用绝对路径
* `{{import “../header”}}`，导入模板，使用相对路径
* `{{import “module/header” data}}`，导入模板，第二个参数允许指定数据。

例如我们现在在数据库`globa.js`的文件格式如下：

	module.exports = {
	    "title": "Silky",
	    "footer": {
	        "copyright": "Copyright(at)Silky"
	    }
	}

`index.bhs`文件：

	<!DOCTYPE html>
	<html>
	    {{import "module/head"}}
	    <body>
	        {{import "module/header"}}
	        <!--其它内容，第二个参数中，我们指定数据源为global.footer-->
	        {{import "module/footer" global.footer}}
	    </body>
	</html>		
	
`module/footer.hbs`文件：

	<footer>
	    Copyright &copy; <a href="http://example.com/" target="_blank">{{copyright}}</a>
	</footer>

这样在`footer.hbs`文件中，我们就可以用`{{copyright}}`来引用`global.footer.copyright`了。这样做的目的就是为了代码更简洁。否则当嵌套引用模块的时候，我们有会需要写很冗长的代码才能引用到数据。

** 注意，这里会有陷阱！**

如果指定了数据源，那么作用域将被改变，你在子模板中(上例的`footer.hbs`)所引用的数据源都是你所指定的数据源下（上例的数据源是`global.footer`）。在上面的示例中，如果你想在`header.hbs`中引用`global.title`这个数据，你无法使用`{{global.title}}`进行引用。但Silky提供另一种方法来引用全局数据，使用`_`可以在任何模块引用全局数据，如：`{{import _.global.title}}`。

#### css

引入 css文件，可以实现批量引入，有如下的用法：

* `{{css "main.css"}}` 直接引入css文件，此处引用`main.css`，你也可以在路径参数中添加变量。例如：`{{css "<global.root>/main.css"}}`，此时`<global.root>`表示`.silky/data/normal/global.js`文件`root`节点的值
* `{{css "/css" "file1.css,file2.css"}}` 引用同一个文件夹下的多个css文件，第一个参数同也可以添加`<global.root>`这种方式的变量
* `{{css data}}` 根据配置引入css，这种方式可以扫描某个文件夹下所有的文件并引用，是一种很方便实用的功能

在示例项目中，我们可以看到从数据文件的配置引入css的例子。

`global.js`文件：

	module.exports = {
	    "title": "Silky",
	    "linkCSS": {
	        "baseUrl": "<global.root>/css/",
	        "dir": "/css",
	        "match": /\.css|less$/,
	        path: /(less)$/,
	        to: 'css'
	    }
	}
	
然后我们在`index.hbs`中用`{{css global.linkCSS}}`引用，它会自动扫描`/css`文件夹下所有的less和css并引用。

我们有必要解释一下上述例子中各参数，在上述例子中

* `baseUrl` 根路径
* `dir`	要扫描的文件夹
* `match` 匹配规则，可选。如上述例子中会匹配所有的css和less文件
* `path` 替换规则，可选。如上述例子中会替换less为css
* `to` 将要替换的文字

即：path + to = url.replace(path, to)


#### script

`script`指令与`css`指令的功能参数都是一样的，只不过script指令将会引用`js`文件而非`css`

#### loop

循环一个模板N次，或者根据数据源进行循环模板。

使用示例：

* `{{loop "module/cell" 5}}` 循环cell这个子模板5次
* `{{loop "module/cell" data}}` 根据data来循环cell这个子模板，此时data的数据类型应该是`array`。**注意：此时cell的数据作用域会被改变**

#### print

print指令可以打印数据，一般用于调试使用，例如你可以使用`{{print global}}`来打印出整个global文件的数据。

#### markdown

转换为markdown，注意此时我们需要用三个花括号来显示HTML内容

使用示例：

* `{{{markdown content}}}` 将markdown转换为html，此时content的内容应该是markdown格式

#### date

输出日期，与`now`指令不同的时，`date`允许指定日期。允许指定三个参数：`{{date 数据源 输出的日期格式(可选) 数据源的日期格式(可选)}}`

使用示例：

* `{{date 1423735598048}}` 将输出`2015-02-12`，默认的日期格式为：`YYYY-MM-DD`
* `{{date 1423735598048 "MM-DD"}}` 指定输出的日期格式，将输出`02-12`
* `{{date "2015-02-12" "MM-DD" "YYYY-MM-DD"}}` 指定输出的日期格式，且指定源日期格式，将输出`02-12`。这种情况一般适用于数据源为字符串，且不是规范的日期格式这种情况

#### now

打印出当前时间，如果没有指定日期格式，则会输出当前的时间戳。一般来说，可以用`{{now}}`来给文件引用加入时间戳，例如`<script src="js/main.js?timestamp={{now}}"></script>`

使用示例：

* `{{now}}` 输出时间戳
* `{{now "YYYY-MM-DD hh:mm:ss"}}` 根据指定的格式输出时间，更多时间格式的规则请参数[momentjs](http://momentjs.com/docs/#/parsing/string-format/)的**String + Format**部分

#### substr

截取字符串，例如`{{substr "这是一个长的字符" 2}}`，输出结果为：`这是`

#### ifEqual

逻辑与的判断，例如：

	{{#ifEqual 0 1}}
		两者相等	
	{{/ifEqual}}

#### or

等同于javascript中的`a || b`, 在模板中可以实现同样的功能`{{or a b}}`




