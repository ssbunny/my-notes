# Rust 编程语言

## 1.介绍

Rust特有的核心概念：**ownership**

``````rust
fn main() {
    let mut x = vec!["Hello", "world"];
    {
        let y = &x[0];
    }
    x.push("foo");
}
``````
