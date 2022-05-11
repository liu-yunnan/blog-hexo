---
title: new操作符实现原理
date: 2022-03-30 13:41:17
tags: [JavaScript]
cover: https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202204131011233.png
---
# new操作符实现原理

new ：实例化一个对象，继承到构造函数的属性和方法

```js
function Foo(e){
      this.name = "dokidoki";
      this.age = e;
    }
console.log(new Foo(18))
```

打印结果为：![image-20220213213033567](https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202203301454343.png)

![image-20220213221154250](https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202203301455976.png)

## 前置知识：

一篇文章看懂_proto_和prototype的关系及区别:https://www.jianshu.com/p/7d58f8f45557

![image-20220213215411156](https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202203301458020.png)

prototype:显式原型链

每一个函数在创建之后都会自动拥有的属性，这个属性是一个指针，指向函数的原型对象；且只有函数才有

所以说 **Person.prototype就是原型对象**，也就是实例person1和person2的原型。

原型对象的好处是可以让所有对象实例共享它所包含的属性和方法

所以构造函数和原型之间的关系为：

![image-20220213220011593](https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202203301457474.png)

`__proto__`：隐式原型链

Foo.prototype -->{constructor: f Foo()}

[[prototype]]私有属性，通过proto方法访问

**实例的隐式原型指向这个对象的函数的prototype(显式原型)**

```js
function abc(){

this.name = 'dokidoki'

}
let test = new abc()
test.__proto__ = abc.prototype
```

## 实现原理：

```js
function Foo(e){
      this.name = "dokidoki";
      this.age = e;
      // 如果在函数中使用了return值为object类型
      return {}
    }
function objectFactory(){
      // 1.因为new出来的实例时一个对象，所以创建一个对象
      const obj = {}
      // 3.拿到当前参数里的第一位 Foo，也就是构造函数
      const Constructor = [].shift.call(arguments)//shift拿到数组的第一项
      // const [Constructor,...args] = [...arguments];//得到argums参数里的第一个值，Foo
      
      // 5.原型连接，将构造函数的原型给到对象的私有属性上
      obj.__proto__ = Constructor.prototype
      // 4.把构造函数的方法都添加到obj对象中
      const ret = Constructor.apply(obj,arguments)
      // 2.返回一个对象
      // return obj
      // 优化 ------- 判断返回结果
      return typeof ret === 'object' ? ret : obj 
}
    // console.log(new Foo(18))
    console.log(objectFactory(Foo,18));
```