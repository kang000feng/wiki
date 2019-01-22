# HTML5 Audio标签的使用
## Audio
今天整理了一下三年前写的生日快乐网页，发现自动播放音乐莫名其妙出问题了，音乐不会自动播放。虽然并没有什么大碍但是因为好奇所以花了不少时间来找出为什么。
## 实验
测试后发现，音乐文件本身并没有被加载。设置autoplay无效，添加controls之后用户点击播放按钮就可以自动播放。
## 结论
最终找到原因：2018年4月，Chrome浏览器发布了一次更新，禁用了自动播放功能。
## 关于Audio
audio标签本身的使用是非常方便的。具体使用方法可以参考：
http://www.w3school.com.cn/tags/tag_audio.asp

## 解决方法
### Web Audio API  
这个东西本身是一个很高级的东西，这里只是使用它来完成自动播放的功能。  
web audio的文档：https://www.w3.org/TR/webaudio/  
github：https://github.com/WebAudio/web-audio-api  

参考博客：https://www.cnblogs.com/lovemj/p/9915735.html  
（内容虽然还可以，但是UI称得上是博客界的反面教材啊）
### 新建JS
```js
try {
    var AudioContext = window.AudioContext || window.webkitAudioContext || window.mozAudioContext || window.msAudioContext;
    var context = new window.AudioContext();
    var source = null;
    var audioBuffer = null;

    function stopSound() {
      if (source) {
        source.stop(0);
      }
    }

    function playSound() {
      source = context.createBufferSource();
      source.buffer = audioBuffer;
      source.loop = true;
      source.connect(context.destination);
      source.start(0);
    }

    function initSound(arrayBuffer) {
      context.decodeAudioData(arrayBuffer, function (buffer) {
        audioBuffer = buffer;
        playSound();
      }, function (e) {
        console.log('Error decoding file', e);
      });
    }

    function loadAudioFile(url) {
      var xhr = new XMLHttpRequest();
      xhr.open('GET', url, true);
      xhr.responseType = 'arraybuffer';
      xhr.onload = function (e) {
      initSound(this.response);
    };
      xhr.send();
    }
} catch (e) {
  console.log('!Your browser does not support AudioContext');
}
```
引入这个js之后，在页面或者后面引入的js中直接调用：
```js
loadAudioFile("music/music.mp3");
```
就可以了，不过需要注意顺序，需要先引入定义的js，再引入调用的js。

*注意*：如果在本地打开静态文件会报跨域错误，使用nginx或者tomcat服务器即可。

## 结果
怎么说呢，结果应该算还可以吧。起作用了，不过不知道是不是我的服务器的网速过慢还是怎么样，过了很久才播放出来声音。。。   
总之，就这样吧。也不是做前端的，有点了解就可以了。

## 什么？？？
等等，等会儿，我先理一下。  
我昨天搞了半天，自认为搞清楚了audio标签autoplay失效的属性，又找了很久找到了解决办法。结果今天整理一个静态网页时打开竟然自动播放音乐了？？？而且还是别人基于我昨天改的不能放音乐的网页修改的，代码改动的不多，而且他并没有加任何处理自动播放的代码，还是我写的audio标签！没错，就是我昨天尝试过很多次认为不能自动播放音乐的audio标签，就是我自己写的！  
我自己被自己打脸了？？？  
不行，我得搞懂为什么。

## 原因
看了一下，找到了原因，上述我查到的内容是真实的，Chrome确实禁用了audio的autoplay。而今天打开的网站之所以可以自动播放音乐则是因为使用了iframe，在一个页面登录开始就使用了iframe，后面的页面其实都是从属于第一个页面的，而且因为第一个页面用户登录时已经有了交互操作（点击登录），所以autoplay就能够生效了。  
iframe的使用见：使用iframe

也不知道他是不是有意这样改的，这样确实能够解决问题。。。
