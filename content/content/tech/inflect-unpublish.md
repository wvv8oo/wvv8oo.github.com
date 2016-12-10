<!--
title: npm包inflect引发大面积的NPM错误
-->

> 广告时间：如果你是一位前端工程师，你一定要尝试一下[Silky](http://silky.wvv8oo.com/)，它会让你的前端开发更加酷。

今天在升级Silky的时候，发现在所有的服务器都无法正常升级，尝试升级npm至2.7.4后，安装silky提示错误如下：

````
npm ERR! Linux 2.6.32-358.el6.x86_64
npm ERR! argv "node" "/root/.nvm/v0.11.13/bin/npm" "install" "silky" "-g"
npm ERR! node v0.11.13
npm ERR! npm  v2.7.5

npm ERR! version not found: i@0.3.2
npm ERR!
npm ERR! If you need help, you may report this error at:
npm ERR!     <https://github.com/npm/npm/issues>

npm ERR! Please include the following file with any support request:
npm ERR!     /root/npm-debug.log
````

经过多方排查，发现是因为`i@0.3.2`在服务器上无法找到，原因是原作者不小心在NPM服务器上删除了0.3.2版本，而根据NPM的规定，已经被unpublish的package无法被再发布，大约3小时前，作者在github的仓库上发出声明：

> NOTE: 0.3.2 was accidentally unpublished from the server and npm doesn't allow me to publish it back. Please upgrade to 0.3.3

> 提示：0.3.2版本被意外删除，但npm无法回滚，请更新至0.3.3

## 影响

因为`i`被大量的npm包引用，根据官方的最新数据，当天的下载量为56,659 ，最近一个月的下载量更是高达1,188,607，大量的第三方npm包引用了i并指定了版本为0.3.2，而0.3.2版本在服务器已经被删除，预计未来几天将会有大量的npm包将无法被正常安装。

## 反思

npm目前的机制是一旦被unpublish，该版本就被禁用，并且npm存在着海量深层引用的问题，想要查找问题并修复并不是一件很容易的事，特别是一个知名的npm包出问题之后，所带来的影响将是严重的。

## 解决方案

1. npm官方恢复0.3.2版本
2. taobao等npm镜像恢复0.3.2版本
3. 自己搭建私有镜像
4. 其它解决方案，我还在处理这个事，有解决方案第一时间放出