## Proxy
- `Proxy`用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种“元编程”（`meta programming`），即对编程语言进行编程。
- `Proxy`可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。
```javascript
const obj = new Proxy({}, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});
// 对一个空对象架设了一层拦截，重定义了属性的读取（get）和设置（set）行为。

obj.count = 1
//  setting count!
++obj.count
//  getting count!
//  setting count!
```

`new Proxy()`表示生成一个`Proxy`实例，`target`参数表示所要拦截的目标对象，`handler`参数也是一个对象，用来定制拦截行为。
```javascript
const proxy = new Proxy(target, handler);
```

如果`handler`没有设置任何拦截，那就等同于直接通向原对象。
```javascript
const target = {};
const handler = {};
const proxy = new Proxy(target, handler);
proxy.a = 'b';
target.a // "b"
// 访问proxy就等同于访问target。
```

一个技巧是将`Proxy`对象，设置到`object.proxy`属性，从而可以在`object`对象上调用。
```javascript
const object = { proxy: new Proxy(target, handler) };
```

同一个拦截器函数，可以设置拦截多个操作：
```javascript
const handler = {
  get: function(target, name) {
    if (name === 'prototype') {
      return Object.prototype;
    }
    return 'Hello, ' + name;
  },

  apply: function(target, thisBinding, args) {
    return args[0];
  },

  construct: function(target, args) {
    return {value: args[1]};
  }
};

const fproxy = new Proxy(function(x, y) {
  return x + y;
}, handler);

fproxy(1, 2) // 1
new fproxy(1, 2) // {value: 2}
fproxy.prototype === Object.prototype // true
fproxy.foo === "Hello, foo" // true
```

##### 下面是`Proxy`支持的拦截操作一览，一共13种：
- `get(target, propKey, receiver)`：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。
- `set(target, propKey, value, receiver)`：拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。
- `has(target, propKey)`：拦截`propKey in proxy`的操作，返回一个布尔值。
- `deleteProperty(target, propKey)`：拦截`delete proxy[propKey]`的操作，返回一个布尔值。
- `ownKeys(target)`：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象自身的可遍历属性。
- `getOwnPropertyDescriptor(target, propKey)`：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
- `defineProperty(target, propKey, propDesc)`：拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
- `preventExtensions(target)`：拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
- `getPrototypeOf(target)`：拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
- `isExtensible(target)`：拦截`Object.isExtensible(proxy)`，返回一个布尔值。
- `setPrototypeOf(target, proto)`：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- `apply(target, object, args)`：拦截`Proxy`实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。
- `construct(target, args)`：拦截`Proxy`实例作为构造函数调用的操作，比如`new proxy(...args)`。

### `Proxy`实例的方法
#### `get()`
`get`方法用于拦截某个属性的读取操作，可以接受三个参数，依次为目标对象、属性名和`proxy`实例本身（严格地说，是操作行为所针对的对象），其中最后一个参数可选。
```javascript
const person = {
  name: "张三"
};

const proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.");
    }
  }
});

proxy.name // "张三"
proxy.age // Property "age" does not exist.
```

`get`方法可以继承。
```javascript
let proto = new Proxy({}, {
  get(target, propertyKey, receiver) {
    console.log('GET ' + propertyKey);
    return target[propertyKey];
  }
});

let obj = Object.create(proto);
obj.foo // "GET foo"
```

下面的例子使用get拦截，实现数组读取负数的索引：
```javascript
function createArray(...elements) {
  const handler = {
    get(target, propKey, receiver) {
      const index = Number(propKey);
      if (index < 0) {
        propKey = String(target.length + index);
      }
      return Reflect.get(target, propKey, receiver);
    }
  };
  return new Proxy([...elements], handler);
}

const arr = createArray('a', 'b', 'c');
console.log(arr[-1]); // c
```

`get`方法的第三个参数总是指向原始的读操作所在的那个对象。
```javascript
const proxy = new Proxy({}, {
  get: function(target, property, receiver) {
    return receiver;
  }
});
proxy.getReceiver === proxy // true
```

