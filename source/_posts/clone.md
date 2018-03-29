---
layout: '[layout]'
title: JavaScript深拷贝与浅拷贝
date: 2018-3-12 11:12:31
comments: true
toc: true
tags:
	- 前端
	- JavaScript
---
### 1.JavaScript变量类型
说到js的深拷贝与浅拷贝，先说一下js的变量类型。js的变量分为基本数据类型与引用类型。

基本数据类型有：`number`、`string`、`boolean`、`null`、`undefined`。变量是直接以值的方式存放在栈内存当中的，可以直接访问。
<!-- more -->

引用类型：`object`、`array`。存储在堆内存当中，而存储指向堆内存地址的变量（例如你家的门牌号）存储在栈内存中，因此访问引用类型时，先从栈内存中找出改对象的地址，才能访问到堆内存中存储的数据。
### 2.浅拷贝
```
    var x = {
        name:'ttc',
        age:28
    };
    var y = x;
    y.age = 30;
    console.log(x.age) //30
```
其中`y = x`只不过是指向x地址的指针复制给了y，因此y跟x指向的是同一个对象，无论通过y还是x修改了引用类型中的数据，堆内存中改引用类型的数据都会改变。

注意：Object.assign
Object.assign实现的操作很容易让人误会是深拷贝，举个栗子:chestnut:
```
   var x = {
        name:"ttc",
        age:28,
        child:{age:2}
   };
   var  y = Object.assign({},x);
   y.age = 30;
   console.log(x.age) //28  看起来像是深拷贝的操作
   y.child.age = 5;
   console.log(x.child.age) //5 本性暴露，原来它是浅拷贝！
   类似的还有Array.prtotype.slice()和Array.prototype.contact()
```
### 3.深拷贝
既然明白了浅拷贝的原理，那深拷贝的原理就很容易理解了。意思就是拷贝出一个新的实例，新实例与旧实例之间互不影响。
##### 实现深拷贝的方法：
###### 3.1 JSON.parse与JSON.stringify
```
 var x = {
    arr:[1,2,3],
    obj:{
        value:"tom"
    },
    func:function(){
        return 1;
    }
 };
 var y = JSON.parse(JSON.stringify(x));
 y.arr[1] = 5;
 y.obj.value = "jack";
 console.log(x) //没有发生改变
 console.log(y) //{arr: [1,5,3], obj: {value: 'jack'}}
```
这里确实会实现深拷贝，但是源对象的方法会在拷贝过程中丢失。这是因为`在序列化JavaScript对象时，所有函数和原型成员会被有意忽略`。因此这种方式只能实现较为简单的需求。
结构化克隆算法：
https://developer.mozilla.org/zh-CN/docs/Web/Guide/API/DOM/The_structured_clone_algorithm
###### 3.2 借助lodash或者immutable.js
```
    var objects = [{ 'a': 1 }, { 'b': 2 }];
    var deep = _.cloneDeep(objects);
    console.log(deep[0] === objects[0]);
    // => false
```
链接：https://lodash.com/docs/4.17.5#cloneDeep
```
  var map1 = Immutable.Map({a:1, b:2, c:3});
  var map2 = map1.set('b', 50);
  map1.get('b'); // 2
  map2.get('b'); // 50
```
链接：https://facebook.github.io/immutable-js/

lodash也提供了 cloneDeepWith 可以 customize 特殊对象的拷贝行为。不过 lodash 的实现并不会复制原型，只是复制原型上的属性。
###### 3.3递归实现深拷贝
```
    function deepClone(source){
       if(!source || typeof source !== 'object'){
         throw new Error('error arguments', 'shallowClone');
       }
       var targetObj = source.constructor === Array ? [] : {};
       for(var keys in source){
          if(source.hasOwnProperty(keys)){
             if(source[keys] && typeof source[keys] === 'object'){
               targetObj[keys] = source[keys].constructor === Array ? [] : {};
               targetObj[keys] = deepClone(source[keys]);
             }else{
               targetObj[keys] = source[keys];
             }
          }
       }
       return targetObj;
    }
```
