# Maven常用命令

## Maven安装本地jar包
```
 mvn install:install-file -Dfile=[file path] -DgroupId=[groupId] -DartifactId=[artifactId] -Dversion=[version] -Dpackaging=jar
 example:
 mvn install:install-file -Dfile=/code/Github/kieker-1.13/build/libs/kieker-1.13.jar -DgroupId=cn.icedsoul -DartifactId=kieker -Dversion=1.0 -Dpackaging=jar
```