如果一个属性不可写（`writable`）及不可配置（`configurable`）时，则返回的值必须与该目标属性的值相同，否则通过`Proxy`对象访问该属性会报错。
```javascript
const target = Object.defineProperties({}, {
  foo: {
    value: 123,
    writable: false,
    configurable: false
  },
});

const handler = {
  get(target, propKey) {
    return 'abc';
  }
};

const proxy = new Proxy(target, handler);

proxy.foo
// TypeError: Invariant check failed
```

#### `set()`
`set`方法用来拦截某个属性的赋值操作，可以接受四个参数，依次为目标对象、属性名、属性值和`Proxy`实例本身，其中最后一个参数可选。
```javascript
const validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }
    obj[prop] = value;
  }
};

const person = new Proxy({}, validator);

person.age = 100;
person.age = 'young' // 报错
person.age = 300 // 报错
```

下面是`set`方法第四个参数的例子：
```javascript
const handler = {
  set: function(obj, prop, value, receiver) {
    obj[prop] = receiver;
  }
};
const proxy = new Proxy({}, handler);
proxy.foo = 'bar';
proxy.foo === proxy // true
```

如果目标对象自身的某个属性不可写及不可配置时，那么`set`方法将不起作用。
```javascript
const obj = {};
Object.defineProperty(obj, 'foo', {
  value: 'bar',
  writable: false,
});

const handler = {
  set: function(obj, prop, value) {
    obj[prop] = value;
  }
};

const proxy = new Proxy(obj, handler);
proxy.foo = 'baz';
proxy.foo // "bar"
```

#### `apply()`
- `apply`方法拦截函数的调用、`call`、`apply`和`Reflect.apply()`操作。
- `apply`方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（`this`）和目标对象的参数数组。
```javascript
const handler = {
  apply (target, ctx, args) {
    return Reflect.apply(...arguments);
  }
};
```

```javascript
const twice = {
  apply (target, ctx, args) {
    return Reflect.apply(...arguments) * 2;
  }
};
function sum (left, right) {
  return left + right;
};
const proxy = new Proxy(sum, twice);
proxy(1, 2) // 6
proxy.call(null, 5, 6) // 22
proxy.apply(null, [7, 8]) // 30
Reflect.apply(proxy, null, [9, 10]) // 38
```

#### `has()`
- `has`方法用来拦截`HasProperty`操作，即判断对象是否具有某个属性时，这个方法会生效。
- `has`方法可以接受两个参数，分别是目标对象、需查询的属性名。
```javascript
const handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false;
    }
    return key in target;
  }
};
const target = { _prop: 'foo', prop: 'foo' };
const proxy = new Proxy(target, handler);
'_prop' in proxy // false
```

如果原对象不可配置或者禁止扩展，这时`has`拦截会报错。
```javascript
var obj = { a: 10 };
Object.preventExtensions(obj);

var p = new Proxy(obj, {
  has: function(target, prop) {
    return false;
  }
});

'a' in p // TypeError is thrown
```

- **`has`方法拦截的是`HasProperty`操作**，而不是`HasOwnProperty`操作，即`has`方法不判断一个属性是对象自身的属性，还是继承的属性。
- 虽然`for...in`循环也用到了`in`运算符，但是`has`拦截对`for...in`循环不生效。

#### `construct()`
- `construct`方法用于拦截`new`命令
- `construct`方法可以接受两个参数，分别是目标对象、构造函数的参数对象。
- `construct`方法返回的必须是一个对象，否则会报错。
```javascript
const handler = {
  construct (target, args, newTarget) {
    return new target(...args);
  }
};
```

```javascript
const p = new Proxy(function () {}, {
  construct: function(target, args) {
    console.log('called: ' + args.join(', '));
    return { value: args[0] * 10 };
  }
});

(new p(1)).value
// "called: 1"
// 10
```

