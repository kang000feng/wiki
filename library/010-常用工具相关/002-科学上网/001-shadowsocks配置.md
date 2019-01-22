# 配置ShadowSocks
## 关于ShadowSocks
很久之前就听说过ShadowSocks，因为平常有使用google的需求，正好自己也有一台香港的服务器，不用白不用，因此来配置一个vpn用于电脑和手机上网。  
而这个代理原理其实也简单，你不能访问外网，但是非大陆服务器是可以访问外网的，而我们是可以访问一些特定的非大陆服务器的。所以借助ShadowSocks，我们的网络请求会先发送到，服务器把网络请求转到外网，再把结果返回回来，这年头上个网真不容易（= =）

## 材料准备
阿里云香港云服务器（Centos 7）

## 配置ShadowSocks Server
只要按照官方文档来其实很容易就能够安装和运行Server，可参考：[Server配置使用说明](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)：  
```shell
# Centos 执行以下语句以安装shadowsocks
yum install python-setuptools && easy_install pip
pip install git+https://github.com/shadowsocks/shadowsocks.git@master

# Debian / Ubuntu: 执行如下语句
apt-get install python-pip
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```
接下来其实已经能够运行了，按照说明直接执行以下语句就可以运行。
```Shell
ssserver -p [port] -k [password] -m aes-256-cfb
```
但是为了方便以后的使用，我们还是写一个配置文件来存储我们的配置，配置文件的具体内容参考官方文档：[Config File说明](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)  
```shell
#新建文件(vi或者vim都可以，有哪个用哪个)
vim /etc/shadowsocks/config.json
```
我的配置文件内容为，这里的为基本配置，其中server不要填自己的服务器地址，保持原样，一般需要更改的仅仅为server_port和password两项。  
如果是自己想要搞一些东西比如优化速度、配置多个端口之类的可以查找教程来修改此配置文件。  
按i进入插入模式，粘贴修改或者打出以下内容，其中设置的端口和密码需要自己记得。
```json
{
  "server":"0.0.0.0",
  "server_port":20000,
  "local_address":"127.0.0.1",
  "local_port":1080,
  "password":"passsword",
  "timeout":300,
  "method":"aes-256-cfb",
  "fast_open":false,
  "workers": 1
}
```
之后按ecs键退出插入模式，输入 :wq，退出并保存此文件，然后继续执行以下语句以启动shadowsocks server端。

## 配置安全组
如果是阿里云服务器，那么一定需要配置安全组规则以放行端口，否则一定无法成功访问外网。  
登录阿里云官网，进入控制台，打开云服务器管理界面，依次点击管理、本实例安全组、配置规则（一般会有默认的安全组，如果没有安全组默认全部放行的话那么就不用管了，一般用过阿里云的人都会配吧。）。
![](assets/010/20190122-13b5819e.png)  
![](assets/010/20190122-0bf40b69.png)  
![](assets/010/20190122-181f123a.png)  
然后添加安全组规则，输入如下规则，其中上面自己配置文件中的server_port一定要写在这个端口范围填写的两个数区间之内（两个数/隔开）。
![](assets/010/20190122-86327fe3.png)  

## Windows客户端
服务器安装配置好之后，接下来只需要下载客户端然后就可以愉快的上网了。这依然有官方文档可供参考：[windows端说明](https://github.com/shadowsocks/shadowsocks-windows/wiki/Shadowsocks-Windows-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)。  
之后只需要去下载最新版的客户端然后配置就好了：[官方下载地址](https://github.com/shadowsocks/shadowsocks-windows/releases)

下载之后再合适的文件夹中加压并且打开exe文件，增加服务器配置：
![](assets/010/20190122-0971d42f.png)  

如图，在对应的位置填写自己的服务器IP，之前配置和放行的端口以及设置的密码，之后选择启用系统代理，代理模式选择全局模式，就可以访问外网了。如果出现问题，那么需要参考官方文档并且回顾之前的步骤是否出现问题。

## Android客户端
Android端要稍微麻烦些，需要先安装Google三件套。  
我的方法是安装谷歌安装器来进行安装，这个软件市面上很多，根据自己的机型做选择，一个装不了换另一个即可。
安装成功后下载ShadowSocks Android端：[下载地址](https://github.com/shadowsocks/shadowsocks-android/releases)
打开后和Windows端一样，填写服务器地址、IP和密码即可。

## 相关资料
1. Github地址： https://github.com/shadowsocks/shadowsocks/tree/master
2. Server配置使用说明：https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E
3. Config File说明： https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File
