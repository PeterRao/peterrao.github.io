title: "ES6实践"
date: 2015-05-26 16:02:14
tags:
- JavaScript
- ES6
---
在一个新项目上使用 ES6 语法有一段时间了，自己在学习 React 时，也尝试了不少 ES6 的语法。这里记录一下学习实践的经验。
<!--more -->
## Compiler
使用 [Babel](https://babeljs.io/) 进行编译。
实际使用中遇到的问题：
- 由于需要兼容 IE8，不支持 `Object.defineProperty`，使用 Babel 时，需要使用关闭 `es6.module` [选项](http://babeljs.io/docs/advanced/loose/#es6-modules)。
- Babel 严格遵守 ES2015 模块标准，ES2015 模块默认是在严格模式下，所以最外层的 `this` 并不是指的 `window` 对象，在使用其他第三方库时遇到过这个问题，在编译时，需要加入 `blacklist: ["strict"]` 选项。

## Main Syntax
[这里](https://github.com/lukehoban/es6features#readme)说明了 ES6 的主要语法，实际根据实用程度以及对 ES6 语法的理解程度，并不会全部用到，这里介绍目前用到的语法，后面用到并理解后一点一点补充。

### Arrows
箭头函数是很常用的语法了，基本抛弃了 `function` 关键字，更加简洁。
arrow functions 带来最大的改变就是固定了 `this` 的对象，箭头函数中的 `this` 指向包含该箭头函数的代码的 `this` 值，在使用过程中需要特别注意 `this` 的使用。
对比以下两段代码。
```
function Person() {
  var self = this;
  self.age = 0;
  setInterval(function growUp() {
    self.age++;
  }, 1000);
}
```
setInterval 中 `this` 一直指向 `window` 对象，在 Person 中缓存 `this` 值。
```
function Person(){
  this.age = 0;
  setInterval(() => {
    this.age++;
  }, 1000);
}
```
使用箭头函数后，`this` 指向其外层的 this 值。前者既是后者使用 bable 编译的[结果][1]
Nicholas 有篇介绍 arrow functions 中 `this` 的[文章](http://www.nczonline.net/blog/2013/09/10/understanding-ecmascript-6-arrow-functions/)。

### Class
Class 语法，除了在用 React 时，自己到用的不多，这里是详细的语法[介绍](http://www.2ality.com/2015/02/es6-classes-final.html)，在使用前需要好好看一下这个文档，后续有较多使用经验时在补充吧，这里不介绍了。
这里学习下，其编译后的[代码][2]原理。
首先可以看到其通过 `_classCallCheck` 来判断是否通过 `new` 关键字来创建对象，直接调用 `Point()` 会报错。
```
function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError('Cannot call a class as a function');
    }
}
```
然后调用 `_createClass` 方法来绑定属性和方法。原型方法绑定到 `Constructor.prototype`(`Point.prototype`) 上，静态方法绑定到 `Constructor`(`Point`) 上。在对象上定义属性时，使用了 `Object.defineProperty()` 方法，[这里](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)是其详细的说明。
```
function (Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
```
当有类继承时，如例子中的 ColorPoint 继承 Point 类时，在子类只增加了一些处理。
`_inherits` 方法将 ColorPoint.prototype 指向 由Point 的 prototype 上新创建的对象，并设置 'constructor'。
```
function _inherits(subClass, superClass) {
    if (typeof superClass !== 'function' && superClass !== null) {
        throw new TypeError('Super expression must either be null or a function, not ' + typeof superClass);
    }
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    if (superClass) subClass.__proto__ = superClass;
}
```
继续看 下面的代码
```
_get(Object.getPrototypeOf(ColorPoint.prototype), 'constructor', this).call(this, x, y);
```
`Object.getPrototypeOf(ColorPoint.prototype)` 得到的是 `ColorPoint.prototype.__proto__` 即 `Point.prototype`.
`_get()` 函数，返回父类 `Point` ，并在 ColorPoint 的 this 上，向 `Point` 传入 x, y的值，进行子类的初始化。

### Enhanced Object Literals
增强的对象字面量，可以继续看官方的[例子][3]。
这里是主要通过 `Object.defineProperty` 来实现属性的设置，Loose mode 下，直接给对象设置属性
```
function _defineProperty(obj, key, value) {
    return Object.defineProperty(obj, key, {value: value, enumerable: true, configurable: true, writable: true});
}

// lose mode
_obj2.__proto__ = theProtoObj;
_obj2.handler = handler;
...
```
这里有以下几个用法：
#### 同名变量简写
```
var obj = {
    handler
}
相当于
var handler = xxx;
var obj = {
    handler: handler
}
```
#### 方法简写
省略了 `function` 关键字
```
var handler = xxx;
var obj = {
    toString() {
    return "d " + super.toString();
    }
}
// 相当于
var obj = {
    toString: function toString() {
        return "d " + _get(Object.getPrototypeOf(_obj), "toString", this).call(this);
    }
}
```
`_get(Object.getPrototypeOf(_obj), "toString", this).call(this)` 即调用了 Object 的 toString方法。
#### 动态属性名
```
var obj = {
    [ 'prop_' + (() => 42)() ]: 42
};
```

### Template Strings
很好用的一个语法，主要用于多行字符串和字符串变量替换
```
`In JavaScript this is
 not legal.`

var name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

### Destructuring
解构用的不多，还是推荐看一下 用 bable 后的[代码](4)，加深理解。

### Default + Rest + Spread
函数参数具有默认值，剩余参数以及参数展开功能，使用方法和原理也比较简单。

### Let + Const
ES6 加入的很常用的关键字，代替了 `var` 的使用。
`let` 表示块作用域变量，`const` 表示常量。
观察其编译后的代码，可以发现其还是用 `var` 实现，Babel 会在编译时去检查语法的使用（相同块作用域不能重复使用 let 声明，常量不可以在赋值等等）

### Modules
目前使用该语法，主要通过 Broswerify 或者 Webpack 实现模块的 bundle，不过异步加载模块并没有相应的语法 ，平时最常用也是推荐用的是 `Default exports`，其完整语法比较复杂，如果要用的话，推荐深入看一下[文档](http://www.2ality.com/2014/09/es6-modules-final.html)。

其他语法都不是很常用，等实际中有使用经验后在总结。
最后还有一个非常重要而且非常有用的语法 Promise，这个后面单独介绍。

相关资源：
使用ES6进行开发的思考：[http://efe.baidu.com/blog/es6-develop-overview/](http://efe.baidu.com/blog/es6-develop-overview/)


[1]: https://babeljs.io/repl/#?experimental=true&evaluate=true&loose=false&spec=false&code=function%20Person()%7B%0A%20%20this.age%20%3D%200%3B%0A%20%20setInterval(()%20%3D%3E%20%7B%0A%20%20%20%20this.age%2B%2B%3B%0A%20%20%7D%2C%201000)%3B%0A%7D
[2]: https://babeljs.io/repl/#?experimental=true&evaluate=true&loose=false&spec=false&code=class%20Point%20%7B%0A%20%20constructor(x%2C%20y)%20%7B%0A%20%20%20%20this.x%20%3D%20x%3B%0A%20%20%20%20this.y%20%3D%20y%3B%0A%20%20%7D%0A%20%20toString()%20%7B%0A%20%20%20%20return%20'('%20%2B%20this.x%20%2B%20'%2C%20'%20%2B%20this.y%20%2B%20')'%3B%0A%20%20%7D%0A%7D%0Aclass%20ColorPoint%20extends%20Point%20%7B%0A%20%20%20%20constructor(x%2C%20y%2C%20color)%20%7B%0A%20%20%20%20%20%20%20%20super(x%2C%20y)%3B%0A%20%20%20%20%20%20%20%20this.color%20%3D%20color%3B%0A%20%20%20%20%7D%0A%20%20%20%20toString()%20%7B%0A%20%20%20%20%20%20%20%20return%20super.toString()%20%2B%20'%20in%20'%20%2B%20this.color%3B%0A%20%20%20%20%7D%0A%7D
[3]: https://babeljs.io/repl/#?experimental=true&evaluate=true&loose=false&spec=false&code=var%20obj%20%3D%20%7B%0A%20%20%20%20%2F%2F%20__proto__%0A%20%20%20%20__proto__%3A%20theProtoObj%2C%0A%20%20%20%20%2F%2F%20Shorthand%20for%20%E2%80%98handler%3A%20handler%E2%80%99%0A%20%20%20%20handler%2C%0A%20%20%20%20%2F%2F%20Methods%0A%20%20%20%20toString()%20%7B%0A%20%20%20%20%20%2F%2F%20Super%20calls%0A%20%20%20%20%20return%20%22d%20%22%20%2B%20super.toString()%3B%0A%20%20%20%20%7D%2C%0A%20%20%20%20%2F%2F%20Computed%20(dynamic)%20property%20names%0A%20%20%20%20%5B%20'prop_'%20%2B%20(()%20%3D%3E%2042)()%20%5D%3A%2042%0A%7D%3B
[4]: https://babeljs.io/repl/#?experimental=true&evaluate=true&loose=false&spec=false&code=%2F%2F%20list%20matching%0Avar%20%5Ba%2C%20%2C%20b%5D%20%3D%20%5B1%2C2%2C3%5D%3B%0A%0A%2F%2F%20object%20matching%0Avar%20%7B%20op%3A%20a%2C%20lhs%3A%20%7B%20op%3A%20b%20%7D%2C%20rhs%3A%20c%20%7D%0A%20%20%20%20%20%20%20%3D%20getASTNode()%0A%0A%2F%2F%20object%20matching%20shorthand%0A%2F%2F%20binds%20%60op%60%2C%20%60lhs%60%20and%20%60rhs%60%20in%20scope%0Avar%20%7Bop%2C%20lhs%2C%20rhs%7D%20%3D%20getASTNode()%0A%0A%2F%2F%20Can%20be%20used%20in%20parameter%20position%0Afunction%20g(%7Bname%3A%20x%7D)%20%7B%0A%20%20console.log(x)%3B%0A%7D%0Ag(%7Bname%3A%205%7D)%0A%0A%2F%2F%20Fail-soft%20destructuring%0Avar%20%5Ba%5D%20%3D%20%5B%5D%3B%0Aa%20%3D%3D%3D%20undefined%3B%0A%0A%2F%2F%20Fail-soft%20destructuring%20with%20defaults%0Avar%20%5Ba%20%3D%201%5D%20%3D%20%5B%5D%3B%0Aa%20%3D%3D%3D%201%3B
