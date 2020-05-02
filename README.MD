
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
 // 例4
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

## 3. 实现then方法

尽人皆知的时，执行到then时，promise可能会是pending状态，此时就要把then里的回调函数保存起来，也可能会是resolved或者rejected状态，此时要执行onResolved或onRejected方法。

```js
Promise.prototype.then = function(onResolved,onRejected){

        var self = this

        if(self.status === 'pending'){
            // promise当前状态还是pending状态，将回调函数保存起来
            self.callbacks.push({
                onResolved(){onResolved(self.data)},
                onRejected(){onRejected(self.data)}
            })
        }else if(self.status === 'resolved'){
            setTimeout(()=>{
                onResolved(self.data)
            })
        }else{
            setTimeout(()=>{
                onResolved(self.data)
            })
        }

    }

```

而且我们知道，执行完then是要返回一个新的promise的，而新的promise的状态则由当前then的执行结果来确定。


```js
 Promise.prototype.then = function(onResolved,onRejected){

        var self = this

        return new Promise((resolve,reject)=>{
            if(self.status === 'pending'){
                // promise当前状态还是pending状态，将回调函数保存起来
                self.callbacks.push({
                    onResolved(){onResolved(self.data)},
                    onRejected(){onRejected(self.data)}
                })
            }else if(self.status === 'resolved'){
                setTimeout(()=>{
                    onResolved(self.data)
                })
            }else{
                setTimeout(()=>{
                    onResolved(self.data)
                })
            }
        })

    }
```
当当前的promise状态为resolved的时候，则当前执行的onresolved函数由三种情况
1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。

```js
// 例5
    let promise = new Promise((resolve,reject)=>{
        resolve(1)
    })

    promise.then(
        value=>{
            return value
        },
        reason=>{
            console.log("onRejected:",reason);
        }
    )
```
因此，我们可以这样实现


```js
Promise.prototype.then = function(onResolved,onRejected){

        var self = this

        return new Promise((resolve,reject)=>{
            if(self.status === 'pending'){
                // promise当前状态还是pending状态，将回调函数保存起来
                self.callbacks.push({
                    onResolved(){onResolved(self.data)},
                    onRejected(){onRejected(self.data)}
                })
            }else if(self.status === 'resolved'){
                setTimeout(()=>{
                    const result = onResolved(self.data)
                    if (result instanceof Promise){

                    } else {
                    // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
                        resolve(result)
                    }
                })
            }else{
                setTimeout(()=>{
                    onResolved(self.data)
                })
            }
        })

    }
```

1. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果，如代码所示，我们返回一个新的promise。这个新的promise执行了resolve，因此返回的promise的状态是resolved的
   
```js
// 例6
 let promise = new Promise((resolve,reject)=>{
        resolve(1)
    })

    promise.then(
        value=>{
            return new Promise((resolve,reject)=>{
                resolve(2)
            })
        },
        reason=>{
            console.log("onRejected:",reason);
        }
    )
```
因此我们可以这样实现

```js
Promise.prototype.then = function(onResolved,onRejected){

        var self = this

        return new Promise((resolve,reject)=>{
            if(self.status === 'pending'){
                // promise当前状态还是pending状态，将回调函数保存起来
                self.callbacks.push({
                    onResolved(){onResolved(self.data)},
                    onRejected(){onRejected(self.data)}
                })
            }else if(self.status === 'resolved'){
                setTimeout(()=>{
                    const result = onResolved(self.data)
                    if (result instanceof Promise){
                        // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
                        result.then(
                            value => {resolve(value)},
                            reason => {reject(reason)}
                        )
                    } else {
                        // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
                        resolve(result)
                    }
                })
            }else{
                setTimeout(()=>{
                    onResolved(self.data)
                })
            }
        })

    }

```
在这里说明一下：

```
result.then(
    value => {resolve(value)},
    reason => {reject(reason)}
)
```
由于我们在例6中执行了then里的resolve函数，
将会导致`value => {resolve(value)},`这个回调函数的执行，因此会把即将返回的新的promise的data设置为value，会把状态设置为resolved。

3. 如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected，我们可以用try catch来实现

```js
setTimeout(()=>{
    try{
        const result = onResolved(self.data)
        if (result instanceof Promise){
            // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
            result.then(
                value => {resolve(value)},
                reason => {reject(reason)}
            )
        } else {
            // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
            resolve(result)
        }
    }catch (e) {
      //  3.如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected
        reject(e)
    }
})
```
异曲同工之妙的是当当status === 'rejected'，道理一样

