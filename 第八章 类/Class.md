## Class

#### ES5的仿类结构
```javascript
function PersonType(name) {
    this.name = name;
}
PersonType.prototype.sayName = function() {
    console.log(this.name);
};
let person = new PersonType("Nicholas");
person.sayName(); // 输出 "Nicholas"
```

#### ES6基本类声明
```javascript
class PersonClass {
    // 等价于 PersonType 构造器
    constructor(name) {
        this.name = name;
    }
    // 等价于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
}
let person = new PersonClass("Nicholas");
person.sayName(); // 输出 "Nicholas"
console.log(person instanceof PersonClass); // true
console.log(person instanceof Object); // true
console.log(typeof PersonClass); // "function"
console.log(typeof PersonClass.prototype.sayName); // "function"
```

* 注意：
  1. 类声明不会被提升，这与函数定义不同。类声明的行为与 let 相似，因此在程序的执行到达
     声明处之前，类会存在于暂时性死区内。
  2. 类声明中的所有代码会自动运行在严格模式下，并且也无法退出严格模式。
  3. 类的所有方法都是不可枚举的，这是对于自定义类型的显著变化，后者必须用
  Object.defineProperty() 才能将方法改变为不可枚举。
  4. 类的所有方法内部都没有 [[Construct]] ，因此使用 new 来调用它们会抛出错误。
  5. 调用类构造器时不使用 new ，会抛出错误。
  6. 试图在类的方法内部重写类名，会抛出错误。

#### name 属性
由于本质上，ES6 的类只是 ES5 的构造函数的一层包装，所以函数的许多特性都被Class继承，包括name属性。
```javascript
class Point {}
Point.name // "Point"
```
name属性总是返回紧跟在class关键字后面的类名。


#### Class 的静态方法
类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上`static`关键字，就表示该方法不会被实例继承，而是**直接通过类来调用**，这就称为“静态方法”。
```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function
```

- 注意，如果静态方法包含`this`关键字，这个`this`指的是类，而不是实例。
- 静态方法可以与非静态方法重名。
```javascript
class Foo {
  static bar () {
    this.baz();
  }
  static baz () {
    console.log('hello');
  }
  baz () {
    console.log('world');
  }
}

Foo.bar() // hello
const foo = new Foo();
foo.baz(); // world
```

- 父类的静态方法，可以被子类继承。
- 静态方法也是可以从`super`对象上调用的。
```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod() // 'hello'
```

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';
  }
}

Bar.classMethod() // "hello, too"
```

#### Class 的静态属性和实例属性
静态属性指的是`Class`本身的属性，即`Class.propName`，而不是定义在实例对象（`this`）上的属性。
```javascript
class Foo {
}

Foo.prop = 1;
Foo.prop // 1
```
目前，只有这种写法可行，因为 ES6 明确规定，Class 内部只有静态方法，没有静态属性。

#### new.target 属性
`new`是从构造函数生成实例对象的命令。ES6为`new`命令引入了一个`new.target`属性，该属性一般用在构造函数之中，返回`new`命令作用于的那个构造函数。如果构造函数不是通过`new`命令调用的，`new.target`会返回`undefined`，因此这个属性可以用来确定构造函数是怎么调用的。
```javascript
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}

// 另一种写法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}

var person = new Person('张三'); // 正确
var notAPerson = Person.call(person, '张三');  // 报错
```

- `Class`内部调用`new.target`，返回当前`Class`。
- 子类继承父类时，`new.target`会返回子类。
```javascript
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    // ...
  }
}

class Square extends Rectangle {
  constructor(length) {
    super(length, length);
  }
}

var obj = new Square(3); // 输出 false
```
  
## 类的继承
- 子类必须在`constructor`方法中调用`super`方法，否则新建实例时会报错。这是因为子类自己的`this`对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用`super`方法，子类就得不到`this`对象。而ES5的继承，实质是先创造子类的实例对象`this`，然后再将父类的方法添加到`this`上面。
- 如果子类没有定义`constructor`方法，这个方法会被默认添加。
- 在子类的构造函数中，只有调用`super`之后，才可以使用`this`关键字。
```javascript
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }
    getArea() {
        return this.length * this.width;
    }
}
class Square extends Rectangle {
    constructor(length) {
        // 与Rectangle.call(this, length, length)相同
        // 调用父类的constructor(length, width)
        super(length, length);
    }
}
var square = new Square(3);
console.log(square.getArea()); // 9
console.log(square instanceof Square); // true
console.log(square instanceof Rectangle); // true
```

#### `Object.getPrototypeOf()`
`Object.getPrototypeOf`方法可以用来从子类上获取父类。可以使用这个方法判断，一个类是否继承了另一个类。
```javascript
Object.getPrototypeOf(ColorPoint) === Point // true
```

#### `super`关键字
- `super`作为函数调用时，代表父类的构造函数。
- `super`作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。
```javascript
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
    // 相当于A.prototype.p()
  }
}

