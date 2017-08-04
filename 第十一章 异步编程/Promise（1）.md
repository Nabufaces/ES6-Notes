## 回调模式
当 Node.js 被创建时，它通过普及回调函数编程模式提升了异步编程模型。回调函数模式类
似于事件模型，因为异步代码也会在后面的一个时间点才执行。不同之处在于需要调用的函
数（即回调函数）是作为参数传入的，如下所示：
```ecmascript 6
    readFile("example.txt", function(err, contents) {
        if (err) {
            throw err;
        }
        console.log(contents);
    });
    console.log("Hi!");
```
使用回调函数模式， readFile() 会立即开始执行，并在开始读取磁盘时暂停。这意味着
console.log("Hi!") 会在 readFile() 被调用后立即进行输出，要早于
console.log(contents) 的打印操作。当 readFile() 结束操作后，它会将回调函数以及相关
参数作为一个新的作业添加到作业队列的尾部。在之前的作业全部结束后，该作业才会执
行。

## Promise基础
Promise 是为异步操作的结果所准备的占位符。函数可以返回一个 Promise，而不必订阅一个
事件或向函数传递一个回调参数，就像这样：
```ecmascript 6
    // readFile 承诺会在将来某个时间点完成
    let promise = readFile("example.txt");
```
在此代码中， readFile() 实际上并未立即开始读取文件，这将会在稍后发生。此函数反而
会返回一个 Promise 对象以表示异步读取操作，因此你可以在将来再操作它。你能对结果进
行操作的确切时刻，完全取决于 Promise 的生命周期是如何进行的。

## Promise生命周期
每个 Promise 都会经历一个短暂的生命周期，初始为挂起态（ pending state），这表示异步
操作尚未结束。一个挂起的 Promise 也被认为是未决的（ unsettled ）。上个例子中的
Promise 在 readFile() 函数返回它的时候就是处在挂起态。一旦异步操作结束， Promise
就会被认为是已决的（ settled ），并进入两种可能状态之一：
1. 已完成（ fulfilled ）： Promise 的异步操作已成功结束；
2. 已拒绝（ rejected ）： Promise 的异步操作未成功结束，可能是一个错误，或由其他原
因导致。

* **then()** 方法在所有的 Promise 上都存在，并且接受两个参数。第一个参数是 Promise 被完
  成时要调用的函数，第二个参数则是 Promise 被拒绝时要调用的函数。
```ecmascript 6
    let promise = readFile("example.txt");
    promise.then(function(contents) {
        // 完成
        console.log(contents);
    }, function(err) {
        // 拒绝
        console.error(err.message);
    });
```
* Promise 也具有一个 catch() 方法，其行为等同于只传递拒绝处理函数给 then() 。

## 创建未决的Promise
新的 Promise 使用 Promise 构造器来创建，该执行器会被传递两个名为 resolve()与 
reject() 的函数作为参数。 resolve() 函数在执行器成功结束时被调用，用于示意该
Promise 已经准备好被决议（ resolved ），而 reject() 函数则表明执行器的操作已失败。
```ecmascript 6
// Node.js 范例
    let fs = require("fs");
    function readFile(filename) {
        return new Promise(function(resolve, reject) {
            // 触发异步操作
            fs.readFile(filename, { encoding: "utf8" }, function(err, contents) {
                // 检查错误
                if (err) {
                    reject(err);
                return;
                }
                // 读取成功
                resolve(contents);
            });
        });
    }
    let promise = readFile("example.txt");
    // 同时监听完成与拒绝
    promise.then(function(contents) {
        // 完成
        console.log(contents);
        }, function(err) {
        // 拒绝
        console.error(err.message);
    });
```
在此例中，fs.readFile() 异步调用被包装在一个 Promise 中。执行器要么传递错误对象给
reject() 函数，要么传递文件内容给 resolve() 函数。要记住执行器会在 readFile() 被调
用时立即运行。当 resolve() 或 reject() 在执行器内部被调用时，一个作业被添加到作业队
列中，以便决议（ **resolve** ）这个 Promise 。这被称为作业调度（ job scheduling ）。

## 创建已决的Promise
#### 1.使用 Promise.resolve()
Promise.resolve() 方法接受单个参数并会返回一个处于完成态的 Promise 。这意味着没有
任何作业调度会发生，并且你需要向 Promise 添加一个或更多的完成处理函数来提取这个参
数值。例如：
```ecmascript 6
    let promise = Promise.resolve(42);
    promise.then(function(value) {
        console.log(value); // 42
    });
```
* 若一个拒绝处理函数被添加到此 Promise ，该拒绝处理函数将永不会被调用，因为此 Promise
绝不可能再是拒绝态。

#### 2.使用 Promise.reject()
你也可以使用 Promise.reject() 方法来创建一个已拒绝的 Promise 。此方法像
Promise.resolve() 一样工作，区别是被创建的 Promise 处于拒绝态，如下：
```ecmascript 6
    let promise = Promise.reject(42);
    promise.catch(function(value) {
        console.log(value); // 42
    });
```
* 任何附加到这个 Promise 的拒绝处理函数都将会被调用，而完成处理函数则不会执行。



#### 3.非 Promise 的 Thenable
Promise.resolve() 与 Promise.reject() 都能接受非 Promise 的 thenable 作为参数。当传
入了非 Promise 的 thenable 时，这些方法会创建一个新的 Promise ，此 Promise 会在
then() 函数之后被调用。

当一个对象拥有一个能接受 resolve 与 reject 参数的 then() 方法，该对象就会被认为是
一个非 Promise 的 thenable ，就像这样：
```ecmascript 6
    let thenable = {
        then: function(resolve, reject) {
            resolve(42);
        }
    };
```
可以调用 Promise.resolve() 来将 thenable 转换为一个已完成的 Promise ：
```ecmascript 6
    let thenable = {
        then: function(resolve, reject) {
            resolve(42);
        }
    };
    let p1 = Promise.resolve(thenable);
    p1.then(function(value) {
        console.log(value); // 42
    });
```
在此例中， Promise.resolve() 调用了 thenable.then() ，确定了这个 thenable 的
Promise 状态：由于 resolve(42) 在 thenable.then() 方法内部被调用，这个 thenable 的
Promise 状态也就被设为已完成。一个名为 p1 的新 Promise 被创建为完成态，并从
thenable 中接收到了值（此处为 42 ），于是 p1 的完成处理函数就接收到一个值为 42 的
参数。

## Node.js 的拒绝处理
* 在 Node.js 中， process 对象上存在两个关联到 Promise 的拒绝处理的事件：
    * unhandledRejection ：当一个 Promise 被拒绝、而在事件循环的一个轮次中没有任何拒
   绝处理函数被调用，该事件就会被触发；
   * rejectionHandled ：若一个 Promise 被拒绝、并在事件循环的一个轮次之后再有拒绝处
   理函数被调用，该事件就会被触发。

这两个事件旨在共同帮助识别已被拒绝但未曾被处理 promise。

unhandledRejection 事件处理函数接受的参数是拒绝原因（常常是一个错误对象）以及已被
拒绝的 Promise 。以下代码展示了 unhandledRejection 的应用：
```ecmascript 6
   let rejected;
   process.on("unhandledRejection", function(reason, promise) {
       console.log(reason.message); // "Explosion!"
       console.log(rejected === promise); // true
   });
   rejected = Promise.reject(new Error("Explosion!"));
```