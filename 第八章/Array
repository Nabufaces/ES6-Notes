## forEach()
````javascript
    arr.forEach(function callback(currentValue, index, array) {
        //your iterator
    }[, thisArg]);
````
## Array.of()
````ecmascript 6
    let items = new Array(2);  //创建长度为2的数组
    console.log(items.length); // 2
    console.log(items[0]); // undefined
    console.log(items[1]); // undefined
    
    items = Array.of(2);  //创建包含单个数据项的数组
    console.log(items.length); // 1
    console.log(items[0]); // 2
````
## Array.form()
````ecmascript 6
    let a = Array.from( [1,2,3] );
    let b = Array.from( [1,2,3], (value) => value + 1);
    console.log(a); // [ 1, 2, 3 ]
    console.log(b); // [ 2, 3, 4 ]
````

## find()和findIndex()
*  find() 方法会返回匹配的值，而 findIndex() 方法则会返回匹配位置的索引，均
    会在回调函数**第一次返回 true 时停止查找**。
````ecmascript 6
    let numbers = [25, 30, 35, 40, 45];
    console.log(numbers.find(n => n > 33)); // 35
    console.log(numbers.findIndex(n => n > 33)); // 2
````

## fill()
* 回调参数：
    1. 填充数据；
    2. 起始填充位置；
    3. 终止填充位置（不包含该位置自身）。
````ecmascript 6
    let numbers = [1, 2, 3, 4];
    numbers.fill(1);
    console.log(numbers.toString()); // 1,1,1,1
    
    let numbers = [1, 2, 3, 4];
    numbers.fill(1, 2);
    console.log(numbers.toString()); // 1,2,1,1
    
    let numbers = [1, 2, 3, 4];
    numbers.fill(0, 1, 3);
    console.log(numbers.toString()); // 1,0,0,1
````

## copyWithin()
* 回调参数：
    1. 起始复制位置；
    2. 被用来复制的数据的起始位置索引；
    3. 终止复制位置（不包含该位置自身）。
````ecmascript 6
    let numbers = [1, 2, 3, 4, 5];
    // 从索引 2 的位置开始粘贴
    // 从数组索引 0 的位置开始复制数据
    numbers.copyWithin(2, 0);
    console.log(numbers.toString()); // 1,2,1,2,3
    
    let numbers = [1, 2, 3, 4, 5];
    // 从索引 2 的位置开始粘贴
    // 从数组索引 1 的位置开始复制数据
    // 在遇到索引 3 时停止复制
    numbers.copyWithin(2, 1, 3);
    console.log(numbers.toString()); // 1,2,2,3,5
````