# easyError
一些易错的点

在Chrome和Opera浏览器下，使用CSS3的transform: translate(0, 0)转化位置节点，其所有使用position:fixed定位的子孙节点的定位功能均无效。


目录组织
模块依赖 module 目录下，有 moduleA 目录

[
    'pro/module/regularModule',
    'base/util',
    'text!./index.html',
     ...
    'pro/service/xxx/api',
     ...
    'pro/components/moduleA/subTab1/index',
    'pro/components/moduleA/subTab2/index',
]
每个模块都有其依赖的即远程接口，在 pro/service/xxx/api 文件里进行的声明，而且这个 api 应该还是这个模块的强依赖，别的模块极少依赖，除非业务逻辑有交叉，现在这样组织的话，相当于拆出了模块的一部分。

接着是 pro/components/moduleA， 按照文件组织方式来说，是属于 moduleA 的组件，但是 pro/components/ 给人的理解是一种公用的模块，在其中看到了一些明显属于公共的部分。但实际搜索了下，发现组件仅仅会被 moduleA 依赖。如果仅仅被 moduleA 依赖，说明不具有通用性。文件按目前这样放置的话，相当于又拆出了模块的一部分。

如果有新的需求，需要跑到 pro/components/ 目录下建立新文件，在 pro/service/ 下面声明新的接口调用，如果全把这些文件放入到 moduleA 下面，这些工作都可以在同一个目录下完成，查看的时候也会很方便，在模块有依赖 service 的情况下，也会很轻松找到。

toFixed 的使用，这个一般用来处理金额，可能需要慎重一些
举个例子：859.385.toFixed(2) // => 859.38

一般解法
function toFixed (num, precision) {
  return +(Math.round(num * Math.power(10, precision) / Math.power(10, precision))).toFixed(precision);
}
但是 Math.round( 35.855 * 100 ) / 100 // => 35.85 http://jsfiddle.net/cCX5y/2/

特殊解法
function toFixed( num, precision ) {
    return (+(Math.round(+(num + 'e' + precision)) + 'e' + -precision)).toFixed(precision);
}
lodash https://mdn.io/round#Examples
function createRound(methodName) {
  const func = Math[methodName]
  return (number, precision) => {
    number = toNumber(number)
    precision = precision == null ? 0 : Math.min(toInteger(precision), 292)
    if (precision) {
      // Shift with exponential notation to avoid floating-point issues.
      // See [MDN](https://mdn.io/round#Examples) for more details.
      let pair = `${ toString(number) }e`.split('e')
      const value = func(`${ pair[0] }e${ +pair[1] + precision }`)

      pair = `${ toString(value) }e`.split('e')
      return +`${ pair[0] }e${ +pair[1] - precision }`
    }
    return func(number)
  }
}
'linebreak-style': [2] 换行
文件类型多是 CRLF，但是 eslint 里面配置的类型是 LF

一篇帮助说明 https://help.github.com/articles/dealing-with-line-endings/

这篇文章大概说明了换行符 http://adaptivepatchwork.com/2012/03/01/mind-the-end-of-your-line/

代码格式问题
代码格式是一件很重要的事情，很重要，很重要

package.json 的环境依赖
开发环境所需的依赖装到 devDependencies，下面三个依赖就没装,当然还有babel-cli

"babel-plugin-transform-remove-strict-mode": "0.0.2",
"babel-preset-es2015": "^6.24.1",
"babel-preset-es2016": "^6.24.1",
发现在 package.production.json 安装了。发现另一个打包命令是 bin/build，window 下，打开 git bash 到指定目录输入 bin/build 即可

线下打包
babel: "babel src -d src"，会覆盖掉 src 的代码，万一有修改的代码没有加到暂存区。。。到时候完全找不到了，这也意味着线下打包有坑，可能就不太乐意线下打包了，也就导致无法测试打包后的内容。 解决方案

