## Go语言在Windows环境下如何配置

首先，有一点要说明的是，Go语言在64位的Windows机器上，目前已确认win 8是不能进行debug的。所以，想debug的话，就省省吧，我花了两天时间追查这个问题，直到看到这个页面才确定不是我配置的问题[GOROOT_FINAL substitution fails on Windows](http://code.google.com/p/go/issues/detail?id=5458)

页面底部，Go语言开发人员说会在1.3版本中解决这个问题。看来，目前我的机器上是无法进行debug的了。debug对于我来说很重要，观察一个程序是如何运行的，看懂一段源代码是如何组织的，都是通过debug得到的。

另一个问题是如何配置GOPATH等问题。如果Go安装的时候没有为你设置，那么你可以自己设置一下，在系统--》高级系统设置--》环境变量里设置，如图：
![GOPATH](./img/1.png)

GOPATH可以设置多个，我这里设置了C:\Go\package作为第一个，因此当使用go get安装包的时候，包就会安装到这个目录里来。可以设置多个。

目前比较好的Go语言IDE是国人开发的那个LiteIDE，不过下载下来用的时候，还不能达到智能提示的效果。网上都说要安装gocode，但是我安装了也没有奏效，后来才解决了，方法如下  

+ 把go语言安装里的bin文件夹加到环境变量的Path里
+ 记得系统已经安装了git且也已经加入到环境变量里
+ 打开命令行，执行命令：go get github.com/nsf/gocode ,这样就能安装gocode了
+ 安装好之后，你会在C:\Go\package\bin里找到gocode.exe  （按照你的安装路径去找），把它复制到
C:\Go\bin 文件夹里
+ 现在打开LiteIDE，就能自动代码提示了。

我觉得LiteIDE写得真的很不错，至少界面的文字读起代码来就非常舒服，非常适合中国人的阅读习性。

