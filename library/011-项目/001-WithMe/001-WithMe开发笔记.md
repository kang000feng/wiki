# WithMe开发笔记

## 2019.05.20
目前其实大部分基本工作已经做完了：
1. 前端仍然决定沿用之前的，不做太大的修改。
2. 使用微服务设计思想，服务之间通信使用resttemplate http。
3. 服务器使用Netty + WebSocket

前几天在思考Netty多实例的问题，后来发现还是想的太多了，先把单实例的搭起来调好再说吧。目前准备做的工作是：

1. 实现聊天记录的存储以及查询。
2. 实现离线消息的存储以及查询。
3. 实现群组聊天功能。

第一个问题我其实在纠结，刚开始想的是聊天记录先存redis，然后redis过期时再转存mysql，后来发现这样其实不太好，查了查也都没有这样做的，一般都是用redis当做缓存。后来想想确实还是只用redis当缓存的方式比较好，转存后查询起来确实麻烦。但是插入时就需要使用mysql了，那么像之前一样注入bean的方式当然是不能用了，能想到的最简单的方式是直接发送http请求，但是http请求比较耗时，因为是即时通讯从性能来考虑最好是发异步http请求，异步http又想到是不是可以放到mq里面，但是我对mq实在是不了解。过了一会儿我又想到既然http可以那rpc是不是也可以，然后翻了翻原来整理的使用rpc的demo，发现之前用的双方都是spring boot，这一方是spring boot一方是普通Java的还不太好办，使用dubbo吧发现还得用zookeeper，想想还是算了吧，后面再说。目前先用简单的http来实现吧。  
关于发送http请求之前我就简单查过，决定选用okhttp来发送，不是不想用RestTemplate，只是因为Netty服务器没有Spring的包，想使用有些麻烦，不如直接使用okhttp，正好很久之前就想试着用一下了。

## 2019.05.21
昨天查了一下，最后还是没用使用okhttp，因为它用起来貌似有些麻烦，后来是准备用RestTemplate的，它在非Spring项目中使用倒也不麻烦，只是需要导入spring-web这个包，直接结果就是让websocket-server-service 的jar包大小翻了一倍。。。仅仅为了一个函数就导入一个99%代码用不上的第三方jar包，唉。。。  
后来想了一想，为了避免插入数据库耗时过长影响即时聊天的时间，采用了异步的方式来发送插入聊天记录的http请求似乎更好一点，所以我又查了一下AsyncRestTemplate发现它过时了，推荐的替代方案是位于spring-webflux包下的WebClient，那就用呗，尝试着简单用了一下，调用成功了，不过目前是直接调用block，其实还是类似于同步的，后面等功能做的差不多时专门对比下同步和异步请求时间差多少。  
聊天记录成功插入之后就是聊天记录的查询了，今天调试修改前段和message-service后功能大概也跑通了，之前写的前端处理聊天消息的代码还是蛮好用的2333，小bug基本都出现在新写的后台。下面总结一下今天遇到的问题和知识点：  
1. spring-data-jpa 可以使用分页查询，具体做法是传入一个Pageable类型的参数，在查询之前使用PageRequest.of()新建Pageable对象，设置查询的页数、每页有多少条数据以及排序规则。
```java
Pageable sortedByTime = PageRequest.of(page, number, Sort.by("time"));
List<Message> messages = messageRepository.findAllByFromIdAndToIdOrFromIdAndToId(userIdA, userIdB, userIdB, userIdA, sortedByTime);
```
2. spring-data-jpa 默认可以解析or并且允许查询变量重复出现，如：
```java
/**
 * 查询两个用户之间的聊天记录
 *
 * @param fromId 用户A ID
 * @param toId 用户B ID
 * @param fromId2 用户B ID
 * @param toId2 用户A ID
 * @param pageable 分页对象，用户分页和排序
 * @return
 */
  List<Message> findAllByFromIdAndToIdOrFromIdAndToId(Integer fromId, Integer toId, Integer fromId2, Integer toId2, Pageable pageable);
```
这种查询也是支持的，只是参数传两遍貌似不太优雅，不过也想不到太好的解决方法。  
到目前为止正常实现的功能呢：
1. 登录/注册；
2. 双人聊天聊天记录的存储以及查询；
3. 聊天记录分页查询。
有待优化和实现的地方：
1. 聊天记录查询使用redis缓存;
2. 前端在顶部添加查看全部聊天记录按钮，弹出新窗口显示所有聊天记录。
第一点没有实现的原因是我想到：聊天记录是实时更新的，因为随时会有新的记录插入进去，那么这种情况下用redis做缓存意义是什么，几乎每次都要去查询新的结果，这种情况似乎不太适合吧。但是直接用mysql感觉又不太好，所以现在索性就先这样，后面等压测调优时找瓶颈，找到这里的话那就再改。  
第二点是属于功能上的完善，要加一些前端的内容，后面再加，先把几个基础功能先恢复了再说。

接下来应该实现的是：
1. 群组消息的转发。
2. 群组消息的存储以及查询（分页，优化结构）。
3. 创建群组、邀请人加群、查看群组成员功能的完善。
4. 离线消息的存储和查询、转发。

后面加油，慢慢做吧。
