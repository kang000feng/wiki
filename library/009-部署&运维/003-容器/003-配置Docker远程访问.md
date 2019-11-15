# 配置Docker远程访问

首先，查看docker.service所在的目录：

```shell
systemctl status docker
```

可以看到docker.service 所在的目录，然后修改docker.service:

```
vi /usr/lib/systemd/system/docker.service
```

然后找到ExecStart=/usr/bin/docker这一行，在后面添加;

```shell
-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

然后运行：

```shell
systemctl daemon-reload
systemctl restart docker
```

