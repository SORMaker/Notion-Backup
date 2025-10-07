# Reference Cycles and Self Reference

# Weak与循环引用

## 什么是循环引用

 先给出下面一段代码：

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a的初始化rc计数 = {}", Rc::strong_count(&a));
    println!("a指向的节点 = {:?}", a.tail());

    // 创建`b`到`a`的引用
    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("在b创建后，a的rc计数 = {}", Rc::strong_count(&a));
    println!("b的初始化rc计数 = {}", Rc::strong_count(&b));
    println!("b指向的节点 = {:?}", b.tail());

    // 利用RefCell的可变性，创建了`a`到`b`的引用
    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("在更改a后，b的rc计数 = {}", Rc::strong_count(&b));
    println!("在更改a后，a的rc计数 = {}", Rc::strong_count(&a));

    // 下面一行println!将导致循环引用
    // 我们可怜的8MB大小的main线程栈空间将被它冲垮，最终造成栈溢出
    // println!("a next item = {:?}", a.tail());
}
```

在这段代码中，我们可以看出：

1. 我们创造了`a`后，紧接着使用了`a`创造了`b`，因此`b`引用了`a`。
2. 我们又使用`Rc`克隆了`b`，然后通过 `RefCell` 的可变性，让 `a` 引用了 `b`。
3. 最终我们创造了循环引用：`a`-> `b` -> `a` -> `b` ····

观察一下输出：

```rust
a的初始化rc计数 = 1
a指向的节点 = Some(RefCell { value: Nil })
在b创建后，a的rc计数 = 2
b的初始化rc计数 = 1
b指向的节点 = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
在更改a后，b的rc计数 = 2
在更改a后，a的rc计数 = 2
```

在 `main` 函数结束前，`a` 和 `b` 的引用计数均是 `2`，随后 `b` 触发 `Drop`，此时引用计数会变为 `1`，并不会归 `0`，因此 `b` 所指向内存不会被释放，同理可得 `a` 指向的内存也不会被释放，最终发生了内存泄漏。

一张图可以很好的展示这种循环引用关系：

![v2-2dbfc981f05019bf70bf81c93f956c35_1440w.png](Reference%20Cycles%20and%20Self%20Reference%201d4b9c07c31280dbbb1fc54de4952be3/v2-2dbfc981f05019bf70bf81c93f956c35_1440w.png)

甚至，我们注释一下最后一行的代码，通过调用 `a.tail` ，Rust 试图打印出 `a -> b -> a ···` 的所有内容，但是在不懈的努力后，`main` 线程终于不堪重负，发生了[栈溢出](https://course.rs/pitfalls/stack-overflow.html)。

```rust
RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell {
...无穷无尽
thread 'main' has overflowed its stack
fatal runtime error: stack overflow

```

## Weak

`Weak` 非常类似于 `Rc`，但是与 `Rc` 持有所有权不同，`Weak` 不持有所有权，它仅仅保存一份指向数据的弱引用：如果你想要访问数据，需要通过 `Weak` 指针的 `upgrade` 方法实现，该方法返回一个类型为 `Option<Rc<T>>` 的值。

看到这个返回，相信大家就懂了：何为弱引用？就是**不保证引用关系依然存在**，如果不存在，就返回一个 `None`！

因为 `Weak` 引用不计入所有权，因此它**无法阻止所引用的内存值被释放掉**，而且 `Weak` 本身不对值的存在性做任何担保，引用的值还存在就返回 `Some`，不存在就返回 `None`。

### Weak vs Rc

我们来将 `Weak` 与 `Rc` 进行以下简单对比：

| `Weak` | `Rc` |
| --- | --- |
| 不计数 | 引用计数 |
| 不拥有所有权 | 拥有值的所有权 |
| 不阻止值被释放(drop) | 所有权计数归零，才能 drop |
| 引用的值存在返回 `Some`，不存在返回 `None` | 引用的值必定存在 |
| 通过 `upgrade` 取到 `Option<Rc<T>>`，然后再取值 | 通过 `Deref` 自动解引用，取值无需任何操作 |

通过这个对比，可以非常清晰的看出 `Weak` 为何这么弱，而这种弱恰恰非常适合我们实现以下的场景：

- 持有一个 `Rc` 对象的临时引用，并且不在乎引用的值是否依然存在
- 阻止 `Rc` 导致的循环引用，因为 `Rc` 的所有权机制，会导致多个 `Rc` 都无法计数归零

使用方式简单总结下：**对于父子引用关系，可以让父节点通过 `Rc` 来引用子节点，然后让子节点通过 `Weak` 来引用父节点**。

### Weak总结

因为 `Weak` 本身并不是很好理解，因此我们再来帮大家梳理总结下，然后再通过一个例子，来彻底掌握。

`Weak` 通过 `use std::rc::Weak` 来引入，它具有以下特点:

- 可访问，但没有所有权，不增加引用计数，因此不会影响被引用值的释放回收
- 可由 `Rc<T>` 调用 `downgrade` 方法转换成 `Weak<T>`
- `Weak<T>` 可使用 `upgrade` 方法转换成 `Option<Rc<T>>`，如果资源已经被释放，则 `Option` 的值是 `None`
- 常用于解决循环引用的问题

一个简单的例子：

```rust
use std::rc::Rc;
fn main() {
// 创建Rc，持有一个值5let five = Rc::new(5);

// 通过Rc，创建一个Weak指针let weak_five = Rc::downgrade(&five);

// Weak引用的资源依然存在，取到值5let strong_five: Option<Rc<_>> = weak_five.upgrade();
    assert_eq!(*strong_five.unwrap(), 5);

// 手动释放资源`five`drop(five);

// Weak引用的资源已不存在，因此返回Nonelet strong_five: Option<Rc<_>> = weak_five.upgrade();
    assert_eq!(strong_five, None);
}
```

需要承认的是，使用 `Weak` 让 Rust 本来就堪忧的代码可读性又下降了不少，但是。。。真香，因为可以解决循环引用了。

## unsafe解决循环引用

除了使用 Rust 标准库提供的这些类型，你还可以使用 `unsafe` 里的裸指针来解决这些棘手的问题。

虽然 `unsafe` 不安全，但是在各种库的代码中依然很常见用它来实现自引用结构，主要优点如下:

- 性能高，毕竟直接用裸指针操作
- 代码更简单更符合直觉: 对比下 `Option<Rc<RefCell<Node>>>`

# 自引用

## 结构体自引用

```rust
struct SelfRef<'a> {
    value: String,

    // 该引用指向上面的value
    pointer_to_value: &'a str,
}

