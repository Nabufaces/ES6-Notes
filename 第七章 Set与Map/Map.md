## `Map`
Map类型是键值对的有序列表，而键和值都可以是任意类型。也就是说，`Object`结构提供了“字符串—值”的对应，`Map`结构提供了“值—值”的对应，是一种更完善的`Hash`结构实现。如果你需要“键值对”的数据结构，`Map`比`Object`更合适。

```
let map = new Map();
map.set("title", "Understanding ES6");
map.set("year", 2016);
console.log(map.get("title")); // "Understanding ES6"
console.log(map.get("year")); // 2016
```

##### 创建`Map`
```
let map = new Map([["name", "Nicholas"], ["age", 25]]);
```

#### `Map`属性和方法
* `size`属性：返回`Map`结构的成员总数
* `set(key, value)`：该方法设置键名`key`对应的键值为`value`，然后返回整个`Map`结构。如果`key`已经有值，则键值会被更新
* `get(key)`：该方法读取`key`对应的键值，如果找不到`key`，返回`undefined`
* `has(key)`：判断指定的键是否存在于`Map`中
* `delete(key)`：移除`Map`中的键以及对应的值，返回`true`，如果删除失败，返回`false`
* `clear()`：移除`Map`中所有的键与值
* `keys()`：返回键名的遍历器
* `values()`：返回键值的遍历器
* `entries()`：返回所有成员的遍历器
* `forEach()`：接受一个能接收三个参数的回调函数：
    1. `Map`中下个位置的值；
    2. 该值所对应的键；
    3. 目标`Map`自身。
```
let map = new Map([ ["name", "Nicholas"], ["age", 25]]);
map.forEach((value, key, ownerMap) => {
   // ...
});
```

## `Weak Map`
* `Weak Map`类型是键值对的无序列表，其中键必须是非空的对象，值则允许是任意类型
* `Weak Map`的**键是弱引用，而值不是**。在`Weak Map`的值中存储对象会阻止垃圾回收，即使该对象的其他引用已全都被移除

```
let wm = new WeakMap();
let key = {};
const value = new Array(5 * 1024 * 1024);

wm.set(key, value);
key = null;
```
只要外部的引用消失，`Weak Map`内部的引用，就会自动被垃圾回收清除，因此不存在内存泄漏风险。