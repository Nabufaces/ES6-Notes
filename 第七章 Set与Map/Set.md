## Set
* ES6 新增了 Set 类型，这是一种**无重复值的有序列表**
* Set 不会使用强制类型转换来判断值是否重复

#####创建Set
````ecmascript 6
    let set1 = new Set();
    set1.add(5);
    set1.add("5");
    console.log(set1.size); // 2
    
    let set2 = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
    console.log(set2); // Set { 1, 2, 3, 4, 5 }
````

#####方法
* has() 方法来测试某个值是否存在于 Set 中：
````ecmascript 6
    let set = new Set([1,"2"]);
    console.log(set.has(1));  //true
````

* delete() 方法来移除单个值：
````ecmascript 6
    let set = new Set([1,"2"]);
    set.delete(1);
````

* clear() 方法来将所有值从 Set 中移除：
````ecmascript 6
    let set = new Set([1,"2"]);
    set.clear();
    console.log(set.size);  // 0
````

* forEach() 方法会被传递一个回调函数，该回调接受三个参数：
  1. Set 中下个位置的值；
  2. 与第一个参数相同的值；
  3. 目标 Set 自身。
````ecmascript 6
    let set = new Set(["a","b"]);
    set.forEach((value, key, ownerSet) => {
       console.log(value + " " + key + " " + ownerSet);
    });
    //a a [object Set]
    //b b [object Set]
````

##### 将Set转换为数组
````ecmascript 6
    let set = new Set([1, 2, 3, 3, 3, 4, 5]),
         array = [...set];
    console.log(array); // [1,2,3,4,5]
````

## Weak Set
Weak Set 与正规 Set 之间最大的区别是对象的弱引用。此处有个例子说明了这种差异：
````ecmascript 6
    let set = new WeakSet(),
        key = {};
    // 将对象加入 set
    set.add(key);
    console.log(set.has(key)); // true
    // 移除对于键的最后一个强引用，同时从 Weak Set 中移除
    key = null;
    console.log(set.has(key));  //false
````
与Set差异：
1. 对于 WeakSet 的实例，只要调用 add() 、 has() 或 delete() 方法时传入了非对象的参数，
    就会抛出错误；
2. Weak Set 不可迭代，因此不能被用在 for-of 循环中；
3. Weak Set 无法暴露出任何迭代器（例如 keys() 与 values() 方法），因此没有任何编
    程手段可用于判断 Weak Set 的内容；
4. Weak Set 没有 forEach() 方法；
5. Weak Set 没有 size 属性。