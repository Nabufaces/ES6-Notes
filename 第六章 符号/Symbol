## 创建符号值
* 符号没有字面量形式。
````ecmascript 6
   let firstName = Symbol();
   let person = {};
   person[firstName] = "Nicholas";
   console.log(person[firstName]); // "Nicholas"
````

## 共享符号值
* Symbol.for() 方法首先会搜索全局符号注册表，看是否存在一个键值为 "uid" 的符号值。
  若是，该方法会返回这个已存在的符号值；否则，会创建一个新的符号值，并使用该键值
  将其记录到全局符号注册表中，然后返回这个新的符号值。这就意味着此后使用同一个键
  值去调用 Symbol.for() 方法都将会返回同一个符号值：
````ecmascript 6
   let uid = Symbol.for("uid");
   let object = {
        [uid]: "12345"
   };
   console.log(object[uid]); // "12345"
   console.log(uid); // "Symbol(uid)"
   let uid2 = Symbol.for("uid");
   //第二次调用从全局符号注册表中将其检索了出来。
   console.log(uid === uid2); // true
   console.log(object[uid2]); // "12345"
   console.log(uid2); // "Symbol(uid)"
````
* Symbol.keyFor() 方法在全局符号注册表中根据符号值检索出对应的键值