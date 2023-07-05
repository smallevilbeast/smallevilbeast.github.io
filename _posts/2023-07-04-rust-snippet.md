---
layout: post
title: rust 代码片段
categories: [rust]
---

### 书籍

[【Rust 秘典（死灵书）】](https://nomicon.purewhite.io/intro.html)  可参考如何实现一个 Vec
[【Rust 圣经】](https://course.rs/about-book.html)


### 借用规则

Rust 的借用规则是这样的：

1. 你可以有许多不可变引用（只读）。
2. 或者，你可以有一个可变引用（读/写）。
3. 但是你不能同时拥有可变引用和不可变引用。

Rust 的借用规则仍然适用，因为这些规则旨在防止数据竞争和其他并发问题，即使在单线程上下文中也是如此。

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

