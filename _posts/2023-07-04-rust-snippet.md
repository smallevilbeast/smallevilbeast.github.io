---
layout: post
title: rust 代码片段
categories: [rust]
---

### 使用元组结构体和 Deref 定义新类型

实现 Deref 特征后， 可以自动做一层类似类型转换的操作， 可以将 NewString 变成 String 来使用。 这样就会像直接使用 String 那样去使用 NewString， 而无需为每一个操作都添加上 self.0。

同时， 如果不想 NewString 暴露底层数组的所有方法， 我们还可以为 NewString 去重载这些方法， 实现隐藏的目的

```rust
use std::ops::Deref;


struct NewString(String);

impl Deref for NewString {
    type Target = String;
    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

impl NewString {
    fn reversed(&self) -> String {
        self.0.chars().rev().collect()
    }
}


fn main() {
    let hello  = NewString(String::from("hello"));
    println!("This call String method size: {}, reversed: {}", hello.len(),  hello.reversed());

}
```

