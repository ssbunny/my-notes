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

> 安装时遇到**Rustc: error while loading shared libraries:librustc_driver--4e7c5e5c.so** 是因为非Debian系统默认不信任 /lib 和 /usr/lib , rust安装到 /usr/local, **ldconfig**无法识别新安装的库，只需要执行 `sudo ldconfig /usr/local/lib` 即可。 详见 [https://github.com/rust-lang/rust/issues/24677](https://github.com/rust-lang/rust/issues/24677)

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








### 5.3.原生类型 (Primitive Types)

### 5.4.注释 (Comments)

### 5.5.if

### 5.6.for

### 5.7.while

### 5.8.所有权 (Ownership)

### 5.9.引用和借用 (References and Borrowing)

### 5.10.使用期 (Lifetimes)












