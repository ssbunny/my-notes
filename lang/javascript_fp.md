# Functional JavaScript

Learn Underscore.js

## 1.函数式编程

### 1.1.JavaScript

**灵活性**

1) 函数可以返回另一个函数:

```js
function splat(fun) {
    return function(array) {
        return fun.apply(null, array);
    };}var addArrayElements = splat(function(x, y) { return x + y });addArrayElements([1, 2]); //=> 3
```

2) 函数可以在任意时刻调用，并传入任意数量、任意类型的参数:

```js
function unsplat(fun) {
    return function() {
        return fun.call(null, _.toArray(arguments));
    };}var joinElements = unsplat(function(array) { return array.join(' ') });joinElements(1, 2); //=> "1 2"joinElements('-', '$', '/', '!', ':'); //=> "- $ / ! :"```

**局限性**

* 语法怪异
* 功能特性不稳定
* 大量类库竞争

### 1.2.函数式编程

**概念**

> **Functional programming** is the use of functions that transform values into units of abstraction, subsequently used to build software systems.

#### 1.2.1.本质

本质：将大程序拆分成小的。

1) **面向对象**以“对象”(名词)为分组依据，大对象通过小对象组合而来：

<img src="img/js_1.png" width=400/>

**函数式**以“函数”(动词)为分组依据，大函数通过小函数组合而来：

<img src="img/js_2.png" width=400/>

2) **面向对象**通过方法调用进行对象间的交互：

<img src="img/js_3.png" width=400/>

**函数式**通过数据传递进行函数间的交互：

<img src="img/js_4.png" width=400/>

#### 1.2.2.函数作为抽象单元

UNIX 社区名言：

> Make it run, make it right, make it fast.

使用函数式编程，可以实现：

> Make it run, then make it right, then make it fast.

以函数为抽象单元，拆分大函数。

#### 1.2.3.封装

传统**OOP**中，封装通过打包、隐藏数据并提供操作数据的行为方法实现。

<img src="img/js_5.png" width=200/>

JavaScript的对象系统不能隐藏数据，需通过**闭包**实现：

<img src="img/js_6.png" width=300/>

闭包也是一种函数。

#### 1.2.4.函数作为行为单元

例如，常规数组索引：

```js
var letters = ['a', 'b', 'c'];
letters[1];//=> 'b'
```

可以抽象为 `nth` 函数，并做合法性验证：

```js
function nth(a, index) {    if (!_.isNumber(index)) fail("Expected a number as the index");
    if (!isIndexed(a)) fail("Not supported on non-indexed type");
    if ((index < 0) || (index > a.length - 1))
        fail("Index value is out of bounds");
    return a[index];}
```

使用时：

```js
nth("abc", 0); //=> "a"nth({}, 2);// Error: Not supported on non-indexed type
```

进而抽象出 `second` 函数：

```js
function second(a) {    return nth(a, 1);
}

second(['a','b']); //=> "b"second("fogus"); //=> "o"
```

#### 1.2.5.数据抽象

JavaScript的对象**原型**模型是一种函数式数据模型。
比基于类的对象系统更便于使用数据。

函数用作为两种数据格式间的桥梁。例如解析如下格式的数据表：

```
 name,   age, hair Merble, 35,  red Bob,    64,  blonde
```
使用 `underscore.js` 的函数式编程实现：

```js
function lameCSV(str) {    return _.reduce(str.split("\n"), function(table, row) {        table.push(_.map(row.split(","), function(c) { return c.trim()}));        return table;
    }, []);};
```

#### 1.2.6.函数式JavaScript

两个实用的函数：

```js
function existy(x) { return x != null };
function truthy(x) { return (x !== false) && existy(x) };
```

基于此构建：

```js
function doWhen(cond, action) {
    if(truthy(cond))        return action();
    else        return undefined;
}

function executeIfHasField(target, name) {    return doWhen(existy(target[name]), function() {        var result = _.result(target, name);
        return result;    });
}

executeIfHasField([1,2,3], 'reverse'); //=> [3, 2, 1]executeIfHasField({foo: 42}, 'foo');   //=> 42executeIfHasField([1,2,3], 'notHere'); //=> undefined
```
又如：

```js
[null, undefined, 1, 2, false].map(existy);
//=> [false, false, true, true, true]
[null, undefined, 1, 2, false].map(truthy);
//=> [false, false, true, true, false]```

函数式编程的代码形式即形如上面的例子。

#### 1.2.7.速度

速度确实会稍有下降，但是基于V8等新引擎，速度的影响几乎可以忽略。
函数式编程在实践中，几乎没有速度性能问题，然而却能带来优秀的代码结构。

## 2.一等函数和应用式编程

### 2.1.函数作为一等公民

**first-class** 意味着函数只是一个值，它可以：

1) 被存储在变量中：

```js
var fortytwo = function() { return 42 };
```

2) 被存储在数组中：

```js
var fortytwos = [42, function() { return 42 }];
```

3) 被存储在对象域中：

```js
var fortytwos = {number: 42, fun: function() { return 42 }};
```

4) 按需创建：

```js
42 + (function() { return 42 })(); //=> 84
```

5) 传递给另一个函数：

```js
function weirdAdd(n, f) { return n + f() }
weirdAdd(42, function() { return 42 });
```

6) 作为返回值：

```js
return function() { return 42 };
```

后两个叫做**高位(higher-order)函数**，它可以：

* 将另一个函数作为参数；
* 将另一个函数作为返回值。

#### 2.1.1 JavaScript多范式

**命令式编程(Imperative programming)**

例如打印 *99 Bottles of Beer* 的歌词：

```js
var lyrics = [];for (var bottles = 99; bottles > 0; bottles--) {
    lyrics.push(bottles + " bottles of beer on the wall");
    lyrics.push(bottles + " bottles of beer");
    lyrics.push("Take one down, pass it around");    
    if (bottles > 1) {        lyrics.push((bottles - 1) + " bottles of beer on the wall.");    } else {        lyrics.push("No more bottles of beer on the wall!");    }}
```

通过函数式抽象出 `lyricSegment` 打印歌词：

```js
function lyricSegment(n) {
    return _.chain([])        .push(n + " bottles of beer on the wall")
        .push(n + " bottles of beer")
        .push("Take one down, pass it around")
        .tap(function(lyrics) {
            if (n > 1)                lyrics.push((n - 1) + " bottles of beer on the wall.");
            else                lyrics.push("No more bottles of beer on the wall!");         })        .value();
}
```

抽象出最终的函数：

```js
function song(start, end, lyricGen) {
    return _.reduce(_.range(start,end,-1),        function(acc,n) {            return acc.concat(lyricGen(n));        }, []);
}

song(99, 0, lyricSegment);```

**基于原型的面向对象编程(Prototype-based object-oriented programming)**



