# 开发日记
安装pylint，用于保证代码质量：

```
$ cd PROJECT_ROOT
$ pip install -U git-pylint-commit-hook
$ ln -sf $(pwd)/.hooks/pre-commit .git/hooks/
```

## 整理需要做的事儿
功能：
- [x] 完善阅读模块，查询过的单词变色
- [x] 完善文章的相关信息
- [x] 图片上传
- [x] 个人主页
- 发布、编辑、删除文章功能
- 分页
- 安全问题（转义，cookie等）
- 智能爬虫获取内容

Bug：
- 内容为空不上传
- 源地址如果有http则省略协议
- 文章中单词标签化bug：超链接解析成了单词
- 上传头像不符合规格，应该显示之前的头像，而不是默认头像

优化：
- 编辑页面没有返回按钮，首页的跳转需要改成超链接，同时hover变色
- 文章需要记录最后修改文章的用户的uid
- 更新、发布、删除文章都以异步的形势返回结果
- 排序算法使用：[hacknews的排名算法](http://www.ruanyifeng.com/blog/2012/02/ranking_algorithm_hacker_news.html)
- 编辑内容时支持上传图片
- 优化静态资源（图片，css，js）体积，我觉得现在的加载速度一定很慢～
- 阅读数不能刷
- 模版中的静态文件，改成static_url
- 登录时可以按回车登录

## 编辑文章页面
使用开源项目：[Editor.md](https://github.com/pandao/editor.md)，做为markdown编辑器。
便于发布文章，支持预览。现在只是简单的模式，不支持上传图片等功能。

## 个人主页
现在实现了，更新头像功能。其它信息，后面再说。下一步，准备做管理员的管理文章的功能。

## 图片上传功能完成
使用七牛的服务，图片直接存储到七牛空间中，图片的压缩通过七牛的样式很简单。关于头像的上传，打
算放到个人主页中，如果放在注册中的话，注册耗时太长，不利于用户的积累。

## 安全问题
1. cookie过期时间
2. token防止crxf
3. 检查xss
4. 尝试引入session管理用户状态

## 文章相关信息
文章包含一下元素：作者，发布时间，中文标题，阅读数，文章分数还没有实现（后面会用hacknews的排名算法）

## 首页改版
参考了[稀土博客](https://github.com/xitu/xitu.github.io)的样式，然后使用了[mincss](https://github.com/peterbe/mincss)除去了不需要的css代码。最后，终于做出了个还算
看的过去的样子。现在是东拼西凑，后面还是要认真，系统的学习前端的知识。

![](https://github.com/521xueweihan/holleworld/blob/dev/img/holleworld-show-2016-05-26.gif)

## 选中单词变色
markdown2把文章内容转化成html，最开始我想着用字符串加正则来对单词加样式的class。但是结果
并不好，因为很多这样做会有很多情况没有考虑到：标签可能被污染；标签后面的单词没有加上样式class；
带标点的情况！

所以，我想如果用beautifulsoup中的text，会避免上述的两种情况，这样我只需要考虑带有单词带有
标点的情况。

## 整理代码，加上代码质量检测
整理模版，代码；重构代码，修改了一些变量和类的名字；区分用户和管理员；

## 重写前端基本完成
说在最前面：我的前端的知识几乎为零，所以都是修改、借鉴一些开源项目。

- [ant-design](http://ant.design/#/): 注册和登陆
- [github-markdown-css](https://github.com/sindresorhus/github-markdown-css): github-markdown-css
- [BootStrap](http://v3.bootcss.com/getting-started/)
- [BootStrap模版](http://v3.bootcss.com/examples/jumbotron-narrow/)

## 学习并弄好前端样式（长久任务）
后端的逻辑和功能基本实现，但是后面打算重写一遍，后端就先这样。下一步就是学习react，然后使用
新学的react把前端搞定。努力！

## 分享英文文章的功能（暂时屏蔽）
因为发现直接爬去不同网站的英文文章的爬虫，写好需要很大精力。所以，我转变了一下思路：参照HackNews
的模式，分享好的文章的url。用户可以根据链接先去看，如果觉得好，可以‘赞’，或者‘踩’。这样根据
情况，再把好的文章发布出来。

## 根据查询次数，改变单词的颜色
原理：对文章的内容，进行整理，每一个单词都是一个span，类名就是单词。然后根据，返回的查询次数，
选择响应的class，改变style属性。变色等级分别为:绿——蓝——紫——黄，效果如下：

![](https://github.com/521xueweihan/holleworld/blob/dev/img/show3-2016-04-05.gif)

## 安全：html的转义
因为发布新闻的功能是用户输入的内容，会有恶意用户，输入恶意js脚本，所以需要对用户输入的内容，
进行转义。现在有一个问题:因为文章中有代码块，代码块中的代码片段可以通过hightlight.js转变成
安全的内容。但是如果对用户输入的全部内容进行转义，则会造成：代码块中的代码显示出错。

解决办法：使用正则匹配，对非`<pre></pre>`标签内容进行转义。  
终极解决办法：这个问题把我都弄崩溃了，其实问题很简单，因为highlight.js对于'lt'和'gt'会渲染
成高亮，导致转移后的'>'和'<'，html无法识别！所以只需要，反转义代码块中的`&amp;`。我靠坑啊！

## 内容模块（新闻模块）
内容的来源经过我不懈的努力，终于找到了一个好的高质量，涵盖计算机编程的各方面的内容。现在的任务
就很明确了，爬过来，存到数据库中。现在的想法是：这个爬取内容的操作，做成一个独立的脚本，每天
跑一次。打算用tornado的异步HTTPClient来爬，现在正一边看文档一边着写。太开心了，终于找到了，
一个好的计算机资讯网站！！

前端使用Hightlight.js增加了代码高亮（同时调整了base模版)，实现了简单的抓去内容回来。

TODO：优化爬虫，完善内容展示页面样式，发布内容代码markdown转化过程代码高亮有问题。爬来
的文章如果带有图片，需要本地化（或者上传到七牛）。

## 翻译结果展示效果实现
原来一直走进了死胡同，其实实现这个效果，只要监听鼠标放开的事件。这样的好处就是双击和点拉选择，
都可触发事件。效果如下图（细节部分还没有做）：

![](https://github.com/521xueweihan/holleworld/blob/dev/img/show2-2016-03-29.gif)


## 增加翻译功能
对鼠标选中的内容进行翻译，调用有道翻译的api，以后或许会支持其他的翻译。待完善：由于我的前端水
平太差，所以样式简陋，js(异常处理没有搞），css也需要调（返回的翻译应该在显示在鼠标附近），需
要改的地方很多。

那也记录下现在的样子：

![](https://github.com/521xueweihan/holleworld/blob/dev/img/show1-2016-03-24.gif)

## 增加了新闻模块
新闻模块支持显示，发布。发布的操作只能是管理员才能操作。（感觉权限统一继承一个类不好，但是还没
想到搞好的办法）

## seassion和加密一系列工作
搞懂了tornado的log，之后写了一个父类，用于处理id的工作。还有就是seassion的管理

## tmplate模版
困死宝宝了！本来打算换模版引擎，把默认的换成JinJia2。后来发现好像有些麻烦，虽然实现没什么问题，
但是怕出现我控制不了的问题，暂时先这样。

现在准备抽象html，也就是在html中挖坑，然后方便以后的继承模版。

## 增加model层
因为打算完善功能，所以数据层的model是一定要做的。因为自己水平有限，所以就把廖雪峰老师写的model
层拿来用了。配置相关信息都发在config.py文件中，同时增加gitignore文件。

## 登陆功能
打算周末回家做好用户登录功能，就算做不完，也要做好数据库的工作。我一直很畏惧数据库的操作，事实
证明这种东西是躲不掉的！

## 部署网站
holleworld的部署方案本来原定是nginx＋supervisor。但是我发现tornado自带web服务器，同时
我现在又不要考虑并发问题。

所以今天我就直接没用nginx反向代理，直接启动脚本服务。然后用supervisor监视进程。