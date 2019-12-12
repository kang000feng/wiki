# Java Heap Dump

## Get Heap Dump File

Run：

```shell
# get container id
docker ps -a
# enter container
docker exec -it [container id] /bin/bash
# get process id
ps -ef | grep [key word]
```

to get process id you want to dump.

Then run:

```shell
jmap -dump:format=b,file=/mnt/heapdump.bin [process id]
```

Then come back to host machine and can find the file in `/mnt`, then modify file permission to enable you download this file by scp.

```shell
sudo chmod 666 /mnt/heapdump.bin
```

then just run follow command in your local machine to get this file：

``` shell
scp [username]@[server host or ip]:/mnt/heapdump.bin /root/	
```

## Analyze Heap Dump File

I use eclipse mat to analyze this dump file:



## References

1. https://dzone.com/articles/how-to-capture-java-heap-dumps-7-options