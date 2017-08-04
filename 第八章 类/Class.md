## Class

#### ES5的仿类结构
````ecmascript 6
    function PersonType(name) {
        this.name = name;
    }
    PersonType.prototype.sayName = function() {
        console.log(this.name);
    };
    let person = new PersonType("Nicholas");
    person.sayName(); // 输出 "Nicholas"
````

#### ES6基本类声明
````ecmascript 6
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
````
* 注意：
  1. 类声明不会被提升，这与函数定义不同。类声明的行为与 let 相似，因此在程序的执行到达
     声明处之前，类会存在于暂时性死区内。
  2. 类声明中的所有代码会自动运行在严格模式下，并且也无法退出严格模式。
  3. 类的所有方法都是不可枚举的，这是对于自定义类型的显著变化，后者必须用
  Object.defineProperty() 才能将方法改变为不可枚举。
  4. 类的所有方法内部都没有 [[Construct]] ，因此使用 new 来调用它们会抛出错误。
  5. 调用类构造器时不使用 new ，会抛出错误。
  6. 试图在类的方法内部重写类名，会抛出错误。
  
#### 类的继承
````ecmascript 6
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
            // 与 Rectangle.call(this, length, length) 相同
            super(length, length);
        }
    }
    var square = new Square(3);
    console.log(square.getArea()); // 9
    console.log(square instanceof Square); // true
    console.log(square instanceof Rectangle); // true
````
