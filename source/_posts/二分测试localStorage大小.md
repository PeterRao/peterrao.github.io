title: "二分法测试localStorage大小"
date: 2013-11-27 22:38:31
tags:
- 算法
- JavaScript
---
大概知道localStorage大小大约在2~5M之间，直接用10M的字符串然后二分的方式测试一下不就可以很快的测试出容量的大小了。
<!-- more -->
```
var LS = window.localStorage;

//生成10M的字符串
var value = "aaaaaaaaaa";
for(var i = 0; i < 20; i++) {
    value = value + value;
}
var testStr = value;

function test(str) {
    try {
        localStorage.test = str;
        return true;
    } catch(e) {
        return false;
    }
}

//二分
var low = 0;
var high = value.length;
var lastHigh;
while(low < high) {
    var middle = Math.floor((low + high) /2);
    testStr = value.substring(0, middle  + 1);
    var rel = test(testStr);
    if (rel == false) {
        high  = middle;
        lastHigh = high;
    } else {
        high = lastHigh;
        low = middle + 1;
    }
}
console.log(low);
```
在本机chrome测试下得到值为5242876，大约5M左右。这有个地址可以在线测试localStorage的大小，可以参考其实现方式[http://dev-test.nemikor.com/web-storage/support-test/](http://dev-test.nemikor.com/web-storage/support-test/)和[http://arty.name/localstorage.html](http://arty.name/localstorage.html)