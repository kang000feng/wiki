# Intellij IDEA配置远程 Docker，部署调试docker-compose项目
## 前言
这个其实没啥好说的，大概说一下步骤和参考的内容。

## 配置远程服务器
第一部分首先配置远程服务器，把Docker暴露在端口在，这是远程访问的前提。具体内容参考[1][2][3].  
> 注意，如果是阿里云或者腾讯云的服务器需要在重启Docker后设置安全组允许对应端口上下行。  
同时，如果系统是Centos，请参考资料[3]中提供的方式，[1][2]中提供的方式修改后Docker将无法重启。

按照博客内容，修改好对应的配置文件后重启Docker，查看对应端口发现2375端口已经有Docker进程了，这就意味着可以通过这个端口来连接Docker容器了。

## 配置Intellij IDEA
首先需要配置Docker容器，在Seeting里面配置Docker远程地址：

然后在项目里面配置对应的Docker-Compose文件和Service
## 调试
想要调试docker容器内代码需要升级Intellij IDEA至2019.01月版本。

不行不行，设置失败了，下次再看吧。
## 参考资料
1. https://blog.csdn.net/jackcheng1117/article/details/83080303
2. https://www.jianshu.com/p/41c55bfbcc68
3. https://blog.csdn.net/farYang/article/details/75949611
4. https://blog.jetbrains.com/idea/2019/04/debug-your-java-applications-in-docker-using-intellij-idea/