"prebabel": "git stash",
"babel": "babel src -d src",
"nej": "nej build ../deploy/release.conf -l warn",
"postnej": "git checkout . && git stash pop",
执行脚本不变，但是 pre+XXX 会先于目标脚本执行， post+XXX 会在目标脚本结束执行

prebabel 会将修改暂时保存起来

postnej 会将 babel 覆写的文件恢复，并将保存的修改恢复过来。

这个命令同样有点问题。。。如果之前 git stash 有内容，这次 git stash 无内容，当 git stash pop 的时候就会抛出来上次的内容，可能需要手动加回去

参考链接: http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html

图片压缩 举例: https://c.163.com/res/images/index/banner/banner-win.png?56eeda9455ed50241096f5a1f88f0ff7 近 400KB，在 https://tinypng.com/ 压缩后 72KB。。。而且肉眼没有看出来压缩前后区别。网站首页的图片有2.2M，整体静态资源有2.8M，压缩所有图片后，应该会极大减少消耗！！！

首页

<script src="/pub/core.js?9da9578e5f480450ab48e73a798b8b76"></script>
<script src="/pub/core.js?9da9578e5f480450ab48e73a798b8b76"></script>
<script src="/pub/pt_index.js?823a91687f133ca997e57d97abc973ca"></script>
core.js在首页出现两次。。。 首页m-section-cstory 内部的 u-pointer 没有作用

CDN
静态资源不用 CDN，也很快。。。但是看了下 cookie 的长度有 3448 个字符。 3448 / 1024 = 3.3671875。每个在 c.163.com 下的静态资源，当向服务器发起请求时，都需要携带这些信息，意味着 N * 3448。同时服务端每次头部也都返回 set Cookie,传输的数据同样是 N * XXX,所以需要把资源放到另一个服务器上比较好，或者走 CDN

静态资源
Cache-Control:max-age=604800
Date:Tue, 25 Apr 2017 08:33:52 GMT
ETag:"58f9d452-e401"
Expires:Tue, 02 May 2017 08:33:52 GMT
Last-Modified:Fri, 21 Apr 2017 09:43:46 GMT
缓存周期是 604800/3600/24 = 7 天，开发的迭代周期是两周。。。

帮助文档
http://support.c.163.com/md.html#!容器服务/服务管理/产品简介/服务管理功能使用协议.md 这里每点一个，就会跳转页面，但看链接模式，应该不会跳页的吧

静态资源路径，一律已 // 开头，以应对 https 化，可以全文件查找修改，也可以明确使用 https 开头
var iframe = document.createElement('iframe');
iframe.src = window.env === 'develop' ? 'http://id-dev-urs-offline.163yun.com/logout' : 'http://id.163yun.com/logout';
iframe.onload = function() {
    location.href = '/logout';
    // if (status) {
    //     console.log(iframe.contentWindow.name)
    // } else {
    //     status = 1;
    //     iframe.contentWindow.location = '/res/html/blank.html';
    // }
}
document.body.appendChild(iframe);
iframe.src 这里，线下没问题，因为是 http，但是线上是 https 的，这个代码根本无法退出，直接会被浏览器拦截掉

线上线下环境一致，尽量尽量保持一致，比如说上面一点(12)，可能要到上线才会出问题

fetch、socket.io terminator 是未压缩版本。。。而且部分加载的文件库体积都不大

https://console-test.163yun.com/sdk/163yun-0.1.1.min.js 文件会挂起，在 ttfb（服务器端返回首个字节的时间） 等了 10s+，不过只遇到了一次， https://console-test.163yun.com/sdk/s/pt_sdk.js 这个文件有点怪

公司内部有一个私有的 npm 库 在 bin/build 可以找到 npm --registry=http://rnpm.hz.netease.com/ --registryweb=http://npm.hz.netease.com/ --cache=/home/omad/.nenpm/.cache --userconfig=/home/omad/.nenpmrc install

目前 js 打包方式，已经拆成了按模块打包，之前看起来是打包成一整个文件
