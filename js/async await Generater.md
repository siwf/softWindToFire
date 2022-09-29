## Generator

> - 一是，function关键字与函数名之间有一个星号；
> - 二是，函数体内部使用yield语句，定义不同的内部状态（yield在英语里的意思就是“产出”）。



```typescript
function* g() {
    yield 'a';
    yield 'b';
    yield 'c';
    return 'ending';
}
g(); // 返回一个对象
```

> #### Generator 函数神奇之一：g()并不执行g函数

- g()并不会执行g函数，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是迭代器对象（Iterator Object）。

> #### Generator 函数神奇之二：分段执行

```typescript
function* g() {
    yield 'a';
    yield 'b';
    yield 'c';
    return 'ending';
}

var gen = g();
gen.next(); // 返回Object {value: "a", done: false}
```

- `gen.next()`返回一个非常非常简单的对象`{value: "a", done: false}`，'a'就是g函数执行到第一个yield语句之后得到的值，false表示g函数还没有执行完，只是在这暂停。
- 如果再写一行代码，还是`gen.next();`，这时候返回的就是`{value: "b", done: false}`，说明g函数运行到了第二个yield语句，返回的是该yield语句的返回值'b'。返回之后依然是暂停。
- 再写一行`gen.next()`;返回`{value: "c", done: false}`，再写一行`gen.next();`，返回`{value: "ending", done: true}`，这样，整个g函数就运行完毕了。



#### 提问：如果再写一行gen.next();呢？

答：返回{value: undefined, done: true}，这样没意义。

#### 提问：如果g函数没有return语句呢？

答：那么第三次.next()之后就返回{value: undefined, done: true}，这个第三次的next()唯一意义就是证明g函数全部执行完了。

#### 提问：如果g函数的return语句后面依然有yield呢？

答：js的老规定：return语句标志着该函数所有有效语句结束，return下方还有多少语句都是无效，白写。

#### 提问：如果g函数没有yield和return语句呢？

答：第一次调用next就返回{value: undefined, done: true}，之后也是{value: undefined, done: true}。

#### 提问：如果只有return语句呢？

答：第一次调用就返回{value: xxx, done: true}，其中xxx是return语句的返回值。之后永远是{value: undefined, done: true}。

#### 提问：下面代码会有什么结果？

```typescript
function* g() {
    var o = 1;
    yield o++;
    yield o++;
    yield o++;

}
var gen = g();

console.log(gen.next()); // 1

var xxx = g();

console.log(gen.next()); // 2
console.log(xxx.next()); // 1
console.log(gen.next()); // 3
```

答：见上面注释。每个迭代器之间互不干扰，作用域独立。

#### 继续提问：如果第二个yield o++;改成yield;会怎样？

答：那么指针指向这个yield的时候，返回{value: undefined, done: false}。

#### 继续提问：如果第二个yield o++;改成o++;yield;会怎样？

答：那么指针指向这个yield的时候，返回{value: undefined, done: false}，因为返回的永远是yield后面的那个表达式的值。

所以现在可以看出，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield语句（或return语句）为止。换言之，Generator函数是分段执行的，yield语句是暂停执行的标记，而next方法可以恢复执行。

总之，每调用一次Generator函数，就返回一个迭代器对象，代表Generator函数的内部指针。以后，每次调用迭代器对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield语句后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束。

**所以可以看出，Generator 函数的特点就是：**

- 1、分段执行，可以暂停
- 2、可以控制阶段和每个阶段的返回值
- 3、可以知道是否执行到结尾

#### yield语句

迭代器对象的next方法的运行逻辑如下。

（1）遇到yield语句，就暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回的对象的value属性值。

（2）下一次调用next方法时，再继续往下执行，直到遇到下一个yield语句。

（3）如果没有再遇到新的yield语句，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。

（4）如果该函数没有return语句，则返回的对象的value属性值为undefined。

