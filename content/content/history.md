#History
## 0.2.1

* 支持路由转发，可以根据项目实际情况使用实际并不存在的URL
* 支持Crash后自动重启
* 支持配置启否使用livereload，并支持AMD
* 修复在Linux和Windows下会出现因Deep Watch而Crash的问题?
* 支持作为Express中间件的方式被调用
* 修复一些Bug

##0.1.2 2014-05-06
* 增加logo，并修复`silky init -f`没有初始化images和images-demo文件夹的bug
* 修复无法响应静态文件的bug

##0.1.0 2014-05-05
* 增加可运行的示例项目，默认运行即可以查看示例项目
* 路由支持目录式访问
* 全新的路由规则，使用正则统一判断
* 支持`silky init`的方式初始化项目，`silky init -f`可以创建一个完整的示例项目
* 用配置文件判断金鹰网特有的部分处理


##0.0.9 2014-05-04
* 增加了详细的README文件
* 增加了模板的`import`命令，用于替换`partial`

##0.0.8 2014-04-39
* 生成的代码包括HTML做了美化处理，代码会很整齐漂亮
* 增加代理功能
* 修复build的bug
* 修复其它的一些bug


##0.0.7 2014-04-30

* 兼容Windows的
* 全局silky增加了端口
* 增加合并honey.go
* 修复一些bug


##0.0.6 2014-04-28

* 增加handlebars预编译模块
* 扩展loop命令和partial命令

##0.0.5 2014-04-24

* 增加`silky build`命令，用于构建编译项目