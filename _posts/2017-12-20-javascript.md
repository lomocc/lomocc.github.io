---
title: Javascript 随笔
tags: js
---
# Javascript 随笔

## 简单笔记
* 在严格模式下，如果`this`未在执行的上下文中定义，那它将会默认为`undefined`。

```js
function f1(){
  return this;
}
//在浏览器中：
f1() === window;   //在浏览器中，全局对象是window

//在Node中：
f1() === global;
```
```js
function f2(){
  "use strict"; // 这里是严格模式
  return this;
}

f2() === undefined; // true
```

* 虽然构造器返回的默认值是`this`所指的那个对象，但它仍可以手动返回其他的对象（如果返回值不是一个对象，则返回`this`对象）。

```js
function C(){
  this.a = 37;
}

var o = new C();
console.log(o.a); // logs 37


function C2(){
  this.a = 37;
  return {a:38};
}

o = new C2();
console.log(o.a); // logs 38
```
[MDN this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)

* `reduce/reduceRight` `(callback[, initialValue])` 参数 `initialValue` 的坑

Value to use as the first argument to the first call of the callback. **If no initial value is supplied, the first element in the array will be used.** Calling reduce() on an empty array without an initial value is an error.

## 造轮子与包装
作为一个 `JavaScript 程序员`，非常不想看到想用的库被过多的包装。比如 [Ant-Design](https://github.com/ant-design/ant-design) , 以及刚看到的 [Alibaba ICE](https://github.com/alibaba/ice)。核心又依赖其他包，这样做拿来即用，对新手很方便，但如果需要定制化开发就很难。

## greenlet
[greenlet](https://github.com/developit/greenlet) 是一个创建 `Worker` 的库, 支持 `Promise`, 目前因为 `Async` 语法的原因只支持 `Chrome 55+`.

有个作用域限制：

`Caveat: the function you pass cannot rely on its surrounding scope, since it is executed in an isolated context.`

* `Async`
```js
import greenlet from 'greenlet'

let getName = greenlet( async username => {
    let url = `https://api.github.com/users/${username}`
    let res = await fetch(url)
    let profile = await res.json()
    return profile.name
});

console.log(await getName('developit'))
```
* `Promise`
```js
let getName = greenlet((username)=>{
  let url = `https://api.github.com/users/${username}`;
  return fetch(url)
    .then(res=>res.json())
    .then(profile=>profile.name);
});

getName('developit').then(console.log, console.error);
```
