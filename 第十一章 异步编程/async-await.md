## async/await

#### 基本规则
* async 表示这是一个async函数，await只能用在这个函数里面。
* await 表示在这里等待promise返回结果了再继续执行。
* await 后面跟着的应该是一个promise对象（当然，其他返回值也可以，只是会立即执行）
* await等待的虽然是promise对象，但不必写.then(..)，直接可以得到返回值。
```ecmascript 6
    const sleep = (time) => {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                resolve('ok')
            },time);
        })
    };
    
    (async () => {
        let result = await sleep(3000);
        console.log(result);    //ok
    })();
```

#### 捕捉错误
* .catch(..)可以不用写，直接用标准的try catch语法捕捉错误。
```ecmascript 6
    const sleep = (time) => {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                reject('rejectError');    //模拟出错
            },time);
        })
    };
    
    (async () => {
        try {
            await sleep(1000);
            console.log('ok');  //不会执行
        } catch (err) {
            console.log(err);   //rejectError
        }
    })();
```

#### 循环多个await
* await可以写在for循环里。
* **await必须在async函数的上下文中的**。
```ecmascript 6
    //事件循环，一个请求响应后回调第二个请求
    const urlList = [...];
    (async () => {
        for(let i of urlList) {
            await fetch(i);
        }
    })();
```