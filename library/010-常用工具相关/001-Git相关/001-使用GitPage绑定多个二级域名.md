## 使用Github Pages 绑定多个二级域名
### 关于Github Pages
这是Github Pages的官方介绍：https://pages.github.com/  

大意就是给个人组织或者官方提供部署静态网页的功能，官方推荐的使用方法有作为个人或者组织的介绍主页，包括个人博客、个人介绍、组织介绍等等。同时也推荐用它来介绍某个项目，很多项目的介绍也都是使用Github Pages搭建的。

其实归纳一下主要有以下几个重点：
1. 免费
2. 每个账号可以拥有一个Github Page，默认域名为username.github.io
3. 只允许放置静态资源，默认寻找index.html作为入口
4. 可以自定义域名

### 使用
相信大多数人使用Github Pages都是用来搭建个人博客的，无论是Github推荐的Jekyll或是hexo，网上都有不少非常详细的搭建教程，这里也就不再多说。  
搭建一个博客不用多说，自定义域名也很简单，主要就是购买域名进行解析，然后在repository的seeting里面填写custom domain即可。可是一个博客还好说，我今天因为搭建了个人wiki，所以想要解析两个二级域名，并且都使用Github Pages。  
我直接新建了一个repository，设置好Github Pages，发现默认给我解析的域名是我之前博客的自定义域名+这个repository的名字。这也证实了每个账号只拥有一个Github Pages的说法，并不是每个repository都有一个，新的repository仍然可以作为Github Pages被访问，但是只能作为之前定义好的Github Pages的子项目，但是我的需求是把这两个Repository的内容分别解析到两个二级域名里，摸索了一番之后，我找到了解决方法。

### 实现方法
1. 分别新建两个Repository，如我的两个分别名为blog和wiki。（关于Repository命名，曾经必须命名为username.github.io，现在好像不用了？我很久之前就用了Github Pages所以不确定。如果这样不行那就先建一个username.github.io的仓库开启Github Pages后再按照这个方法来。）
![](assets/010/20190121-e37b01fa.png)  
2. 在两个Repository分别上传对应的静态页面，根目录需要包含index.html入口。
3. 在两个Repository的Setting中的Github Pages里面选择Source项为分支名（默认上传为为master，如果自己上传到别的分支那么选择上传的分支），然后在Custom domain里面填写自己的二级域名。另一个仓库也如此设置，填写另外一个二级域名。
![](assets/010/20190121-27bbc2ad.png)  
4. 在域名供应商的域名解析服务中进行解析。首先删除其它不必要的解析，然后将一级域名使用CNAME方式解析到username.github.io。如图，以阿里云为例。
![](assets/010/20190121-cd10bd41.png)  

之后等待一小会儿，就可以使用两个二级域名分别访问两个Repository里面的静态页面了。

### 存疑
其中Github Pages提供了一个Enforce Https的功能，一个二级域名使用的使用用蛮好的，但是当我把两个二级域名都勾选时，其中一个wiki会报不安全的错误，仔细查看发现颁发的https证书是属于github而不是wiki.icedsoul.cn的，所以会报当前网站不安全的错误。去掉勾选使用http访问就不会有错误了。而第一个blog.icedsoul.cn的证书则是属于自身的，莫非Github只会给一个域名颁发https证书？应该将证书给一级域名？现在还不太确定，等明天看看如果还是这个样子的话那么就再寻找解决办法吧。
