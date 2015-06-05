# Rust 编程语言 (2015-06)

## 1.介绍
Rust特有的核心概念：**ownership**

```rust
fn main() {
    let mut x = vec!["Hello", "world"];
    {
        let y = &x[0];
    }
    x.push("foo");
}
```

## 2.起步

### 2.1.安装Rust
**安装**使用官方提供的安装脚本：`rustup.sh`

```sh
$ curl -sf -L https://static.rust-lang.org/rustup.sh | sh
```

**卸载**脚本：`uninstall.sh`

```sh
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

> 安装时遇到**Rustc: error while loading shared libraries:librustc_driver--4e7c5e5c.so** 
> 是因为非Debian系统默认不信任 /lib 和 /usr/lib , 
> rust安装到 /usr/local, **ldconfig**无法识别新安装的库，
> 只需要执行 `sudo ldconfig /usr/local/lib` 即可。 
> 详见 [rust-issues 24677](https://github.com/rust-lang/rust/issues/24677)

### 2.2.Hello, world
Rust不限制代码存放路径。

```sh
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
$ touch main.rs
```

在 `main.rs` 中：

```rust
fn main() {
    println!("Hello, world!");
}
```

```sh
$ rustc main.rs
$ ./main
```

说明：

1. 命名 `hello_world.rs` 优于 `helloworld.rs`.
2. 花括号与函数声明同行，并空一个空格。
3. 缩进使用四个空格而非tab.
4. `println!()` 表示调用 `macro`, 它是Ｒust的元编程，普通函数形如 `println()`.
5. Rust是表达式导向型语言，表达式以 `;` 结束。

## 3.学习Rust

## 4.Rust有效编程

## 5.语法

### 5.1.变量绑定 (Variable Bindings)

```rust
fn main() {
    let x = 5;
}
```

`let` 表达式左侧是一个 `pattern` 而非单纯的变量名。
例如下面式子执行后 x 为 1 ，y 为 2 ：

```rust
let (x, y) = (1, 2);
```

Rust是静态语言，在编译期确定类型。但是它可以进行类型推断，因此可以省略类型。
明确的类型声明使用冒号：

```rust
let x: i32 = 5;
```

> `i` 表示有符号整数， `u` 表示无符号整数:

> i8, i16, i32, i64, 
> u8, u16, u32, u64

默认情况下绑定是**不可变 (immutable)**的(其中一个重要原因是安全)，
可变绑定可使用 `mut` ：

```rust
let mut x = 5; // mut x: i32
x = 10;
```

变量绑定使用前必须初始化，否则会报错。未初始化但未使用的变量编译时会报警告。


### 5.2.函数 (Functions)

Rust程序至少包含一个 `main `函数：

```rust
fn main() {
}
```

使用 `fn` 进行函数声明，一个带参数的函数：

```rust
fn print_number(x: i32) {
    println!("x is: {}", x);
}
```

参数必须指明类型, 多个参数用逗号分隔。

返回值的类型使用 `->` 符号指定：

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```

Rust只允许单一返回值，最后一行代表返回值。**注意： 没有分号！**

#### 表达式和语句

Rust是基于表达式的语言，它只有两种语句，其它均为表达式。

__1) 声明语句 (declaration statement)__

Rust中 `let` 变量绑定**不能**接表达式, 只能是语句。

赋值语句的值是一个空元组 `()`，而非值本身(尽管这可能没什么用)：

```rust
let mut y = 5;
let x = (y = 6);  // x 值为 `()`, 而非 `6`
```

__2) 表达式语句 (expression statement)__

目的是将表达式转换为语句; Rust的语法认为语句后面应该还是语句，不同表达式间用分号分隔。

#### 提前返回

提前返回结果使用 `return` 关键字：

```rust
fn foo(x: i32) -> i32 {
    return x;
    // 不会执行
    x + 1
}
```

`return` 用在最后一行也是可以的，但这被认为是糟糕的编码风格。

```rust
// poor style:
fn foo(x: i32) -> i32 {
    return x + 1;
}
```

#### Diverging functions

一种特殊语法 `diverging functions` :

```rust
fn diverges() -> ! {
    panic!("This function never returns!");
}
```

> 宏 `panic!` 会导致当前执行线程crash掉。
> 由于函数永远没有返回值，故类型为 `!` (读作 diverges)

该函数可以被用作任意类型：

