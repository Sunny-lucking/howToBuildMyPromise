\
![](https://imgkr.cn-bj.ufileos.com/a5ade491-dde3-48ce-b91f-276c9e689eeb.png)

## 整体流程的介绍
1. 先定义整体结构
2. 实现Promise构造函数
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

## 2. 实现Promise构造函数

1. 众所周知，构造函数里由resolve和reject方法，而且这两个方法会被传入executor，并且executor立即同步执行

```js
    /*
    Promise构造函数
    executor:执行器函数
     */
    function Promise(executor) {

        function resovle() {

        }
        function reject() {

        }

        // 立即同步执行executor
        executor(resovle,reject)
    }

```

2. 家喻户晓的是，每个promise都有一个状态可能为pending或resolved，rejected。因此需要添加个status，此外，当我们这样使用Promise的时候，


```js
// 例1
var promise = new Promise((resovle,reject)=>{
    
})

promise.then(resolve=>{},reject=>{})
```

这时执行到then，promise的状态还是pending，这时要把值和then里面的回调函数保存起来，所以需要个data和callbacks


```js
function Promise(executor) {

        var self = self

        self.status = 'pending' // 给promise对象指定status属性，初始值为pending
        self.data = undefined // 给promise对象指定一个存储结果的data
        self.callbacks = []  // 每个元素的结构：{onResolved(){}，onRejected(){}}


        function resovle() {

        }
        function reject() {

        }

        // 立即同步执行executor
        executor(resovle,reject)
    }
```
3. 妇孺皆知的是，在上面的例子1的基础上，当我们执行resovle（value）时，

```js
// 例2
var promise = new Promise((resolve,reject)=>{
    setTimeout(function () {
        resolve(1)
    })
})

promise.then(resolve=>{},reject=>{})

```

会把promise对象的status改为resovle，并且把value保存到data，然后执行之前保存的callbacks（上面说过当执行到then时，发现当前的promise是pending状态，会把then里的回调函数保存到promise的callbacks里）。

```js
function resolve(value) {
            // 如果当前状态不是pending，则不执行
            if(self.status !== 'pending'){
                return
            }
            // 将状态改为resolved
            self.status = 'resolved'
            // 保存value的值
            self.data = value

            // 如果有待执行的callback函数，立即异步执行回调函数onResolved
            if (self.callbacks.length>0){
                setTimeout(()=>{
                    self.callbacks.forEach(callbackObj=>{
                        callbackObj.onResolved(value)
                    })
                })
            }
        }
```

4. 我们还知道，promise的状态只能改变一次，因此当执行resolve的时候要判断是不是promise是不是pending的状态，否则是不能执行的

```js
function resolve(value) {
            // 如果当前状态不是pending，则不执行
            if(this.status !== 'pending'){
                return 
            }
            // 将状态改为resolved
            this.status = 'resolved'
            // 保存value的值
            this.data = value

            // 如果有待执行的callback函数，立即异步执行回调函数onResolved
            if (this.callbacks.length>0){
                setTimeout(()=>{
                    this.callbacks.forEach(callbackObj=>{
                        callbackObj.onResolved(value)
                    })
                })
            }
        }
```
5. 异曲同工之妙的是reject方法也是这个道理，因此这里无需赘述


```js
function reject(value) {
            // 如果当前状态不是pending，则不执行
            if(self.status !== 'pending'){
                return
            }
            // 将状态改为resolved
            self.status = 'rejected'
            // 保存value的值
            self.data = value

            // 如果有待执行的callback函数，立即异步执行回调函数onResolved
            if (self.callbacks.length>0){
                setTimeout(()=>{
                    self.callbacks.forEach(callbackObj=>{
                        callbackObj.onRejected(value)
                    })
                })
            }
        }
```
6. 我们又知道，当在执行executor的时候，如果执行异常的话，这个promise的状态会直接执行reject方法。

```js
// 例 3
var promise = new Promise((resolve,reject)=>{

    执行到这里出错了

    setTimeout(function () {
        resolve(1)
    })
})
```
要实现这个功能，我们可以在executor外让try catch来捕获


```js
        try{
            // 立即同步执行executor
            executor(resolve,reject)
        }catch (e) { // 如果执行器抛出异常，promise对象变为rejected状态
            reject(e)
        }
```

好了，现在来测试一下，为了测试，我们需要简单的实现下then，让它往callbacks里push then里的回调函数


```js
   Promise.prototype.then = function(onResolved,onRejected){
        // 假设当前状态还是pending状态，将回调函数保存起来
        this.callbacks.push({
            onResolved,
            onRejected
        })
    }
```

好了，现在测试下，发现成功。
```js
 let promise = new Promise((resolve,reject)=>{
        
        setTimeout(function () {
            // resolve(1)
            reject(1)
        },100)
    })

    promise.then(
        value=>{
            console.log("onResolved:",value);
        },
        reason=>{
            console.log("onRejected:",reason);
        }
    )

```


![](https://imgkr.cn-bj.ufileos.com/0eadfeaf-5a1c-45d4-9f4a-f6ab39fbf53a.png)

