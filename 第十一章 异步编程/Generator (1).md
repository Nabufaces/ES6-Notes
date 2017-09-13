## 基本概念
```ecmascript 6
    function* helloWorldGenerator() {
      yield 'hello';
      yield 'world';
      return 'ending';
    }
    var hw = helloWorldGenerator();
```
* 调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是遍历器对象（Iterator Object）。
* Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行。
```
    hw.next()
    // { value: 'hello', done: false }
    
    hw.next()
    // { value: 'world', done: false }
    
    hw.next()
    // { value: 'ending', done: true }
    
    hw.next()
    // { value: undefined, done: true }
```
* 调用 Generator 函数，返回一个遍历器对象，代表 Generator 函数的内部指针。以后，每次调用遍历器对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield表达式后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束。

## yield 表达式   

遍历器对象的next方法的运行逻辑如下:
   
   （1）遇到yield表达式，就暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回的对象value属性值。yield表达式后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行。
   
   （2）下一次调用next方法时，再继续往下执行，直到遇到下一个yield表达式。
   
   （3）如果没有再遇到新的yield表达式，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。如果该函数没有return语句，则返回的对象的value属性值为undefined。

```ecmascript 6
    function* gen(){
        yield 123+456;
    }
    /* 上面代码中，yield后面的表达式不会立即求值，只会在next方法将指针移到这一句时，才会求值。*/
```
 
 #### yield与return相似与区别
    相似之处在于都能返回紧跟在语句后面的那个表达式的值。
    
    区别在于每次遇到yield，函数暂停执行，下一次再从该位置继续向后执行，而return语句不具备位置记忆的功能。
    
    一个函数里面，只能执行一次（或者说一个）return语句，但是可以执行多次（或者说多个）yield表达式。
    
    Generator函数可以不用yield表达式，这时就变成了一个单纯的暂缓执行函数。

* yield表达式如果用在另一个表达式之中，必须放在圆括号里面。
```ecmascript 6
    function* demo() {
         console.log('Hello' + yield); // SyntaxError
         console.log('Hello' + yield 123); // SyntaxError
       
         console.log('Hello' + (yield)); // OK
         console.log('Hello' + (yield 123)); // OK
    }
```

* yield表达式用作函数参数或放在赋值表达式的右边，可以不加括号。
```ecmascript 6
    function* demo() {
        foo(yield 'a', yield 'b'); // OK
        let input = yield; // OK
    }
```
