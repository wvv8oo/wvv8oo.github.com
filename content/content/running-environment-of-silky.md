<!--
title: Silky的多环境与数据引用
-->

## 摘要

Silky支持数据与实现分离，从而实现对多环境的支持，在Silky中，你可以轻松实现在开发环境与产品环境引用不同的数据。例如在开发环境中使用相对链接，在产品环境使用绝对链接。

## 数据文件

Silky的数据文件放在`your_project/.silky/data`文件夹中，一般来说，数据文件会有四个文件夹，分别是`normal`、`development`、`language`、`production`。其中`language`用于存放多国语言的数据文件，关于多国语言请参考[在Silky中如何使用多国语言](/post/how-to-use-mutil-language-with-silky.html)

数据文件默认的数据类型为json，扩展名应该为.json。但因为`json`文件支持的数据格式过于简单，Silky支持引入`js`文件，在js中可以使用`module.exports={}`返回一个对象，这就意味你甚至可以在这里自己写函数。

## 多环境下的数据覆盖

我们以一个典型的Silky项目数据文件结构为例：

    ├── config.js
    └── data
        ├── development
        │   └── global.js
        ├── normal
        │   └── global.js
        │   └── override.less
        │   └── product.json
        ├── production
        │   └── global.js
        └── language
            ├── cn
            │   └── default.json
            └── en
                └── default.json

多数情况下，数据文件应该放在`normal`文件夹中，假如你只有一种工作环境，那么你就不需要`development`和`production`文件夹。当前工作环境所对应文件夹中的数据文件会覆盖掉`normal`中的同名文件中的内容，理解这一点非常重要。在上面目录结构的数据文件中，如果当前的工作环境是`development`，那么`development/global.js`会合并覆盖掉`normal/global.js`中的内容。也就是说，用哪个文件合并覆盖，取决于当前的工作环境。

## 使用例子

假定我们现在除了默认环境`normal`外，还有`production`和`development`两种环境，对应的数据环境分别如下：

默认环境`your_project/.silky/data/normal/global.js`

    module.exports = {
        "title": "Silky",
        "root": "/"
    }

产品环境`your_project/.silky/data/production/global.js`

    module.exports = {
        "root": "http://silky.wvv8oo.com/"
    }

开发环境`your_project/.silky/data/development/global.js`

    module.exports = {
        "root": "http://localhost:14422/"
    }

在`index.hbs`中我们进行引用

	<!DOCTYPE html>
	<html>
	    <head>
            <meta charset="utf-8" />
            <title>Silky Example</title>
            <link rel="stylesheet" href="{{global.root}css/main.css" type="text/css" charset="utf-8" />
        </head>
	    <body>
	    </body>
	</html>

上述例子中，`{{global.root}}`中的`global`表示读取数据文件中的`global.js`文件的`root`键值。而当前工作环境的数据文件会对`normal`中的数据文件进行合并覆盖。如果你当前的工作环境是`development`，那么`{{globa.root}}`得到的值会是`http://localhost:14422/`。如果当前的工作环境是`production`，那么`{{globa.root}}`得到的值会是`http://silky.wvv8oo.com/`

## 在数据文件中使用函数

Handlebars并不支持复杂的表达式，如果你想实现复杂的功能，你可能需要在数据文件中自己写函数了。下面我们看在示例项目中是如果通过函数实现返回一个随机数的：

`examples/.silky/data/normal/global.js`

    module.exports = {
        random: function(){
            return Math.random();
        }
    }

`index.hbs`

	<!DOCTYPE html>
	<html>
	    <head>
            <meta charset="utf-8" />
            <title>Silky Example</title>
        </head>
	    <body>
	        通过函数实现随机数：{{global.random}}
	    </body>
	</html>