#### `deleteProperty()`
`deleteProperty`方法用于拦截`delete`和`Reflect.deleteProperty()`操作，如果这个方法抛出错误或者返回false，当前属性就无法被delete命令删除。
```javascript
const handler = {
  deleteProperty (target, key) {
    invariant(key, 'delete');
    return true;
  }
};
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}

const target = { _prop: 'foo' };
const proxy = new Proxy(target, handler);
delete proxy._prop
// Error: Invalid attempt to delete private "_prop" property
```

如果目标对象自身的不可配置（`configurable`）的属性，不能被`deleteProperty`方法删除，否则报错。

#### `defineProperty()`
`defineProperty`方法拦截了`Object.defineProperty`和`Reflect.defineProperty()`操作。
```javascript
const handler = {
  defineProperty (target, key, descriptor) {
    return false;
  }
};
const target = {};
const proxy = new Proxy(target, handler);
proxy.foo = 'bar' // 不会生效
```

如果目标对象不可扩展（`extensible`），则`defineProperty`不能增加目标对象上不存在的属性，否则会报错。另外，如果目标对象的某个属性不可写（`writable`）或不可配置（`configurable`），则`defineProperty`方法不得改变这两个设置。

#### `getOwnPropertyDescriptor()`
`getOwnPropertyDescriptor`方法拦截`Object.getOwnPropertyDescriptor()`和`Reflect.getOwnPropertyDescriptor()`，返回一个属性描述对象或者`undefined`。
```javascript
const handler = {
  getOwnPropertyDescriptor (target, key) {
    if (key[0] === '_') {
      return;
    }
    return Object.getOwnPropertyDescriptor(target, key);
  }
};
const target = { _foo: 'bar', baz: 'tar' };
const proxy = new Proxy(target, handler);
Object.getOwnPropertyDescriptor(proxy, 'wat')
// undefined
Object.getOwnPropertyDescriptor(proxy, '_foo')
// undefined
Object.getOwnPropertyDescriptor(proxy, 'baz')
// { value: 'tar', writable: true, enumerable: true, configurable: true }
```

#### `getPrototypeOf()`
`getPrototypeOf`方法主要用来拦截获取对象原型。具体来说，拦截下面这些操作：
- `Object.prototype.__proto__`
- `Object.prototype.isPrototypeOf()`
- `Object.getPrototypeOf()`
- `Reflect.getPrototypeOf()`
- `instanceof`

```javascript
const proto = {};
const p = new Proxy({}, {
  getPrototypeOf(target) {
    return proto;
  }
});
Object.getPrototypeOf(p) === proto // true
```

`getPrototypeOf`方法的返回值必须是对象或者`null`，否则报错。另外，如果目标对象不可扩展（`extensible`），`getPrototypeOf`方法必须返回目标对象的原型对象。

#### `isExtensible()`
- `isExtensible`方法拦截`Object.isExtensible()`和`Reflect.isExtensible()`操作。
- 该方法只能返回布尔值，否则返回值会被自动转为布尔值。
- 这个方法有一个强限制，它的返回值必须与目标对象的`isExtensible`属性保持一致，否则就会抛出错误。
```javascript
const p = new Proxy({}, {
  isExtensible: function(target) {
    console.log("called");
    return true;
  }
});

Object.isExtensible(p)
// "called"
// true
```

#### `ownKeys()`
`ownKeys`方法用来拦截对象自身属性的读取操作。具体来说，拦截以下操作：
- `Object.getOwnPropertyNames()`
- `Object.getOwnPropertySymbols()`
- `Object.keys()`
- `Reflect.ownKeys()`
- `for...in循环`

```javascript
const target = {
  a: 1,
  b: 2,
  _c: 3,
};

const handler = {
  ownKeys(target) {
    return Reflect.ownKeys(target).filter(key => key[0] !== '_');
  }
};

const proxy = new Proxy(target, handler);

Object.keys(proxy)
// [ 'a' , 'b']
```

使用`Object.keys`方法时，有三类属性会被`ownKeys`方法自动过滤，不会返回：
- 目标对象上不存在的属性
- 属性名为`Symbol`值
- 不可遍历（`enumerable`）的属性