yield语句与return语句既有相似之处，也有区别。

相似之处在于，都能返回紧跟在语句后面的那个表达式的值。

区别在于每次遇到yield，函数暂停执行，下一次再从该位置继续向后执行，而return语句不具备位置记忆的功能。一个函数里面，只能执行一次（或者说一个）return语句，但是可以执行多次（或者说多个）yield语句。正常函数只能返回一个值，因为只能执行一次return；Generator函数可以返回一系列的值，因为可以有任意多个yield。从另一个角度看，也可以说Generator生成了一系列的值，这也就是它的名称的来历（在英语中，generator这个词是“生成器”的意思）。

> 注意：yield语句只能用于function*的作用域，如果function*的内部还定义了其他的普通函数，则函数内部不允许使用yield语句。

注意：yield语句如果参与运算，必须用括号括起来。

```
console.log(3 + yield 4); // 语法错误
console.log(3 + (yield 4)); // 打印7
```

#### next方法可以有参数

一句话说，next方法参数的作用，是为上一个yield语句赋值。由于yield永远返回undefined，这时候，如果有了next方法的参数，yield就被赋了值，比如下例，原本a变量的值是0，但是有了next的参数，a变量现在等于next的参数，也就是11。

next方法的参数每次覆盖的一定是undefined。next在没有参数的时候，函数体里面写let xx = yield oo;是没意义的，因为xx一定是undefined。

```typescript
function* g() {
    var o = 1;
    var a = yield o++;
    console.log('a = ' + a);
    var b = yield o++;
}
var gen = g();

console.log(gen.next());
console.log('------');
console.log(gen.next(11));
```

得到：

首先说，console.log(gen.next());的作用就是输出了{value: 1, done: false}，注意var a = yield o++;，由于赋值运算是先计算等号右边，然后赋值给左边，所以目前阶段，只运算了yield o++，并没有赋值。

然后说，console.log(gen.next(11));的作用，首先是执行gen.next(11)，得到什么？首先：把第一个yield o++重置为11，然后，赋值给a，再然后，console.log('a = ' + a);，打印a = 11，继续然后，yield o++，得到2，最后打印出来。

从这我们看出了端倪：带参数跟不带参数的区别是，带参数的情况，首先第一步就是将上一个yield语句重置为参数值，然后再照常执行剩下的语句。总之，区别就是先有一步先重置值，接下来其他全都一样。

这个功能有很重要的语法意义，通过next方法的参数，就有办法在Generator函数开始运行之后，继续向函数体内部注入值。也就是说，可以在Generator函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。

提问：第一个.next()可以有参数么？
答：设这样的参数没任何意义，因为第一个.next()的前面没有yield语句。

#### for...of循环

for...of循环可以自动遍历Generator函数时生成的Iterator对象，且此时不再需要调用next方法。for...of循环的基本语法是：

```typescript
for (let v of foo()) {
  console.log(v);
}
```

其中foo()是迭代器对象，可以把它赋值给变量，然后遍历这个变量。

```typescript
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

let a = foo();

for (let v of a) {
  console.log(v);
}
// 1 2 3 4 5
```

上面代码使用for...of循环，依次显示5个yield语句的值。这里需要注意，一旦next方法的返回对象的done属性为true，for...of循环就会中止，且不包含该返回对象，所以上面代码的return语句返回的6，不包括在for...of循环之中。

下面是一个利用Generator函数和for...of循环，实现斐波那契数列的例子。

斐波那契数列是什么？它指的是这样一个数列 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144........
这个数列前两项是0和1，从第3项开始，每一项都等于前两项之和。

```typescript
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) { // 这里请思考：为什么这个循环不设定结束条件？
    [prev, curr] = [curr, prev + curr];
    yield curr;
  }
}

for (let n of fibonacci()) {
  if (n > 1000) {
    break;
  }
  console.log(n);
}
```

## 2.手写generator核心原理

