title: "《高性能JavaScript编程》读书笔记（1）"
date: 2015-06-01 00:08:34
tags: JavaScript
---

断断续续的读完了 《高性能JavaScript》，一半看得英文，一半看得中文，算是粗读了一番。不过收获还是巨大的，对  Nicholas C.Zakas 的崇拜又增加许多。书里讲了很多细节的东西，有大量的数据进行展示，隐含的编程思想和良好的编码习惯非常受用，简直相见恨晚，唯一遗憾的是时间有点久，很多数据无法提供最新版本。

对于 js 的性能问题，其实从工作到现在，一直没有系统的去了解，学习，实践和总结。想起了 PHP 大牛鸟哥关于性能优化的演讲时说的一句话，"优秀的程序员，应该在写代码时就有性能优化的意识"，

其实书中每章的 summary 总结的非常好，这里我只记录觉得很有用，平时也没注意的内容，外加一些自己的实践体会。

## 第一章 Loading and Execution
首先得明白，`<script>` 标签的出现使整个页面因解析、运行而出现等待，阻塞页面解析和用户交互。
书中提及一个 `<script>` 标签在下载外部资源时，不必阻塞其他 `<script>`，但会阻塞其他资源的下载过程。但我测试了下，在最新版本的 chrome 中 js 并没有阻塞其他资源的下载，而 Firefox，Safari 阻塞了图片资源的下载，没有阻塞 css 的下载，对浏览器所做的优化还是不太明白。（相关[资源](http://www.ravelrumba.com/blog/script-downloading-chrome/)，后续研究）
保证 JavaScript 的加载性能，方法有几种：位置（一般 `<script>` 紧靠 `</body>` 上方），合并压缩（目前主流的构建工具 grunt，gulp都能很好的完成该任务）。
当然也有实现非阻塞的方式，有几种方法。

1. Deferred Script
```
<script>
console.log('script');
</script>
<script defer>
console.log('defer');
</script>
<script>
window.onload = function() {console.log('load')}
</script>
//script, defer, load
```
标记为 defere 的 `<script>` 元素在 onload 事件句柄处理之前被调用。

1. Dynamic Script Elements
```
function loadScript(url, cb) {
    var script = document.createElement('script');
    script.type = "text/javascript";
    if (script.readyState) {
        script.onreadystatechange = function() {
            if (script.readyState === 'loaded' || scrpt.readyState='complete') {
                script.onreadystatechange = null;
                cb();
            }
        }
    } else {
        script.onload = function() {
            cb();
        }
    }
    script.src = url;
    document.getElementsByTagName('head')[0].appendChild(script);
}
```

1. XMLHttpRequest Script Injection
```
var xhr = new XMLHttpRequest();
xhr.open('get', 'flie.js', true);
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
        if(xhr.status >= 200 && xhr.status < 300 || xhr.status == 304) {
            var script = document.createElement('script');
            script.type = "text/javascript";
            script.txt = xhr.responseText;
            document.body.appendChild(script);
        }
    }
}
```
无法跨域，因而无法使用 CDN。