```rust
let x: i32 = diverges();
let x: String = diverges();
```

### 5.3.原生类型 (Primitive Types)

Rust提供了一些语言内建的原生类型，另一些标准库中提供的实用类型则是构建在原生类型之上。

#### 布尔 (Booleans)

`bool` 类型有两个值： `true` 和 `false`

```rust
let x = true;
let y: bool = false;
```

#### 字符 (char)

`char` 类型通过单引号 `'` 创建：

```rust
let x = 'x';
let two_hearts = '💕';
```

Rust的char类型并非一个字节，而是四个字节。

#### 数字类型

* i8 / i16 / i32 / i64
* u8 / u16 / u32 / u64
* isize / usize
* f32 / f64

默认类型：

```rust
let x = 42;  // i32
let y = 1.0; // f64
```

分类：

* 有符号 / 无符号： i32, u8
* 固定大小 / 可变大小： u32, isize
* 浮点: f32, f64

> 可变大小的数字类型，其大小基于底层机器的指针大小。

#### 数组 (Arrays)

数组默认是不可变的：

```rust
let a = [1, 2, 3];
let mut m = [1, 2, 3];
```

数组类型为 `[T; N]` , 其中 `N` 为编译期常量, 代表数组的长度。

将数组全部初始化为 0 的简化写法：

```rust
let a = [0; 20];
```

可通过 `len()` 获取数组长度：

```rust
let a = [1, 2, 3];
println!("a has {} elements.", a.len());
```

可通过下标符号获取特定的数组元素(下标从 **0** 开始)：

```rust
let names = ["Graydon", "Brian", "Niko"]; // names: [&str; 3]
println!("The second name is: {}", names[1]);
```

#### 切片 (Slices)

切片是对数组的引用，通常不会直接创建。可变或不变类型均可:

```rust
let a = [0, 1, 2, 3, 4];
let middle = &a[1..4];   // 1, 2, 3
let complete = &a[..];
```

切片类型为 `&[T]` .

#### str

`str` 是Rust最原生的字符串类型。自身并没什么作用，通常作为引用使用： `&str`.

#### 元组 (Tuples)

元组是指固定大小的有序列表：

```rust
let x = (1, "hello");
```

加上类型后是这样：

```rust
let x: (i32, &str) = (1, "hello");
```

类型及参数数量相同时，元组可以赋值：

```rust
let mut x = (1, 2);
let y = (2, 3);

x = y;
```

destructuring let:

```rust
let (x, y, z) = (1, 2, 3);
println!("x is {}", x);
```

区分：

```rust
(0,);   // 单元素元组
(0);    // 括号中的 0
```

元组索引：

```rust
let tuple = (1, 2, 3);
let x = tuple.0;
let y = tuple.1;
let z = tuple.2;
println!("x is {}"，x);
```

#### 函数

函数也是一种类型：

```rust
fn foo(x: i32) -> i32 { x }
let x: fn(i32) -> i32 = foo;
```

例子中，x 是一个函数指针，占用i32返回i32.


### 5.4.注释 (Comments)

Rust的注释分为两种： **行注释**和**文档注释**

```rust
// this is a line comment.
let x = 5;
```

文档注释支持Markdown语法，可以通过 `rustdoc` 工具生成HTML并测试示例代码：

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, add_one(5));
/// ```
fn add_one(x: i32) -> i32 {
    x + 1
}
```


### 5.5.if

`if` 的语法更像动态语言，而非系统语言。传统用法：

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
} else if x == 6 {
    println!("x is six!");
} else {
    println!("x is not five or six :(");
}
```

稍复杂些的用法：

```rust
let x = 5;
let y = if x == 5 { 10 } else { 15 };  // y 为 10
```

`if` 是一个表达式而非语句，表达式的值是某分支的最后一个表达式的值。


### 5.6.for

Rust的 `for` 语法如下，其中expression为 `iterator`：

```
for var in expression {
    code
}
```

例如：

```rust
for x in 0..10 {
    println!("{}", x);
}
```


### 5.7.while

```rust
let mut x = 5;
let mut done: bool = false;

while !done {
    x += x -3;
    println!("{}", x);

    if x % 5 == 0 {
        done = true;
    }
}
```

`while` 用于不确定循环次数的情况。无限循环可以使用：

```rust
while true {
}
```

然而此情况Rust有专用关键字 `loop` :

