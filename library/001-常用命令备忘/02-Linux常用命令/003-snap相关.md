# Snap相关

##　安装Snap

```shell
yum copr enable ngompa/snapcore-el7
yum -y install snapd
yum copr enable ngompa/snapcore-el7
yum install epel-release
yum install yum-plugin-copr
yum copr enable ngompa/snapcore-el7
yum -y install snapd
systemctl enable --now snapd.socket
ln -s /var/lib/snapd/snap /snap
```