我们从一个简单的例子开始，一步步探究Generator的实现原理：

```typescript
function* foo() {
  yield 'result1'
  yield 'result2'
  yield 'result3'
}
  
const gen = foo()
console.log(gen.next()) //{value: "result1", done: false}
console.log(gen.next()) //{value: "result2", done: false}
console.log(gen.next()) //{value: "result3", done: false}
console.log(gen.next()) //{value: undefined, done: true}
```

看到这种整齐的结构，我想起了switch case，也是这么地整齐，所以这两种之间应该存在一种关系。

我们尝试写一个用switch/case来实现下：

```typescript
function gen$(nextStep) {
    while (1) {
        switch (nextStep) {
            case 0:
                return 'result1';
            case 2:
                return 'result2';
            case 4:
                return 'result3';
            case 6:
                return undefined;
        }
    }
}
```

如代码所示，我们每次调用gen$然后传对应的参数，就能返回对应的值（也就是原本函数yield后面的值）

但是nextStep应该是一个自动增加的函数，应该不是我们传进去的。所以这里应该用一个闭包来实现

```typescript
function gen$() {
    var nextStep = 0
    return function () {
        while (1) {
            switch (nextStep) {
                case 0:
                    nextStep = 2;
                    return 'result1';

                case 2:
                    nextStep = 4;
                    return 'result2';

                case 4:
                    nextStep = 6;
                    return 'result3';

                case 6:
                    return undefined
            }
        }
    }
}
```

现在我们可以通过

```
var a = gen$()
```

获得内函数。
这样每次执行

```
a()
```

nextStep就会改成下一次执行a()应该对应的值，并且返回相应的result了。

但是generator的底层原理不是用闭包的。而是用一个全局变量，因为这样为了后面的实现方便很多，为了遵循原理，我们改成用全局变量来实现。

先定义一个全局变量

```typescript
context = {
  prev:0,
  next:0
}
function gen$(context) {
    while (1) {
        switch (context.prev = context.next) {
            case 0:
                context.next = 2;
                return 'result1';

            case 2:
                context.next = 4;
                return 'result2';

            case 4:
                context.next = 6;
                return 'result3';

           case 6:
                    return undefined
        }
    }
}
```

第一次执行`gen$(context)`,swtich判断的时候，是用**prev来判断这一次应该执行那个case**，执行case时再改变next的值，**next表示下次应该执行哪个case**。第二次执行`gen$(context)`的时候，将next的值赋给prev。

但是直接返回这么一个值是不对的。我们看前面的例子是返回一个对象。那该怎么实现呢？

再把例子搬下来：

```typescript
function* foo() {
  yield 'result1'
  yield 'result2'
  yield 'result3'
}
  
const gen = foo()
console.log(gen.next()) //{value: "result1", done: false}
console.log(gen.next()) //{value: "result2", done: false}
console.log(gen.next()) //{value: "result3", done: false}
console.log(gen.next()) //{value: undefined, done: true}
```

我们发现 gen 有next这个方法。所以可以判断出 执行foo返回的应该是一个对象，这个对象有next这个方法。所以我们初步实现foo的转化后的函数。

```typescript
let foo = function () {
    return {
        next: function () {

        }
    }
}
```

而每次执行next，就会返回拥有value和done的对象，

所以，可以完善返回值

```typescript
let foo = function () {
  return {
      next: function () {
          return {
              value,
              done
          }
      }
  }
}
```

但是我们这里还没定义这value和done啊，该怎么定义呢？

我们先看value的实现。我们在上面实现gen$的时候，就发现它返回的是value了。所以可以在这里获取`$gen`的返回值作为value。

```typescript
 let foo = function () {
            return {
                next: function () {
                    value = gen$(context)
                    return {
                        value,
                        done
                    }
                }
            }
        }
```

那done怎么定义呢？

其实done作为一个全局状态表示generator是否执行结束，因此，我们可以在

