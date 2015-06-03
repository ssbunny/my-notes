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

> **i8**, **i16**, **i32**, **i64**, 
> **u8**, **u16**, **u32**, **u64**

默认情况下绑定是**不可变 (immutable)**的，可变绑定可使用 `mut` ：

```rust
let mut x = 5; // mut x: i32
x = 10;
```




### 5.2.函数 (Functions)

### 5.3.原生类型 (Primitive Types)

### 5.4.注释 (Comments)

### 5.5.if

### 5.6.for

### 5.7.while

### 5.8.所有权 (Ownership)

### 5.9.引用和借用 (References and Borrowing)

### 5.10.使用期 (Lifetimes)












