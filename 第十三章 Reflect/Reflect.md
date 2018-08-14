## Reflect
`Reflect`对象的设计目的有这样几个：
1. 将`Object`对象的一些明显属于语言内部的方法（比如`Object.defineProperty`），放到`Reflect`对象上。现阶段，某些方法同时在`Object`和`Reflect`对象上部署，未来的新方法将只部署在`Reflect`对象上。也就是说，从`Reflect`对象上可以拿到语言内部的方法。
2. 修改某些`Object`方法的返回结果，让其变得更合理。比如`Object.defineProperty(obj, name, desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj, name, desc)`则会返回`false`。
```javascript
// 老写法
try {
  Object.defineProperty(target, property, attributes);
  // success
} catch (e) {
  // failure
}

// 新写法
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}
```
3. 让`Object`操作都变成函数行为。
```javascript
// 老写法
'assign' in Object // true

// 新写法
Reflect.has(Object, 'assign') // true
```
4. `Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，完成默认行为，作为修改行为的基础。也就是说，不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为。
```javascript
Proxy(target, {
  set: function(target, name, value, receiver) {
    const success = Reflect.set(target,name, value, receiver);
    if (success) {
      log(`property ${name} on ${target} set to ${value}`);
    }
    return success;
  }
});
```

有了`Reflect`对象以后，很多操作会更易读。

##### `Reflect`对象一共有13个静态方法：
- `Reflect.apply(target, thisArg, args)`
- `Reflect.construct(target, args)`
- `Reflect.get(target, name, receiver)`
- `Reflect.set(target, name, value, receiver)`
- `Reflect.defineProperty(target, name, desc)`
- `Reflect.deleteProperties(target, name)`
- `Reflect.has(target, name)`
- `Reflect.ownKeys(target)`
- `Reflect.isExtensible(target)`
- `Reflect.preventExtensions(target)`
- `Reflect.getOwnPropertyDescriptor(target, name)`
- `Reflect.getPrototypeOf(target)`
- `Reflect.setPrototypeOf(target, prototype)`

#### `Reflect.get(target, name, receiver)`
- `Reflect.get`方法查找并返回`target`对象的`name`属性，如果没有该属性，则返回`undefined`。
- 如果`name`属性部署了读取函数（`getter`），则读取函数的`this`绑定`receiver`。
- 如果第一个参数不是对象，`Reflect.get`方法会报错。
```javascript
const myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  },
}

const myReceiverObject = {
  foo: 4,
  bar: 4,
};

Reflect.get(myObject, 'foo') // 1
Reflect.get(myObject, 'bar') // 2
Reflect.get(myObject, 'baz', myReceiverObject) // 8
```

#### `Reflect.set(target, name, value, receiver)`
- `Reflect.set`方法设置`target`对象的`name`属性等于`value`。
- 如果`name`属性设置了赋值函数，则赋值函数的`this`绑定`receiver`。
- 如果第一个参数不是对象，`Reflect.set`会报错。
```javascript
const myObject = {
  foo: 1,
  set bar(value) {
    return this.foo = value;
  },
}

const myReceiverObject = {
  foo: 0,
};

myObject.foo // 1

Reflect.set(myObject, 'foo', 2);
myObject.foo // 2

Reflect.set(myObject, 'bar', 3)
myObject.foo // 3

Reflect.set(myObject, 'bar', 1, myReceiverObject);
myObject.foo // 3
myReceiverObject.foo // 1
```

如果`Proxy`对象和`Reflect`对象联合使用，前者拦截赋值操作，后者完成赋值的默认行为，如果传入了`receiver`，那么`Reflect.set`会触发`Proxy.defineProperty`拦截（`receiver`总是当前的`Proxy`实例，如果提供了`receiver`，相当于改变`this`对象的指向，将新的属性赋值在`Proxy`实例上面，导致会调用`defineProperty`）。
```javascript
const p = {
  a: 'a'
};

const handler = {
  set(target, key, value, receiver) {
    console.log('set');
    Reflect.set(target, key, value, receiver)
  },
  defineProperty(target, key, attribute) {
    console.log('defineProperty');
    Reflect.defineProperty(target, key, attribute);
  }
};

