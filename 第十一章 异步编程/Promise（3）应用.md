### 应用1
* 10个请求同时发出，等待所有请求返回后再回调：

```ecmascript 6
    const urlList = [url1, url2, ...];
    const p1 = new Promise((resolve, reject) => {
        resolve(ajax());
    })
    ...
    let P = Promise.all([p1, p2, ...]);
    p.then(function(value){
        console.log(Array.isArray(value)) //true
    })
```

### 应用2
* promise实现jsonp：

```ecmascript 6
    function test(url) {
        const script = document.createElement('script');
        script.src = url;
        document.appendChild(script);
        
        return new Promise((resolve, reject) => {
            script.onload = (data) => {
                resolve(data);
            };
            script.onerror = (data) => {
                reject(data);
            }
        })
    }
```