context里定义，默认值为false。

```typescript
var context = {
  next:0,
  prev: 0,
  done: false,

}
```

所以，每次返回，直接返回context.done就可以了

```typescript
let foo = function () {
    return {
        next: function () {
            value = gen$(context);
            done = context.done
            return {
                value,
                done
            }
        }
    }
}
```

那done是怎么改变为true的。我们知道，generator执行到后面，就会返回done:true。我们可以看例子的第四个执行结果

```typescript
function* foo() {
  yield 'result1'
  yield 'result2'
  yield 'result3'
}
  
const gen = foo()
console.log(gen.next()) //{value: "result1", done: false}
console.log(gen.next()) //{value: "result2", done: false}
console.log(gen.next()) //{value: "result3", done: false}
console.log(gen.next()) //{value: undefined, done: true}
```

因此，我们需要在最后一次执行gen$的时候改变context.done的值。

思路，给context添加一个stop方法。用来改变自身的done为true。在执行$gen的时时候让context执行stop就好

```typescript
var context = {
  next:0,
  prev: 0,
  done: false,
  新增代码
  stop: function stop () {
    this.done = true
  }
}
function gen$(context) {
    while (1) {
        switch (context.prev = context.next) {
            case 0:
                context.next = 2;
                return 'result1';

            case 2:
                context.next = 4;
                return 'result2';

            case 4:
                context.next = 6;
                return 'result3';

            case 6:
                新增代码
                context.stop();
                return undefined
        }
    }
}
let foo = function () {
    return {
        next: function () {
            value = gen$(context);
            done = context.done
            return {
                value,
                done
            }
        }
    }
}这样执行到case为6的时候就会改变done的值了。
```

实际上这就是generator的大致原理

并不难理解，我们分析一下流程：

**我们定义的function\*生成器函数被转化为以上代码**

**转化后的代码分为三大块：**

`gen$(_context)`由yield分割生成器函数代码而来

**context**对象用于储存函数执行上下文

**迭代器法定义next()**，用于执行gen$(_context)来跳到下一步

从中我们可以看出，**「Generator实现的核心在于上下文的保存，函数并没有真的被挂起，每一次yield，其实都执行了一遍传入的生成器函数，只是在这个过程中间用了一个context对象储存上下文，使得每次执行生成器函数的时候，都可以从上一个执行结果开始执行，看起来就像函数被挂起了一样」**

## 3.参照源码实现Context类

不过，我们这里的context是个全局对象啊？我们都知道如果是下面这种情况：

```typescript
function* g() {
    var o = 1;
    yield o++;
    yield o++;
    yield o++;

}
var gen = g();

console.log(gen.next()); // 1

var xxx = g();

console.log(gen.next()); // 2
console.log(xxx.next()); // 1
console.log(gen.next()); // 3
```

我们发现 每个迭代器之间互不干扰，作用域独立。

也就是说每个迭代器的context是独立的。但是与我们目前实现的一个全局context不一致，这个我是百思不得其解，所以看下源码。

利用babel将下面代码转化一下

```typescript
function* foo() {
  yield 'result1'
  yield 'result2'
  yield 'result3'
}
```

我们可以在babel官网上在线转化这段代码，看看ES5环境下是如何实现Generator的：

```typescript
"use strict";

var _marked =
/*#__PURE__*/
regeneratorRuntime.mark(foo);

function foo() {
  return regeneratorRuntime.wrap(function foo$(_context) {
    while (1) {
      switch (_context.prev = _context.next) {
        case 0:
          _context.next = 2;
          return 'result1';

        case 2:
          _context.next = 4;
          return 'result2';

        case 4:
          _context.next = 6;
          return 'result3';

        case 6:
        case "end":
          return _context.stop();
      }
    }
  }, _marked);
}
```

看源码，你可能觉得跟我们实现的有点不一样，实际上结构是基本一样的，基本都是分成那三部分

