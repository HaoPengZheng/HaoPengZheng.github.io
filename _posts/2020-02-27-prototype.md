---
layout: post
title: 继承与原型链
categories: [JavaScript, prototype]
description: 面向对象是软件开发的一个重要思想，在基于类的语言中，继承是十分浅显易懂的。本文将总结JavaScript中的一些原型链知识，并探讨如何基于原型实现继承。
keywords: keyword1, keyword2
---

## 继承
面向对象的三大特性：封装、继承、多态。继承可以使我们更好的复用代码，极大的提高了软件开发的效率。<br>
关于JavaScript中原型相关的属性：
1. \_\_proto\_\_  
2. [[Prototype]] 
3. Object.getPrototypeOf() 
4. prototype
   
    在方法中，默认还有prototype属性，可以结合new关键字使用。

5. 

### 继承属性
在JavaScript中，对象是一个属性的集合，当我们试图访问一个对象的属性时，它不仅会在对象本身上搜索，还会搜索该对象的原型，直至搜索到该属性或者原型末端为止。
```js
//下面演示下原型链搜索的过程
function People(){
  this.money = 0 //刚出生没有钱
}
People.prototype.eyeCount = 2 //人有两个眼睛
People.prototype.mounthCount = 1 //人有一个嘴巴

//让我们来创建一个人
var somebody = new People()

//这个过程 somebody 的原型是People的原型，People的原型是Object
// {money：0} ==> {eyeCount：2，mounthCount：1} ==> Object.prototype ==> null

//试图搜索money属性，是自身属性
console.log(somebody.money)//0
//试图搜索eyeCount，是原型上的属性，可以访问
console.log(somebody.eyeCount) //2
console.log(somebody.mounthCount)//1
//试图访问其他属性
console.log(somebody.age)//undefined
```

### 继承方法
JavaScript 并没有其他基于类的语言所定义的“方法”。在 JavaScript 里，任何函数都可以添加到对象上作为对象的属性。函数的继承与其他的属性继承没有差别，包括上面的“属性遮蔽”（这种情况相当于其他语言的方法重写）。

当继承的函数被调用时，this 指向的是当前继承的对象，而不是继承的函数所在的原型对象。

```js
var People = {
  name:'',
  sayHello(){
    console.log('Hello!I am '+this.name)
  }
}
var Tom = Object.create(People)
Tom.name = 'Tom'
var Tim = Object.create(People)
Tim.name = 'Tim'
Tom.sayHello()
//this -> Tom
Tim.sayHello()
```

## 使用不同的方法来生成原型链

### 使用语法结构创建的对象
```js
var o = {}
// o ==> Object.prototype ==> null
// o可以访问Object.prototype的hasOwnProperty方法

var arr = [1,2]
// arr ==> Array.prototype ==> Object.prototype ==> null


function f(){
  return 2;
}

// 函数都继承于 Function.prototype
// (Function.prototype 中包含 call, bind等方法)
// 原型链如下:
// f ==> Function.prototype ==> Object.prototype ==> null
```
### 使用构造器创建的对象
```js
function People(){
  this.money = 0 
}
People.prototype.eyeCount = 2 
People.prototype.mounthCount = 1
People.prototype.sayHello = function (){
  console.log('hello')
}

var somebody = new People()

// somebody ==> People.prototype ==> Object.prototype ==> null
```
### 使用Object.create (ES5)创建的对象
ECMAScript 5 中引入了一个新方法：Object.create()。可以调用这个方法来创建一个新对象。新对象的原型就是调用 create 方法时传入的第一个参数：
```js
var a = {a: 1}; 
// a ---> Object.prototype ---> null

var b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (继承而来)

var c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

var d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); // undefined, 因为d没有继承Object.prototype
```
### 使用class(ES6)创建的对象
ECMAScript6 引入了一套新的关键字用来实现 class。使用基于类语言的开发人员会对这些结构感到熟悉，但它们是不同的。JavaScript 仍然基于原型。这些新的关键字包括 class, constructor，static，extends 和 super。
```js
'use strict'
class People{
  constructor(){
    this.money = 0
  }
  sayHello(){
    console.log('hello')
  }
}
class Student extends People{
  constructor(name){
    super()
    this.name = 'Tom'
  }
  sayHello(){
    console.log('hello,I am a student')
  }
  sayName(){
    console.log('hello,I am ' + this.name)
  }
}
var tom = new Student('Tom')

// tom ==> Student.prototype ==> People.prototype ==> ...

```


## 拓展
### 实现一个new

```js
/*
 *既然理解了原型链的过程，我们也基本了解新建一个对象的过程
 *JS编译器会做的四件事情：
 *1.创建空的对象
 *2.将构造函数的作用域指向新的对象
 *3.将新对象的原型指向构造函数的原型
 *4.返回对象
 */
function myNew(con){
  if(typeof con == 'function'){
    var newObj = {}
    var arg = Array.from(arguments)
    arg.splice(0,1)
    con.apply(newObj,arg)
    newObj.__proto__ = con.prototype
    return newObj
  }
  console.error(' TypeError: con is not a constructor')
}

```


### 性能
在原型链上查找属性比较耗时，对性能有副作用，这在性能要求苛刻的情况下很重要。另外，试图访问不存在的属性时会遍历整个原型链。

遍历对象的属性时，原型链上的每个可枚举属性都会被枚举出来。要检查对象是否具有自己定义的属性，而不是其原型链上的某个属性，则必须使用所有对象从 Object.prototype 继承的 hasOwnProperty 方法。下面给出一个具体的例子来说明它：
```js
function People(){
  this.money = 0 
}
People.prototype.eyeCount = 2 
People.prototype.mounthCount = 1

var somebody = new People()
somebody.hasOwnProperty('eyeCount')
//false
somebody.hasOwnProperty('money')
//true
somebody.__proto__.hasOwnProperty('eyeCount')
//true
```