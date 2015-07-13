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

![oop](img/js_1.png =100)

**函数式**以“函数”(动词)为分组依据，大函数通过小函数组合而来：

![fp](img/js_2.png)

2) **面向对象**通过方法调用进行对象间的交互：

![oop](img/js_3.png)

**函数式**通过数据传递进行函数间的交互

![oop](img/js_4.png)

#### 1.2.2.函数作为抽象单元





