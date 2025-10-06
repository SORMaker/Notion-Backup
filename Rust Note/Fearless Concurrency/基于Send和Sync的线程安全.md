# 基于Send和Sync的线程安全

# Rc和Arc源码对比

在介绍`Send`特征之前，再来看看`Arc`为何可以在多线程使用，玄机在于两者的源码实现上：

```rust
// Rc源码片段impl<T: ?Sized> !marker::Send for Rc<T> {}
impl<T: ?Sized> !marker::Sync for Rc<T> {}

// Arc源码片段unsafe impl<T: ?Sized + Sync + Send> Send for Arc<T> {}
unsafe impl<T: ?Sized + Sync + Send> Sync for Arc<T> {}
```

`!`代表移除特征的相应实现，上面代码中`Rc<T>`的`Send`和`Sync`特征被特地移除了实现，而`Arc<T>`则相反，实现了`Sync + Send`，再结合之前的编译器报错，大概可以明白了：`Send`和`Sync`是在线程间安全使用一个值的关键。

# Send和Sync

`Send`和`Sync`是 Rust 安全并发的重中之重，但是实际上它们只是标记特征（marker trait，该特征未定义任何行为，也就是说，这个标记仅仅是为了对付一下编译器，告诉他这个东西是线程安全的，至于是不是线程安全，需要你自己来保证），来看看它们的作用：

- 实现`Send`的类型可以在线程间安全的传递其所有权
- 实现`Sync`的类型可以在线程间安全的共享（通过引用）

这里还有一个潜在的依赖：一个类型要在线程间安全的共享的前提是，指向它的引用必须能在线程间传递。因为如果引用都不能被传递，我们就无法在多个线程间使用引用去访问同一个数据了。

由上可知，**若类型 T 的引用`&T`是`Send`，则`T`是`Sync`**。

