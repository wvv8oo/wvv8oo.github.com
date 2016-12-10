<!--
title: Silky插件之markdown-blog
-->

## 前置条件

* 已经安装好silky
* 安装插件`markdown-blog`，安装命令：`silky install markdown-blog` 

## 功能介绍

`markdown-blog`是一个根据markdown文档自动生成静态博客的插件，支持文章索引。一方面，可以将项目文档自动生成可索引的技术文档，另一方面，团队或者成员也可以用`markdown-blog`来写博客。现在Silky的官方博客就是用`markdown-blog`生成的。

## 主题

`markdown-blog`支持主题功能，`markdown-blog`使用了`hyde`作为默认主题，如果你希望使用其它主题，只需要将主题复制到`./themes`就可以了。当然你也可以自己设计开发一款主题，这非常简单容易。

## 新手入门

## 自定义文章属性

`markdown-blog`允许自定义文章的一些属性，如标题、发布时间、作者等。自定义属性的方法是在文章头部加上如下代码，属性与值以键值对应的形式出现。

	<!--
	title: 关于Silky
	publish_date: 2015-01-01 10:10
	link: about.html
	type: post
	status: publish
	excerpt: 关于Silky的介绍
	-->
	
* `title` 文章的标题，如果没有指定，则使用文件名作为标题
* `publish_date` 发布时间，默认的时间格式为：YYYY-MM-dd hh:mm。如果没有设置，则使用文件的最后修改时间作为发布时间
* `link` 文章的链接，如果没有设置，则链接为文件名，例如文件名为`about.md`，那么链接就是`about.html`
* `type` 文章类型，允许为`post`和`page`两种。默认为`post`，`page`不会列入索引
* `status` 文章状态，允许为`publish`和`draft`，`draft`状态不会被发布，默认为`publish`
* `excerpt` 文章摘要

除了上述指定的键值之外，如果你是使用自己的主题，你还可以加入更多的自定义键值。例如你在文章中增加一个自定义的键`my_attr`，那么在`post.hbs`中，你就可以使用`{{$$.plugin.post.my_attr}}`来引用。

## 进阶配置

### dataDir

markdown文章所在的目录，默认为项目的根目录。

### rss

rss配置，示例如下：

	rss: {
	    // 输出多少条
	    limit: 20,
	    // 是否全文输出
	    full: true
	}
	
### theme

指定博客的主题，默认为`hyde`。假如你配置了`theme`为`custom-theme`，那么首先会在项目目录下查找，即检查`themes/custom-theme`是否存在。如果没有找到，则会检查`markdown-blog`自带的主题中是否有`themes/custom-theme`，如果还是没有，则会使用默认主题`hyde`。
	
### blog

配置blog相关的信息
	
	blog: {
	    //博客标题，默认为My Blog
	    title: "Silky官方博客",
	    //副标题
	    description: "前端模块化开发与构建工具",
	    //主机地址，如果不设置，则为当前主机地址
	    host: 'http://silky.wvv8oo.com/',
	    //rss地址
	    feed: 'http://silky.wvv8oo.com/rss.xml',
	    //作者
	    author: 'wvv8o'
	}
	
此处需要注意了，有时候你可能想使用`http://feed.silky.wvv8oo.com`这样的feed地址，这时候你可能需要在`nginx`或者`apache`下做一些处理，因为`markdown-blog`最终生成的是静态网页，无法处理url重写。


## 自定义主题

自定义主题放在当前项目的`themes/theme_name`文件夹下，假如我们要创建一个主题`my_theme`，那么你的主题文件应该放在`your_project/themes/my_theme`目录下。

### 目录结构

主题的目录结构与Silky项目的目录结构类似，但是没有`.silky`目录，一个主题的参考目录如下：

````html
.
├── js
├── css
│   └── module
│   └── main.less
└── template
	├── 404.hbs
	├── home.hbs
	├── index.hbs
	├── module
	│   ├── footer.hbs
	│   ├── head.hbs
	│   └── header.hbs
	├── page.hbs
	└── post.hbs
````

### 文件介绍

下面的模板文件是必需存在的

* `template/404.hbs` 找不到文件的模板
* `template/home.hbs` 主页的模板
* `template/index.hbs` 索引页的模板，注意，index不是首页
* `template/page.hbs` page页的模板
* `template/post.hbs` 文章页的模板
* `template/module` 子模板目录

### 数据引用

* `$$.plugin.blog`，获取配置文件`config.js`中的`blog`键
* `$$.plugin.posts`，获取文章的索引列表，示例：

````html
{{#each $$.plugin.posts}}
	<div class="post">
		<h1><a href="{{url}}">{{title}}</a></h1>
		<span class="post-date">[{{date publish_date 'YYYY-MM-DD'}}]</span>
		<div class="content">{{{plugin_content url}}}</div>
		<p>&hellip;</p>
	</div>
{{/each}}
````

* `$$.plugin.pages`，获取page的索引列表，参考`$$.plugin.posts`
* `$$.plugin.post`，获取文章或者page，仅限于`page.hbs`与`post.hbs`两个模板中可以。
	* `{{date $$.plugin.post.publish_date 'YYYY-MM-DD hh:mm'}}`， 输出发布时间
	* `{{$$.plugin.post.title}}`，文章或者page的标题

### Handlebars指令扩展

* `{{{plugin_markdown $$.plugin.post.content}}}`，将markdown的内容转换为html
* `{{{plugin_content url}}}`，根据文章的url获取文章的内容，并将内容转换为html