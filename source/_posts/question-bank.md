---
title: 题库整理
date: 2019-09-28
categories:
- 知识积累
tags:
- 题库
---

# 面试题库整理

## TX题

### 选择题

1、请问一下程序输出的是？

```js
function Foo() {
  var i = 0;
  return function() {
    document.write(i++);
  };
}
var f1 = Foo(), f2 = Foo();
f1();
f1();
f2();

// A. 010
// B. 012
// C. 000
// D. 011
```

2、请问一下JavaScript代码，再浏览器中运行的结果是

```js
var a =  1234 < 0 || typeof(1234 + '');
console.log(a);

// A. true
// B. string
// C. undefined
// D. false
```

3、请问以下程序输出是？

```js
var myObject = {
  foo: 'bar',
  func: function() {
    var self = this;
    console.log(this.foo);
    console.log(self.foo);
    (function() {
      console.log(this.foo);
      console.log(self.foo);
    })();
  },
}

myObject.func();

// A. bar bar bar bar
// B. bar bar bar undefined
// C. bar bar undefined bar
// D. undefined bar undefined bar
```

4、关于这段代码结论是

```js
var F = function() {};
Object.prototype.a = function() {}
Function.prototype.b = function() {}
var f = new F();
// A. f能取到a，但取不不到b;
// B. f能取到a,b
// C. F能取到b，不能取到a
// D. F能取到a, 不能取到b
```

5、请问执行后弹出的值是

```js
var name = 'World!';
(function() {
  var name;
  if (typeof name === 'undefined') {
    name = 'Jack';
    console.log('Goodbye' + name);
  } else {
    console.log('hello' + name);
  }
})();

// A. Hello World!
// B. Goodbye Jack
// C. Hello Jack
// D. Goodbye World!
```

6、下面求a中最大值正确的是

```js
var a = [1,3,5,2,9];
// A. Math.max(a);
// B. Array.max(a);
// C. Math.max.call(null, a);
// D. Math.max.apply(null, a);
```

7、关于跨域问题，下面说法正确的是？

```js
// A. 可以利用flash和http请求，来处理跨域问题
// B. 可以通过iframe设置document.domain可以实现跨域
// C. 一般情况下，m.toutiao.com可以ajax请求www.toutiao.com域名下的接口并获得响应
// D. 通过jsonp方式可以发出post请求其他域名下的接口
```

8、运行结果是

```js
var obj = {
  a: 1,
  b: function() {
    alert(this.a);
  }
}

var fun = obj.b;
fun();

// A. 弹出a
// B. 弹出1
// C. 弹出undefined
// D. 什么也看不到
```

### 编程题

1、怎么实现一个函数，将一个多层嵌套的数组扁平化（非递归实现）

例:

```js
[1,2,5,[3,7,[2,4,6],10],8,12] => [1,2,5,3,7,2,4,6,10,8,12];
```

2、写一个函数，实现数组从小到大排序；考虑复杂度

```js
[1,2,5,3,7,2,4,6,10,8,12]
```

### 面试题

1. 项目亮点，性能优化，组件封装
2. Vue3与Vue2的区别，解决了什么问题
3. proxy是es哪个版本
4. http是怎么缓存的
5. https怎么加密又是怎么鉴定安全的
6. 强缓存和协商缓存的区别，modify和etag区别
7. 跨域问题，如何解决
  a. cros
  b. jsonp
8. nginx负载均衡配置哪个属性
9. 在koa框架中，怎么开启多线程
10. 部署问题
11. CSS清浮动及其原理
12. window.onload和document.contentloaded有什么区别
13. vue的diff算法
14. vue的key的作用，为什么要加key

## AL面试题

### 理论题

1. https了解情况及其加密方式是对称还是非对称？
2. vue，react双向绑定原理
3. react的setState是异步的吗？为什么？原理是什么？
4. react事件的原理
5. localstorage能否跨域
6. localstorage和sessionStorage区别
7. react的diff原理
8. 浏览器内核原理
9. redux的在中间件原理，redux-thunk原理
10. http和https的区别

### 实操题

1. 大数组中包含4个小数组，分别找到每个小数组的最大值，然后把它们串联起来，形成一个新数组。
2. 把一个数组arr按照指定的数组大小size分割成若干个数组块
