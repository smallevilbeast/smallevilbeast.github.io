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

### 生命周期 

Rust 在函数或方法的签名中有一套生命周期参数的速记（省略）规则，这可以使生命周期的使用更为便捷。这些规则被编译器应用，所以在编写代码时，你只需要在需要更明确的生命周期时提供生命周期参数即可。

以下是这些速记规则：

1. **每个引用的参数都有自己的生命周期参数**。例如，一个函数的签名 `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)` 中，`x` 和 `y` 各有自己的生命周期 `'a` 和 `'b`。

2. **如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数**。例如，在 `fn foo<'a>(x: &'a i32) -> &'a i32`，`x` 和返回值都拥有相同的生命周期 `'a`。

3. **如果方法有多个输入生命周期参数并且其中一个参数是 `&self` 或 `&mut self`，那么所有输出生命周期参数被赋予 `self` 的生命周期**。这使得方法可以很容易地输出其它引用到与接收者相关的数据。

速记规则适用于大多数常见的情况。当生命周期需要更明确或者规则不适用时，必须使用显式的生命周期注解。

### 生命周期约束

生命周期约束可以根据需求以多种方式编写。举一些例子：

1. `T: 'a`: 如果 `T` 是某种引用，那么 `T` 的生命周期必须至少和 `'a` 一样长。这也可以用于泛型结构体，表示所有引用类型的数据成员必须至少存活 `'a` 那么久。

2. `T: Trait + 'a`: 对于某种实现了 `Trait` 的类型 `T`，如果 `T` 是引用类型，那么 `T` 的生命周期必须至少和 `'a` 一样长。

3. `'b: 'a`: 生命周期 `'b` 至少和生命周期 `'a` 一样长。

生命周期标注帮助Rust编译器理解引用的有效范围和相互关系，以此来保证内存安全并防止悬垂引用等问题。不过大部分时候Rust可以自动推断出正确的生命周期，你不必显式指定。显式生命周期主要在编译器无法自己推断时使用，例如在某些涉及到泛型的复杂函数或方法中。


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

