title: "JavaScript中~的妙用"
date: 2015-07-23 21:16:02
tags: JavaScript
---
~: JS 中按位取反的一元操作符。在实际工作中，遇到的 JS 的位操作并不是很多，但 `~` 有些妙用。
<!-- more -->
### 判断值是否为 -1
因为 `~-1` 结果为 0，因此实际中配合 indexOf 可以用于判断语句中。

```js
if(~item[key].toLowerCase().indexOf(query)) {
    _result.push(item)
}
```

### ~~ 用于向下取整
~~ 和 Math.floor 用同样的功能，具体性能可参考该[文章](http://rocha.la/JavaScript-bitwise-operators-in-practice)

不过这些用法都属于奇淫技巧，不利于代码的可读性，并不推荐使用。