const obj = new Proxy(p, handler);
obj.a = 'A';
// set
// defineProperty
```

#### `Reflect.has(obj, name)`
- `Reflect.has`方法对应`name in obj`里面的`in`运算符。
- 如果第一个参数不是对象，`Reflect.has`会报错。
```javascript
const myObject = {
  foo: 1,
};

// 旧写法
'foo' in myObject // true

// 新写法
Reflect.has(myObject, 'foo') // true
```

#### `Reflect.deleteProperty(obj, name)`
- `Reflect.deleteProperty`方法等同于`delete obj[name]`，用于删除对象的属性。
- 该方法返回一个布尔值。如果删除成功，或者被删除的属性不存在，返回`true`；删除失败，被删除的属性依然存在，返回`false`。
- 如果第一个参数不是对象，`Reflect.deleteProperty`会报错。
```javascript
const myObj = { foo: 'bar' };

// 旧写法
delete myObj.foo;

// 新写法
Reflect.deleteProperty(myObj, 'foo');
```

#### `Reflect.construct(target, args)`
- `Reflect.construct`方法等同于`new target(...args)`，这提供了一种不使用`new`，来调用构造函数的方法。
```javascript
function Greeting(name) {
  this.name = name;
}

// new 的写法
const instance = new Greeting('张三');

// Reflect.construct 的写法
const instance = Reflect.construct(Greeting, ['张三']);
```

#### Reflect.getPrototypeOf(obj)
- `Reflect.getPrototypeOf`方法用于读取对象的`__proto__`属性，对应`Object.getPrototypeOf(obj)`。
- `Reflect.getPrototypeOf`和`Object.getPrototypeOf`的一个区别是，如果参数不是对象，`Object.getPrototypeOf`会将这个参数转为对象，然后再运行，而`Reflect.getPrototypeOf`会报错。
```javascript
const myObj = new FancyThing();

// 旧写法
Object.getPrototypeOf(myObj) === FancyThing.prototype;

// 新写法
Reflect.getPrototypeOf(myObj) === FancyThing.prototype;
```

#### `Reflect.setPrototypeOf(obj, newProto)`
- `Reflect.setPrototypeOf`方法用于设置目标对象的原型（`prototype`），对应`Object.setPrototypeOf(obj, newProto)`方法。它返回一个布尔值，表示是否设置成功。
```javascript
const myObj = {};

// 旧写法
Object.setPrototypeOf(myObj, Array.prototype);

// 新写法
Reflect.setPrototypeOf(myObj, Array.prototype);

myObj.length // 0
```

- 如果无法设置目标对象的原型（比如，目标对象禁止扩展），`Reflect.setPrototypeOf`方法返回`false`。
```javascript
Reflect.setPrototypeOf({}, null)
// true
Reflect.setPrototypeOf(Object.freeze({}), null)
// false
```

- 如果第一个参数不是对象，`Object.setPrototypeOf`会返回第一个参数本身，而`Reflect.setPrototypeOf`会报错。
```javascript
Object.setPrototypeOf(1, {})
// 1

Reflect.setPrototypeOf(1, {})
// TypeError: Reflect.setPrototypeOf called on non-object
```

- 如果第一个参数是`undefined`或`null`，`Object.setPrototypeOf`和`Reflect.setPrototypeOf`都会报错。
```javascript
Object.setPrototypeOf(null, {})
// TypeError: Object.setPrototypeOf called on null or undefined

Reflect.setPrototypeOf(null, {})
// TypeError: Reflect.setPrototypeOf called on non-object
```

#### `Reflect.apply(func, thisArg, args)`
`Reflect.apply`方法等同于`Function.prototype.apply.call(func, thisArg, args)`，用于绑定`this`对象后执行给定函数。
```javascript
const ages = [11, 33, 12, 54, 18, 96];

// 旧写法
const youngest = Math.min.apply(Math, ages);
const type = Object.prototype.toString.call(youngest);

// 新写法
const youngest = Reflect.apply(Math.min, Math, ages);
const type = Reflect.apply(Object.prototype.toString, youngest, []);
```

#### `Reflect.defineProperty(target, propertyKey, attributes)`
- `Reflect.defineProperty`方法基本等同于`Object.defineProperty`，用来为对象定义属性。
- 如果`Reflect.defineProperty`的第一个参数不是对象，就会抛出错误。
```javascript
function MyDate() {}