```rust
loop {
}
```

Rust在流程分析时会区别对待两种写法，loop循环可以提供更多信息因此可以更安全的处理。
无限循环时，应用总是使用 `loop` 形式。

结束循环的 `break` 和 `continue` (for中也可以用)：

```rust
let mut x = 5;

loop {
    x += x -3;
    if x % 5 == 0 { break; }
}
```

```rust
for x in 0..10 {
    if x % 2 == 0 { continue; }
}
```


### 5.8.所有权 (Ownership)

变量绑定时具有 `所有权` 属性，超出作用域时，其资源会释放。

任一给定资源在Rust中，只有**明确的一个**绑定，以下代码会报错：

```rust
// error !!!!
let v = vec![1, 2, 3];
let v2 = v;
println!("v[0] is {}", v[0]);
```

调用具有所有权的函数时也有可能产生同样的错误：

```rust
// error !!!!
fn take(v: Vec<i32>) {
    // ...
}

let v = vec![1, 2, 3];
take(v);

println!("v[0] is {}", v[0]);
```

被移动的绑定不能使用**细节说明**：

```rust
let v = vec![1, 2, 3];
let v2 = v;
```

第一行代码的内存分配：向量对象存放在栈中，并包含一个指向实际内容([1, 2, 3])的指针，
内容存放在堆中。当将 `v` 移到 `v2` 时，会为 `v2` 创建其指针的复本，也就是说，
会有两个指向堆中的指针。由于引入的数据竞争违反了Rust的安全保证机制，因此，
Rust之后将会禁止使用 `v` 。另外，优化器有可能会根据情况移除栈内的实际拷贝。

**Copy类型**有不同的表现(和 `traits` 机制有关)：

```rust
let v = 1;
let v2 = v;
println!("v is {}", v);
```

此时，`v` 是i32类型的，它实现了 `Copy` 。赋值时不产生指针复制，而是直接复制数据值。
因此之后 `v` 仍然可以使用。

返还所有权：

```rust
fn foo(v: Vec<i32>) -> Vec<i32> {
    // ...
    v
}
```

一个更有趣的例子(有些难懂，可以通过 `borrowing` 机制解决)：

```rust
fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    (v1, v2, 42)
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let (v1, v2, answer) = foo(v1, v2);
```


### 5.9.引用和借用 (References and Borrowing)

借用示例：

```rust
fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    42
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];
let answer = foo(&v1, &v2);
```

`&T` 类型代表**引用**，引用并不占据资源本身而只是借用其所有权，超出作用域时，
绑定也不会解除。也就是说，调用完 `foo()` 后，原始绑定仍然可以使用。

**引用是不可变的**,以下代码会报错：

```rust
// error !!!!
fn foo(v: &Vec<i32>) {
    v.push(5);
}

let v = vec![];
foo(&v);
```

还有一种**可变**引用: `&mut T` ：

```rust
let mut x = 5;
{
    let y = &mut x;
    *y += 1;
}
println!("{}", x);  // 6
```

不可以借用不可变值，因此 `x` 必须为 `mut` .

#### 借用规则

* 首先，任何借用都要比所有者维持更小的作用域。
* 其次，只能同时拥有以下两种中的一种借用：
  * 0到N个对资源的引用(&T)
  * 明确只有1个可变引用(&mut T)

> 当有两个或更多指针同时指向同一块内存区域时，
> 如果其中至少有一个指针正在进行写数据，操作就不会同步。
> 此时则会产生**数据竞争** (data race)。

可以拥有任意数量的引用，因为它们不会写数据。而同时只能拥有一个可变引用。
Rust以此来阻止数据竞争。

#### 借用常犯的错误

**1) 非法遍历：**

```rust
// error !!!!
let mut v = vec![1, 2, 3];

// 循环借用了v
for i in &v {
    println!("{}", i);
    v.push(34);
}
```

**2) 释放后使用：**

```rust
// error !!!!
let y: &i32;
{
    let x = 5;
    y = &x;
}
println!("{}", y);
```

`y` 只在 `x` 存在的作用域内有效。同样的错误还有引用声明在变量前：

```rust
// error !!!!
let y: &i32;
let x = 5;
y = &x;

println!("{}", y);
```







### 5.10.使用期 (Lifetimes)

### 5.11.可变性 (Mutability)

### 5.12.结构体 (Structs)










