## Map
Map类型是键值对的有序列表，而键和值都可以是任意类型
````ecmascript 6
    let map = new Map();
    map.set("title", "Understanding ES6");
    map.set("year", 2016);
    console.log(map.get("title")); // "Understanding ES6"
    console.log(map.get("year")); // 2016
````

##### 创建Map
````ecmascript 6
    let map = new Map([["name", "Nicholas"], ["age", 25]]);
````

##### Map方法
* has(key) ：判断指定的键是否存在于 Map 中；
* delete(key) ：移除 Map 中的键以及对应的值；
* clear() ：移除 Map 中所有的键与值；
* forEach()：接受一个能接收三个参数的回调函数：
    1. Map 中下个位置的值；
    2. 该值所对应的键；
    3. 目标 Map 自身。
````ecmascript 6
    let map = new Map([ ["name", "Nicholas"], ["age", 25]]);
    map.forEach((value, key, ownerMap) => {
       // ...
    });
````

## Weak Map
* Weak Map类型是键值对的无序列表，其中键必须是非空的对象，值则允许是任意类型
* Weak Map 的键才是弱引用，而值不是。在 Weak Map 的值中存储对象
  会阻止垃圾回收，即使该对象的其他引用已全都被移除。