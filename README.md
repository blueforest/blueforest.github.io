- 本地启动 `jekyll serve`

# github page不能自动更新解决方案

- 一般来说，只要你的github pages repository有新的提交，github的服务器就会运行jekyll编译你的repository（延时很小）

- 要找到出错的原因，首先确认下github pages服务有没有down掉；查看这个网址，[https://status.github.com/](https://status.github.com/)，可以了解服务器的运行状态。如果服务器运行良好，那么极有可能是提交的内容有错误。 可以在本地用jekyll编译一下repository，看看是否有错误。首先，更新一下本地的jekyll。github可能使用了较新版本的jekyll，所以即使你使用以前的本地jekyll编译没有问题，远端的jekyll编译时也可能出错。（当然，最好是保证本地和远端的jekyll版本一样。但是，还没有发现查看远端jekyll版本的方法。）


```
$ sudo gem install jekyll
$ jekyll --version 然后，在本地编译你的repository。

$ jekyll build --safe jekyll给出的出错信息还是很详细的，能看出在什么位置发生了错误.

```

参考[文章](http://rockhong.github.io/github-pages-fails-to-update.html)