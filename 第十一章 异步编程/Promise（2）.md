## 串联 Promise
每次对 then() 或 catch() 的调用实际上创建并返回了另一个 Promise ，仅当前一个
Promise 被完成或拒绝时，后一个 Promise 才会被决议。研究以下例子：
```ecmascript 6
    let p1 = new Promise(function(resolve, reject) {
        resolve(42);
    });
    p1.then(function(value) {
        console.log(value);
    }).then(function() {
        console.log("Finished");
    });
    /* 此代码输出：
       42
       Finished */
```

## 在 Promise 链中返回值
Promise 链的另一重要方面是能从一个 Promise 传递数据给下一个 Promise 的能力。传递给
执行器中的 resolve() 处理函数的参数，会被传递给对应 Promise 的完成处理函数，例如：
```ecmascript 6
    let p1 = new Promise(function(resolve, reject) {
        resolve(42);
    });
    p1.then(function(value) {
        console.log(value); // "42"
        return value + 1;
    }).then(function(value) {
        console.log(value); // "43"
    });
```

## 响应多个 Promise
 ES6 提供了能监视多个 Promise 的两个方法：
#### 1.Promise.all() 方法
Promise.all() 方法接收单个可迭代对象（如数组）作为参数，并返回一个 Promise 。这个可迭
代对象的元素都是 Promise ，**只有在它们都完成后，所返回的 Promise 才会被完成**。例如：
```ecmascript 6
   let p1 = new Promise(function(resolve, reject) {
        resolve(42);
   });
   let p2 = new Promise(function(resolve, reject) {
        resolve(43);
   });
   let p3 = new Promise(function(resolve, reject) {
        resolve(44);
   });
   let p4 = Promise.all([p1, p2, p3]);
   p4.then(function(value) {
       console.log(Array.isArray(value)); // true
       console.log(value[0]); // 42
       console.log(value[1]); // 43
       console.log(value[2]); // 44
   });
```
* 若传递给 Promise.all() 的任意 Promise 被拒绝了，那么方法所返回的 Promise 就会立刻被
拒绝，而不必等待其他的 Promise 结束。 拒绝处理函数总会接收到单个值，该值就是被拒绝的
 Promise 所返回的拒绝值。

#### 2.Promise.race() 方法
Promise.race() 方法一旦来源 Promise 中有一个被解决，所返回的 Promise 就会立刻被解决。例如：
```ecmascript 6
   let p1 = Promise.resolve(42);
   let p2 = new Promise(function(resolve, reject) {
        resolve(43);
   });
   let p3 = new Promise(function(resolve, reject) {
        resolve(44);
   });
   let p4 = Promise.race([p1, p2, p3]);
   p4.then(function(value) {
        console.log(value); // 42
   });
```

## 继承 Promise
```ecmascript 6
   class MyPromise extends Promise {
       // 使用默认构造器
       success(resolve, reject) {
            return this.then(resolve, reject);
       }
       failure(reject) {
            return this.catch(reject);
       }
   }
   let promise = new MyPromise(function(resolve, reject) {
        resolve(42);
   });
   promise.success(function(value) {
       console.log(value); // 42
   }).failure(function(value) {
       console.log(value);
   });
```
 