
![](https://imgkr.cn-bj.ufileos.com/a5ade491-dde3-48ce-b91f-276c9e689eeb.png)

## 整体流程的介绍
1. 先定义整体结构
2. 实现promise.then
3. 实现promise.catch
4. 实现promise.resovle
5. 实现promise.reject
6. 实现promise.all
7. 实现promise.race

## 1.定义整体结构
1. 先写出构造函数，将Promise向外暴露

```js
/*
自定义Promise函数模块：IIFE
 */

(function (window) {
    /*
    Promise构造函数
    executor:执行器函数
     */
    function Promise(executor) {

    }

    // 向外暴露Promise
    window.Promise = Promise
})()

```

2. 添加Promise原型对象上的方法

```js
 /*
    Promise原型对象的then
    指定一个成功/失败的回调函数
    返回一个新的promise对象
     */
    Promise.prototype.then = function(onResolved,onRejected){

    }

    /*
    Promise原型对象的.catch
    指定一个失败的回调函数
    返回一个新的promise对象
     */
    Promise.prototype.catch = function(onRejected){

    }
```

3. 添加Promise函数对象上的方法

```js
/*
    Promise函数对象的resovle方法
    返回一个指定结果的promise对象
     */
    Promise.resolve = function(value){

    }

    /*
    Promise函数对象的reject方法
    返回一个指定reason的失败状态的promise对象
    */
    Promise.reject = function(value){

    }

    /*
    Promise函数对象的all方法
    返回一个promise对象，只有当所有promise都成功时返回的promise状态才成功
    */
    Promise.all = function(value){

    }

    /*
    Promise函数对象的race方法
    返回一个promise对象，状态由第一个完成的promise决定
    */
    Promise.race = function(value){

    }
```


![](https://imgkr.cn-bj.ufileos.com/0eadfeaf-5a1c-45d4-9f4a-f6ab39fbf53a.png)