let b = new B();
```

- 由于`super`指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过`super`调用的。
```javascript
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

- 属性如果定义在父类的原型对象上，`super`就可以取到。
```javascript
class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x) // 2
  }
}

let b = new B();
```

- 在子类普通方法中通过`super`调用父类的方法时，方法内部的`this`指向当前的子类实例。
```javascript
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print();
    // 相当于super.print.call(this)
  }
}

let b = new B();
b.m() // 2
```

- 如果通过`super`对某个属性赋值，这时`super`就是`this`，赋值的属性会变成子类实例的属性。
```javascript
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined，读的是A.prototype.x
    console.log(this.x); // 3
  }
}

let b = new B();
```

- `super`作为对象，用在静态方法之中，这时`super`将指向父类，而不是父类的原型对象。
```javascript
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```

- 在子类的静态方法中通过`super`调用父类的方法时，方法内部的`this`指向当前的子类，而不是子类的实例。
```javascript
class A {
  constructor() {
    this.x = 1;
  }
  static print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  static m() {
    super.print();
  }
}

B.x = 3;
B.m() // 3
```

#### 类的`prototype`属性和`__proto__`属性
每一个对象都有`__proto__`属性，指向对应的构造函数的`prototype`属性。`Class`作为构造函数的语法糖，同时有`prototype`属性和`__proto__`属性，因此同时存在两条继承链：
1. 子类的`__proto__`属性，表示构造函数的继承，总是指向父类。
2. 子类`prototype`属性的`__proto__`属性，表示方法的继承，总是指向父类的prototype属性。
```javascript
class A {}

class B extends A {}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

这样的结果是因为，类的继承是按照下面的模式实现的：
```javascript
class A {}

class B {}

// B 的实例继承 A 的实例
Object.setPrototypeOf(B.prototype, A.prototype);

// B 继承 A 的静态属性
Object.setPrototypeOf(B, A);

const b = new B();
```

`Object.setPrototypeOf`方法的实现：
```javascript
Object.setPrototypeOf = function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```

这两条继承链，可以这样理解：作为一个对象，子类（B）的原型（`__proto__`属性）是父类（A）；作为一个构造函数，子类（B）的原型对象（`prototype`属性）是父类的原型对象（`prototype`属性）的实例。

不存在任何继承时：
```javascript
class A {}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true
```

#### 实例的`__proto__`属性
子类实例的`__proto__`属性的`__proto__`属性，指向父类实例的`__proto__`属性。也就是说，子类的原型的原型，是父类的原型。
```javascript
class A {
  print() {
    console.log("a");
  }
}

class B extends A {}

const a = new A();
const b = new B();

console.log(b.__proto__ === a.__proto__); // false
console.log(b.__proto__.__proto__ === a.__proto__);  // true
```

通过子类实例的`__proto__.__proto__`属性，可以修改父类实例的行为：
```javascript
b.__proto__.__proto__.print = function () {
  console.log("b");
};
```

#### `Mixin`模式的实现
将多个类的接口“混入”（mix in）另一个类：
```javascript
function mix(...mixins) {
  class Mix {}

  for (let mixin of mixins) {
    copyProperties(Mix.prototype, mixin); // 拷贝实例属性
    copyProperties(Mix.prototype, Reflect.getPrototypeOf(mixin)); // 拷贝原型属性
  }

  return Mix;
}

function copyProperties(target, source) {
  for (let key of Reflect.ownKeys(source)) {
    if ( key !== "constructor"
      && key !== "prototype"
      && key !== "name"
    ) {
      let desc = Object.getOwnPropertyDescriptor(source, key);
      Object.defineProperty(target, key, desc);
    }
  }
}
```