fn main(){
    let s = "aaa".to_string();
    let v = SelfRef {
        value: s,
        pointer_to_value: &s
    };
}

 let v = SelfRef {
12 |         value: s,
   |                - value moved here
13 |         pointer_to_value: &s
   |                           ^^ value borrowed here after move

```

因为我们试图同时使用值和值的引用，最终所有权转移和借用一起发生了。

## 解决方案 (未完成)

### Option

最简单的方式就是使用 `Option` 分两步来实现：

```rust

#[derive(Debug)]
struct WhatAboutThis<'a> {
    name: String,
    nickname: Option<&'a str>,
}

fn main() {
    let mut tricky = WhatAboutThis {
        name: "Annabelle".to_string(),
        nickname: None,
    };
    tricky.nickname = Some(&tricky.name[..4]);

    println!("{:?}", tricky);
}
```

在某种程度上来说，`Option` 这个方法可以工作，但是这个方法的限制较多，例如从一个函数创建并返回它是不可能的。

```rust
fn creator<'a>() -> WhatAboutThis<'a> {
    let mut tricky = WhatAboutThis {
        name: "Annabelle".to_string(),
        nickname: None,
    };
    tricky.nickname = Some(&tricky.name[..4]);

    tricky
}

error[E0515]: cannot return value referencing local data `tricky.name`
  --> src/main.rs:24:5
   |
22 |     tricky.nickname = Some(&tricky.name[..4]);
   |                             ----------- `tricky.name` is borrowed here
23 |
24 |     tricky
   |     ^^^^^^ returns a value referencing data owned by the current function
```