- 对上面这句话一些好的解释：
    - 简而言之：`Send` 是 `Sync` 的前提，也就是，传递是共享的前提。而共享是通过引用去共享的，那么如果一个类型 T 的引用&T 是能传递的，说明类型 T 是可以能共享的。
    - 这里我用堆栈举个例子，如果一个变量存储在堆上，在单线程中，我们怎么样才能访问到这个堆上的数据？
        - 答案显而易见，是将这个堆上数据的地址（指针）存在栈上，我们利用栈上的这个地址访问堆上数据。
        
        现在变到多线程了！ 有很多线程都想访问这个数据，怎么办？
        
        - 我们只需要把这个堆上数据的地址（指针）拷贝存不同线程的栈上，这样这些线程就能访问到这个数据了
        
        这就是数据共享，共享的本质就是它的指针/引用能够“发送”到不同线程上，所以 `T: Sync` 就表示 `T` 这个变量的引用 `&T: Send` (有点不规范，但是这个意思）。
        
        再说下那两个例子，首先, `RwLock<T>` 和 `Mutex<T>` 都是实现了`Sync`的，只是对结构体中包裹的字段`T`类型有不一样的限制。 `RwLock` 允许多个线程同时获得`T`的不可变引用，那`T`必定要实现`Sync`,而`Mutex`就没有这个要求了，所以它的字段是否实现`Sync`就无关紧要了
        

没有例子的概念讲解都是耍流氓，来看看`RwLock`的实现：

```rust
unsafe impl<T: ?Sized + Send + Sync> Sync for RwLock<T> {}
```

**`RwLock<T>` 允许并发读的安全前提是 `T: Sync`**（确保共享引用的线程安全性），而跨线程共享 `RwLock<T>` 还要求 `T: Send`（确保独占写锁的转移安全性）。标准库通过 `unsafe impl` 强制这一约束，避免用户绕过类型系统引入数据竞争。即使仅使用写锁，`T: Sync` 仍是必要条件（因 `RwLock<T>` 的 `Sync` 实现依赖此约束）。

| **锁类型** | **并发模型** | **对 T 的要求** | **安全边界条件** |
| --- | --- | --- | --- |
| 读锁 | 多线程并发读 | `T: Sync` | 必须，否则数据竞争 |
| 写锁 | 单线程独占写 | `T: Send` | 必须，否则跨线程转移不安全 |
| 混合锁 | 多读一写 | `T: Send + Sync` | 标准库的 impl 约束 |

果不其然，上述代码中，`T`的特征约束中就有一个`Sync`特征，那问题又来了，`Mutex`是不是相反？再来看看：

```rust
unsafe impl<T: ?Sized + Send> Sync for Mutex<T> {}
```

不出所料，`Mutex<T>`中的`T`并没有`Sync`特征约束。

# 实现Send和Sync的类型

在 Rust 中，几乎所有类型都默认实现了`Send`和`Sync`，而且由于这两个特征都是可自动派生的特征(通过`derive`派生)，意味着一个复合类型（例如结构体），只要它内部的所有成员都实现了`Send`或者`Sync`，那么它就自动实现了`Send`或`Sync`。

正是因为以上规则，Rust 中绝大多数类型都实现了`Send`和`Sync`，除了以下几个(事实上不止这几个，只不过它们比较常见）

- 裸指针两者都没实现，因为它本身就没有任何安全保证
- `UnsafeCell`不是`Sync`，因此`Cell`和`RefCell`也不是
- `Rc`两者都没实现（因为内部的引用计数器不是线程安全的）

当然，如果是自定义的复合类型，那没实现那哥俩的就较为常见了：**只要复合类型中有一个成员不是`Send`或`Sync`，那么该复合类型也就不是`Send`或`Sync`**。

**手动实现 `Send` 和 `Sync` 是不安全的**，通常并不需要手动实现 Send 和 Sync trait，实现者需要使用`unsafe`小心维护并发安全保证。

# 为裸指针实现Send

上面我们提到裸指针既没实现`Send`，意味着下面代码会报错:

```rust
use std::thread;
fn main() {
    let p = 5 as *mut u8;
    let t = thread::spawn(move || {
        println!("{:?}",p);
    });

    t.join().unwrap();
}
```

报错跟之前无二： ``*mut u8` cannot be sent between threads safely`, 但是有一个问题，我们无法为其直接实现`Send`特征，好在可以用[`newtype`类型](https://course.rs/advance/into-types/custom-type.html#newtype) :`struct MyBox(*mut u8);`。

还记得之前的规则吗：复合类型中有一个成员没实现`Send`，该复合类型就不是`Send`，因此我们需要手动为它实现:

```rust
use std::thread;

#[derive(Debug)]
struct MyBox(*mut u8);
unsafe impl Send for MyBox {}
fn main() {
    let p = MyBox(5 as *mut u8);
    let t = thread::spawn(move || {
        println!("{:?}",p);
    });

    t.join().unwrap();
}
```

此时，我们的指针已经可以欢快的在多线程间撒欢，以上代码很简单，但有一点需要注意：`Send`和`Sync`是`unsafe`特征，实现时需要用`unsafe`代码块包裹。

# 为裸指针实现Sync

由于`Sync`是多线程间共享一个值，大家可能会想这么实现：

```rust
use std::thread;
fn main() {
    let v = 5;
    let t = thread::spawn(|| {
        println!("{:?}",&v);
    });

    t.join().unwrap();
}
```

关于这种用法，在多线程章节也提到过，线程如果直接去借用其它线程的变量，会报错:`closure may outlive the current function,`, 原因在于编译器无法确定主线程`main`和子线程`t`谁的生命周期更长，特别是当两个线程都是子线程时，没有任何人知道哪个子线程会先结束，包括编译器！

因此我们得配合`Arc`去使用:

```rust
use std::thread;
use std::sync::Arc;
use std::sync::Mutex;

#[derive(Debug)]
struct MyBox(*const u8);
unsafe impl Send for MyBox {}

fn main() {
    let b = &MyBox(5 as *const u8);
    let v = Arc::new(Mutex::new(b));
    let t = thread::spawn(move || {
        let _v1 =  v.lock().unwrap();
    });

    t.join().unwrap();
}
```

上面代码将智能指针`v`的所有权转移给新线程，同时`v`包含了一个引用类型`b`，当在新的线程中试图获取内部的引用时，会报错：

```
error[E0277]: `*const u8` cannot be shared between threads safely
--> src/main.rs:25:13
|
25  |     let t = thread::spawn(move || {
|             ^^^^^^^^^^^^^ `*const u8` cannot be shared between threads safely
|
= help: within `MyBox`, the trait `Sync` is not implemented for `*const u8`
```

因为我们访问的引用实际上还是对主线程中的数据的借用，转移进来的仅仅是外层的智能指针引用。要解决很简单，为`MyBox`实现`Sync`:

```rust
unsafe impl Sync for MyBox {}
```

# 总结

通过上面的两个裸指针的例子，我们了解了如何实现`Send`和`Sync`，以及如何只实现`Send`而不实现`Sync`，简单总结下：

1. 实现`Send`的类型可以在线程间安全的传递其所有权, 实现`Sync`的类型可以在线程间安全的共享(通过引用)
2. 绝大部分类型都实现了`Send`和`Sync`，常见的未实现的有：裸指针、`Cell`、`RefCell`、`Rc` 等
3. 可以为自定义类型实现`Send`和`Sync`，但是需要`unsafe`代码块
4. 可以为部分 Rust 中的类型实现`Send`、`Sync`，但是需要使用`newtype`，例如文中的裸指针例子