发现源码是将我们的gen$(context)方法传入了wrap中。

我们看下wrap方法

```typescript
function wrap(innerFn, outerFn, self) {
  var generator = Object.create(outerFn.prototype);
  var context = new Context([]);
  generator._invoke = makeInvokeMethod(innerFn, self, context);

  return generator;
}
```

发现它是每生foo()执行一次 ，就会执行一次wrap方法，而在wrap方法里就会new 一个Context对象。这就说明了每个迭代器的context是独立的。

Soga~原来如此~~~~

也就是说如果我们要实现独立context还是 把context改成一个类。

在执行`var gen = g();`的时候再生成context实例即可：

```typescript
class Context {
    constructor() {
        this.next = 0
        this.prev = 0
        this.done = false
    }
    top() {
        this.done = true
    }
}
let foo = function () {
    var context = new Context() 新增代码
    return {
        next: function () {
            value = gen$(context);
            done = context.done
            return {
                value,
                done
            }
        }
    }
}
```

## 4.参照源码实现参数值的保存

好了，这个独立context问题解决。但是发现哈有一个问题：

```typescript
function* foo() {
    var a = yield 'result1'
    console.log(a);
    yield 'result2'
    yield 'result3'
}

const gen = foo()
console.log(gen.next().value)
console.log(gen.next(222).value)
console.log(gen.next().value)
```

我们发现这里用`var a` 来接收传入的参数。

当我们第一次执行gen.next()，foo内部会执行到yield这里。还没给a赋值

当我们第二次执行gen.next()，foo内部会再第一个yield这里执行。把传入的参数222赋值给a

。
那原理是怎么实现的呢？我依旧百思不得其解，不得不再看下源码。

将下面代码babel一下

```typescript
function* foo() {
            var a = yield 'result1'
            console.log(a);
            yield 'result2'
            yield 'result3'
        }
"use strict";

var _marked = /*#__PURE__*/regeneratorRuntime.mark(foo);

function foo() {
  var a; 在这里定义
  return regeneratorRuntime.wrap(function foo$(_context) {
    while (1) {
      switch (_context.prev = _context.next) {
        case 0:
          _context.next = 2;
          return 'result1';

        case 2:
          a = _context.sent; 在这里赋值
          console.log(a);
          _context.next = 6;
          return 'result2';

        case 6:
          _context.next = 8;
          return 'result3';

        case 8:
        case "end":
          return _context.stop();
      }
    }
  }, _marked);
}
```



可见。是将我们在generator定义的变量提到foo函数顶部了。作为一个闭包的变量。

因此，居于这个思路，我们可以完善一下我们的代码。

如果我们在nenerator定义了xxx这个变量，那么就会被提升到函数顶部

```typescript
function gen$(context) {
    var xxx；新增代码
    while (1) {
        switch (context.prev = context.next) {
            case 0:
                context.next = 2;
                return 'result1';

            case 2:
            
                context.next = 4;
                return 'result2';

            case 4:
                context.next = 6;
                return 'result3';

            case 6:
              
                context.stop();
                return undefined
        }
    }
}
```

如果我们将出传入的参数赋值给这个变量

那么

参数就会作为Context的参数。将传入的参数保存到context中。

```typescript
let foo = function () {
    var context = new Context(222) //修改代码
    return {
        next: function () {
            value = gen$(context);
            done = context.done
            return {
                value,
                done
            }
        }
    }
}
```

然后在`gen$()`执行的时候再赋值给变量

```typescript
function gen$(context) {
    var xxx；
    while (1) {
        switch (context.prev = context.next) {
            case 0:
                context.next = 2;
                return 'result1';

            case 2:
                xxx = context._send 新增代码
                context.next = 4;
                return 'result2';

            case 4:
                context.next = 6;
                return 'result3';

            case 6:
                
                context.stop();
                return undefined
        }
    }
}
```