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
