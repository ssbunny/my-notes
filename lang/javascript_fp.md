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
* 安全性差
* 大量类库

### 1.2.函数式编程

**概念**

> Functional programming is the use of functions that transform values into units of ab‐ straction, subsequently used to build software systems.

#### 1.2.1.本质

本质：将大程序拆分成小的。

面向对象以“对象”(名词)为分组依据：

![oop](img/js_1.png)