```js
 setTimeout(()=>{
      try{
          const result = onRejected(self.data)
          if (result instanceof Promise){
              // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
              result.then(
                  value => {resolve(value)},
                  reason => {reject(reason)}
              )
          } else {
              // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
              resolve(result)
          }
      }catch (e) {
          //  3.如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected
          reject(e)
      }
  })
```

到这里，我们发现当执行resolve的时候，`onResolved(self.data)`和`onRejected(self.data)`执行时也会跟上面一样的结果，可以说执行回调函数都要做以上判断，因此我们要将

```
 self.callbacks.push({
    onResolved(){onResolved(self.data)},
    onRejected(){onRejected(self.data)}
})
```
改成

```js
if(self.status === 'pending'){
// promise当前状态还是pending状态，将回调函数保存起来
self.callbacks.push({
    onResolved(){
        try{
            const result = onResolved(self.data)
            if (result instanceof Promise){
                // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
                result.then(
                    value => {resolve(value)},
                    reason => {reject(reason)}
                )
            } else {
                // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
                resolve(result)
            }
        }catch (e) {
            //  3.如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected
            reject(e)
        }
    },
```
到此，我们发现，相同的代码太多了，因此有必要封装一下

```
 function handle(callback) {
    try{
        const result = callback(self.data)
        if (result instanceof Promise){
            // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
            result.then(
                value => {resolve(value)},
                reason => {reject(reason)}
            )
        } else {
            // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
            resolve(result)
        }
    }catch (e) {
        //  3.如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected
        reject(e)
    }
}
```

这样以来就清爽了很多

```js
   Promise.prototype.then = function(onResolved,onRejected){

        var self = this

        return new Promise((resolve,reject)=>{
           /*
            调用指定回调函数的处理，根据执行结果。改变return的promise状态
             */
            function handle(callback) {
                try{
                    const result = callback(self.data)
                    if (result instanceof Promise){
                        // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
                        result.then(
                            value => {resolve(value)},
                            reason => {reject(reason)}
                        )
                    } else {
                        // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
                        resolve(result)
                    }
                }catch (e) {
                    //  3.如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected
                    reject(e)
                }
            }
            if(self.status === 'pending'){
                // promise当前状态还是pending状态，将回调函数保存起来
                self.callbacks.push({
                    onResolved(){
                        handle(onResolved)
                    },
                    onRejected(){
                        handle(onRejected)
                    }
                })
            }else if(self.status === 'resolved'){
                setTimeout(()=>{
                    handle(onResolved)
                })
            }else{ // 当status === 'rejected'
                setTimeout(()=>{
                    handle(onRejected)
                })
            }
        })

    }

```
另外，我们还知道，promise会发生直传透，例如

```js
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
```
运行结果： 1

解释：.then 或者 .catch 的参数期望是函数，传入非函数则会发生值穿透。值传透可以理解为，当传入then的不是函数的时候，这个then是无效的。

因此，要实现直传透这个特性，我们可以这样实现

 添加这两句来判断要不要发生值传透
```js
onResolved = typeof onResolved === 'function'? onResolved: value => value
onRejected = typeof onRejected === 'function'? onRejected: reason => {throw reason}
```

```js
 Promise.prototype.then = function(onResolved,onRejected){
        onResolved = typeof onResolved === 'function'? onResolved: value => value
        onRejected = typeof onRejected === 'function'? onRejected: reason => {throw reason}
        var self = this

        return new Promise((resolve,reject)=>{

            /*
            调用指定回调函数的处理，根据执行结果。改变return的promise状态
             */
            function handle(callback) {
                try{
                    const result = callback(self.data)
                    if (result instanceof Promise){
                        // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
                        result.then(
                            value => {resolve(value)},
                            reason => {reject(reason)}
                        )
                    } else {
                        // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
                        resolve(result)
                    }
                }catch (e) {
                    //  3.如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected
                    reject(e)
                }
            }
            if(self.status === 'pending'){
                // promise当前状态还是pending状态，将回调函数保存起来
                self.callbacks.push({
                    onResolved(){
                        handle(onResolved)
                    },
                    onRejected(){
                        handle(onRejected)
                    }
                })
            }else if(self.status === 'resolved'){
                setTimeout(()=>{
                    handle(onResolved)
                })
            }else{ // 当status === 'rejected'
                setTimeout(()=>{
                    handle(onRejected)
                })
            }
        })

    }

```


![](https://imgkr.cn-bj.ufileos.com/0eadfeaf-5a1c-45d4-9f4a-f6ab39fbf53a.png)