```javascript
const target = {
  a: 1,
  b: 2,
  c: 3,
  [Symbol.for('secret')]: '4',
};

Object.defineProperty(target, 'key', {
  enumerable: false,
  configurable: true,
  writable: true,
  value: 'static'
});

const handler = {
  ownKeys(target) {
    return ['a', 'd', Symbol.for('secret'), 'key'];
  }
};

const proxy = new Proxy(target, handler);

Object.keys(proxy)
// ['a']
```

```javascript
const obj = { hello: 'world' };
const proxy = new Proxy(obj, {
  ownKeys() {
    return ['a', 'b'];
  }
});

for (let key in proxy) {
  console.log(key);
  // ownkeys指定只返回a和b属性，由于obj没有这两个属性，因此for...in循环不会有任何输出
}
```

- `ownKeys`方法返回的数组成员，只能是字符串或`Symbol`值。如果有其他类型的值，或者返回的根本不是数组，就会报错。
- 如果目标对象自身包含不可配置的属性，则该属性必须被`ownKeys`方法返回，否则报错。
- 如果目标对象是不可扩展的（`non-extensition`），这时`ownKeys`方法返回的数组之中，必须包含原对象的所有属性，且不能包含多余的属性，否则报错。

#### `preventExtensions()`
- `preventExtensions`方法拦截`Object.preventExtensions()`和`Reflect.preventExtensions()`。该方法必须返回一个布尔值，否则会被自动转为布尔值。
- 这个方法有一个限制，**只有目标对象不可扩展时（即`Object.isExtensible(proxy)`为`false`）**，`proxy.preventExtensions`才能返回`true`，否则会报错。
```javascript
const p = new Proxy({}, {
  preventExtensions: function(target) {
    console.log('called');
    Object.preventExtensions(target);
    return true;
  }
});

Object.preventExtensions(p)
// "called"
// true
```

#### `setPrototypeOf()`
- `setPrototypeOf`方法主要用来拦截`Object.setPrototypeOf()`和`Reflect.setPrototypeOf()`方法。
- 该方法只能返回布尔值，否则会被自动转为布尔值。
- 如果目标对象不可扩展（`extensible`），`setPrototypeOf`方法不得改变目标对象的原型。
```javascript
const handler = {
  setPrototypeOf (target, proto) {
    throw new Error('Changing the prototype is forbidden');
  }
};
const proto = {};
const target = function () {};
const proxy = new Proxy(target, handler);
Object.setPrototypeOf(proxy, proto);
// Error: Changing the prototype is forbidden
```

### `Proxy.revocable()`
- `Proxy.revocable`方法返回一个可取消的`Proxy`实例。
- `Proxy.revocable`方法返回一个对象，该对象的`proxy`属性是`Proxy`实例，`revoke`属性是一个函数，可以取消`Proxy`实例。
```javascript
const target = {};
const handler = {};

const {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```

### `this`问题
目标对象内部的`this`关键字会指向`Proxy`代理。
```javascript
const target = {
  m: function () {
    console.log(this === proxy);
  }
};
const handler = {};

const proxy = new Proxy(target, handler);

target.m() // false
proxy.m()  // true
```

由于`this`指向的变化，导致`Proxy`无法代理目标对象。
```javascript
const _name = new WeakMap();

class Person {
  constructor(name) {
    _name.set(this, name);
  }
  get name() {
    return _name.get(this);
  }
}

const jane = new Person('Jane');
jane.name // 'Jane'

const proxy = new Proxy(jane, {});
proxy.name // undefined
```

`this`绑定原始对象，就可以解决这个问题：
```javascript
const target = new Date('2015-01-01');
const handler = {
  get(target, prop) {
    if (prop === 'getDate') {
      return target.getDate.bind(target);
    }
    return Reflect.get(target, prop);
  }
};
const proxy = new Proxy(target, handler);

proxy.getDate() // 1
```