// 旧写法
Object.defineProperty(MyDate, 'now', {
  value: () => Date.now()
});

// 新写法
Reflect.defineProperty(MyDate, 'now', {
  value: () => Date.now()
});
```

#### `Reflect.getOwnPropertyDescriptor(target, propertyKey)`
- `Reflect.getOwnPropertyDescriptor`基本等同于`Object.getOwnPropertyDescriptor`，用于得到指定属性的描述对象。
- `Reflect.getOwnPropertyDescriptor`和`Object.getOwnPropertyDescriptor`的一个区别是，如果第一个参数不是对象，`Object.getOwnPropertyDescriptor`不报错，返回`undefined`，而`Reflect.getOwnPropertyDescriptor`会抛出错误，表示参数非法。
```javascript
const myObject = {};
Object.defineProperty(myObject, 'hidden', {
  value: true,
  enumerable: false,
});

// 旧写法
const theDescriptor = Object.getOwnPropertyDescriptor(myObject, 'hidden');

// 新写法
const theDescriptor = Reflect.getOwnPropertyDescriptor(myObject, 'hidden');
```

#### `Reflect.isExtensible (target)`
- `Reflect.isExtensible`方法对应`Object.isExtensible`，返回一个布尔值，表示当前对象是否可扩展。
- 如果参数不是对象，`Object.isExtensible`会返回`false`，因为非对象本来就是不可扩展的，而`Reflect.isExtensible`会报错。
```javascript
const myObject = {};

// 旧写法
Object.isExtensible(myObject) // true

// 新写法
Reflect.isExtensible(myObject) // true
```

#### Reflect.preventExtensions(target)
- `Reflect.preventExtensions`对应`Object.preventExtensions`方法，用于让一个对象变为不可扩展。它返回一个布尔值，表示是否操作成功。
```javascript
const myObject = {};

// 旧写法
Object.preventExtensions(myObject) // Object {}

// 新写法
Reflect.preventExtensions(myObject) // true
```

- 如果参数不是对象，`Object.preventExtensions`在 `ES5`环境报错，在`ES6`环境返回传入的参数，而`Reflect.preventExtensions`会报错。
```javascript
// ES5 环境
Object.preventExtensions(1) // 报错

// ES6 环境
Object.preventExtensions(1) // 1

// 新写法
Reflect.preventExtensions(1) // 
```

#### `Reflect.ownKeys (target)`
`Reflect.ownKeys`方法用于返回对象的所有属性，等同于`Object.getOwnPropertyNames(target).concat(Object.getOwnPropertySymbols(target))`之和。
```javascript
const myObject = {
  foo: 1,
  bar: 2,
  [Symbol.for('baz')]: 3,
  [Symbol.for('bing')]: 4,
};

// 旧写法
Object.getOwnPropertyNames(myObject)
// ['foo', 'bar']

Object.getOwnPropertySymbols(myObject)
//[Symbol(baz), Symbol(bing)]

// 新写法
Reflect.ownKeys(myObject)
// ['foo', 'bar', Symbol(baz), Symbol(bing)]
```

### 使用`Proxy`和`Reflect`实现观察者模式
```javascript
const observableQueue = new Set();

const observer = function(fn) {
  if(Reflect.apply(Object.prototype.toString, fn, []) === '[object Function]') {
    observableQueue.add(fn);
  } else {
    throw new TypeError('Parameters outside the function are passed in');
  }
}

const observable = obj => {
  return new Proxy(obj, {
    get(target, name, receiver) {
      const result = Reflect.get(target, name, receiver);
      console.log(`get name: ${name} value: ${result}`);
      return result
    },
    set(target, name, value, receiver) {
      console.log(`set name: ${name} value: ${value}`);
      const result = Reflect.set(target, name, value, receiver);
      // 执行观察者函数集
      observableQueue.forEach(fn => fn());
      return result;
    }
  })
}

// test
const person = observable({
  name: 'Jack',
  age: 21
});
const print = () => console.log(`watching~`);
observer(print);

person.name;
person.age = 22;
person.height = '180cm';
```