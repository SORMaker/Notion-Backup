# Smart Pointers

# Box<T>

`Box<T>` 允许你将一个值分配到堆上，然后在栈上保留一个智能指针指向堆上的数据，这是rust中**最常见**的智能指针。

## Rust中的堆栈

栈内存从高位地址向下增长，且栈内存是连续分配的，一般来说**操作系统对栈内存的大小都有限制。**在 Rust 中，`main` 线程的[栈大小是 `8MB`](https://course.rs/compiler/pitfalls/stack-overflow.html)，普通线程是 `2MB`，在函数调用时会在其中创建一个临时栈空间，调用结束后 Rust 会让这个栈空间里的对象自动进入 `Drop` 流程，最后栈顶指针自动移动到上一个调用栈顶，无需程序员手动干预，因而栈内存申请和释放是非常高效的。

与栈相反，堆上内存则是从低位地址向上增长，**堆内存通常只受物理内存限制**，而且通常是不连续的，因此从性能的角度看，栈往往比堆更高。

### Rust 堆上对象的特殊之处

它们都拥有一个所有者，因此受所有权规则的限制：当赋值时，发生的是所有权的转移（只需浅拷贝栈上的引用或智能指针即可），例如以下代码：

```rust
fn main() {
    let b = foo("world");
    println!("{}", b);
}

fn foo(x: &str) -> String {
    let a = "Hello, ".to_string() + x;
    a
}
```

在 `foo` 函数中，`a` 是 `String` 类型，它其实是一个智能指针结构体，该智能指针存储在函数栈中，指向堆上的字符串数据。当被从 `foo` 函数转移给 `main` 中的 `b` 变量时，栈上的智能指针被复制一份赋予给 `b`，而底层数据无需发生改变，这样就完成了所有权从 `foo` 函数内部到 `b` 的转移。

在栈上的数据，一般都实现了Copy trait，所以不会发生所有权转移，而是复制一份数据，两个数据同时存在。

```rust
fn main() {
    let x = 32;
    let y = x;

    println!("x 的值是: {}", x);
    println!("y 的值是: {}", y);
}
```

### 堆栈的性能

很多人可能会觉得栈的性能肯定比堆高，其实未必。 由于我们在后面的性能专题会专门讲解堆栈的性能问题，因此这里就大概给出结论：

- 小型数据，在栈上的分配性能和读取性能都要比堆上高
- 中型数据，栈上分配性能高，但是读取性能和堆上并无区别，因为无法利用寄存器或 CPU 高速缓存，最终还是要经过一次内存寻址
- 大型数据，只建议在堆上分配和使用

总之，栈的分配速度肯定比堆上快，但是读取速度往往取决于你的数据能不能放入寄存器或 CPU 高速缓存。 因此不要仅仅因为堆上性能不如栈这个印象，就总是优先选择栈，导致代码更复杂的实现。

## Box的使用场景

`Box`是简单的封装，除了将值存储在堆上外，并没有其它性能上的损耗。而性能和功能往往是鱼和熊掌，因此 `Box` 相比其它智能指针，功能较为单一，可以在以下场景中使用它：

- 特意的将数据分配在堆上
- 数据较大时，又不想在转移所有权时进行数据拷贝
- 类型的大小在编译期无法确定，但是我们又需要固定大小的类型时
- 特征对象，用于说明对象实现了一个特征，而不是某个特定的类型

### 使用Box<T>将数据存储在堆上

如果一个变量拥有一个数值 `let a = 3`，那变量 `a` 必然是存储在栈上的，那如果我们想要 `a` 的值存储在堆上就需要使用 `Box<T>`：

```rust
fn main() {
    let a = Box::new(3);
    println!("a = {}", a); // a = 3

    // 下面一行代码将报错
    // let b = a + 1; // cannot add `{integer}` to `Box<{integer}>`
}
```

并且因为智能指针往往都实现了 `Deref` 和 `Drop` 特征，因此：

- `println!` 可以正常打印出 `a` 的值，是因为它隐式地调用了 `Deref` 对智能指针 `a` 进行了解引用
- 最后一行代码 `let b = a + 1` 报错，是因为在表达式中，我们无法自动隐式地执行 `Deref` 解引用操作，你需要使用  操作符 `let b = *a + 1`，来显式的进行解引用
- `a` 持有的智能指针将在作用域结束（`main` 函数结束）时，被释放掉，这是因为 `Box<T>` 实现了 `Drop` 特征

### 将动态大小类型变为Sized固定大小类型

Rust 需要在编译时知道类型占用多少空间，如果一种类型在编译时无法知道具体的大小，那么被称为动态大小类型 DST。

其中一种无法在编译时知道大小的类型是**递归类型**：在类型定义中又使用到了自身，或者说该类型的值的一部分可以是相同类型的其它值，这种值的嵌套理论上可以无限进行下去，所以 Rust 不知道递归类型需要多少空间：

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

以上就是函数式语言中常见的 `Cons List`，它的每个节点包含一个 `i32` 值，还包含了一个新的 `List`，因此这种嵌套可以无限进行下去，Rust 认为该类型是一个 DST 类型，并给予报错，想要解决这个问题，我们可以使用`Box<T>`：

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```

只需要将 `List` 存储到堆上，然后使用一个智能指针指向它，即可完成从 DST 到 Sized 类型（固定大小类型）的华丽转变。

### 特征对象

在 Rust 中，想实现不同类型组成的数组只有两个办法：枚举和特征对象，前者限制较多，因此后者往往是最常用的解决办法。

```rust
trait Draw {
    fn draw(&self);
}

struct Button {
    id: u32,
}
impl Draw for Button {
    fn draw(&self) {
        println!("这是屏幕上第{}号按钮", self.id)
    }
}

struct Select {
    id: u32,
}

impl Draw for Select {
    fn draw(&self) {
        println!("这个选择框贼难用{}", self.id)
    }
}

fn main() {
    let elems: Vec<Box<dyn Draw>> = vec![Box::new(Button { id: 1 }), Box::new(Select { id: 2 })];

    for e in elems {
        e.draw()
    }
}
```

以上代码将不同类型的 `Button` 和 `Select` 包装成 `Draw` 特征的特征对象，放入一个数组中，`Box<dyn Draw>` 就是特征对象，只要是实现了`Draw`特征的对象都可以放进来。

其实，特征也是 DST 类型，而特征对象在做的就是将 DST 类型转换为固定大小类型。

## Box内存布局

来看看 `Vec<Box<i32>>` 的内存布局：

```rust
                    (heap)
(stack)    (heap)   ┌───┐
┌──────┐   ┌───┐ ┌─→│ 1 │
│ vec2 │──→│B1 │─┘  └───┘
└──────┘   ├───┤    ┌───┐
           │B2 │───→│ 2 │
           ├───┤    └───┘
           │B3 │─┐  ┌───┐
           ├───┤ └─→│ 3 │
           │B4 │─┐  └───┘
           └───┘ │  ┌───┐
                 └─→│ 4 │
                    └───┘
```

上面的 `B1` 代表被 `Box` 分配到堆上的值 `1`。

可以看出智能指针 `vec2` 依然是存储在栈上，然后指针指向一个堆上的数组，该数组中每个元素都是一个 `Box` 智能指针，最终 `Box` 智能指针又指向了存储在堆上的实际值。

因此当我们从数组中取出某个元素时，取到的是对应的智能指针 `Box`，需要对该智能指针进行解引用，才能取出最终的值：

```rust
fn main() {
    let arr = vec![Box::new(1), Box::new(2)];
    let (first, second) = (&arr[0], &arr[1]);
    let sum = **first + **second;
}
```

以上代码有几个值得注意的点：

- 使用`&`借用数组中的元素，否则会报所有权错误
- 表达式不能隐式的解引用，因此必须使用`*`做两次解引用，第一次将 `&Box<i32>` 类型转成 `Box<i32>`，第二次将 `Box<i32>` 转成 `i32`

## Box::leak

`Box` 中还提供了一个非常有用的关联函数：`Box::leak`，它可以消费掉 `Box` 并且强制目标值从内存中泄漏。

```rust
fn main() {
   let s = gen_static_str();
   println!("{}", s);
}

fn gen_static_str() -> &'static str{
    let mut s = String::new();
    s.push_str("hello, world");

    Box::leak(s.into_boxed_str())
}
```

这段代码中，我们将一个`String`类型，变成了一个 `'static` 生命周期的 `&str` 类型。

在之前的代码中，如果 `String` 创建于函数中，那么返回它的唯一方法就是转移所有权给调用者 `fn move_str() -> String`，而通过 `Box::leak` 我们不仅返回了一个 `&str` 字符串切片，它还是 `'static` 生命周期的！

要知道真正具有 `'static` 生命周期的往往都是编译期就创建的值，例如 `let v = "hello, world"`，这里 `v` 是直接打包到二进制可执行文件中的，因此该字符串具有 `'static` 生命周期，再比如 `const` 常量。

那我们就很奇怪了，我不是可以手动为变量标注 `'static` 吗？其实你标注的 `'static` 只是用来忽悠编译器的，但是超出作用域，一样被释放回收。而使用 `Box::leak` 就可以将一个运行期的值转为 `'static`。

### 使用场景

一个简单的场景，**你需要一个在运行期初始化的值，但是可以全局有效，也就是和整个程序活得一样久**，那么就可以使用 `Box::leak`，例如有一个存储配置的结构体实例，它是在运行期动态插入内容，那么就可以将其转为全局有效，虽然 `Rc/Arc` 也可以实现此功能，但是 `Box::leak` 是性能最高的。

# Deref

何为智能指针？能不让你写出 `****s` 形式的解，智能指针的名称来源，主要就在于它实现了 `Deref` 和 `Drop` 特征，这两个特征可以智能地帮助我们节省使用上的负担：

- `Deref` 可以让智能指针像引用那样工作，这样你就可以写出同时支持智能指针和引用的代码，例如 `T`
- `Drop` 允许你指定智能指针超出作用域后自动执行的代码，例如做一些数据清除等收尾工作

常规的解引用这里不过多赘述。

## 智能指针解引用

Rust 中将解引用提升到了一个新高度。

实现 `Deref` 后的智能指针结构体，就可以像普通引用一样，通过 `*` 进行解引用，例如 `Box<T>` 智能指针：

```rust
fn main() {
    let x = Box::new(1);
    let sum = *x + 1;
}
```

智能指针 `x` 被`*`解引用为 `i32` 类型的值 `1`，然后再进行求和。

### 定义自己的智能指针

由于 `Box<T>` 本身很简单，并没有包含类如长度、最大长度等信息，因此用一个元组结构体即可。并为他实现`Deref`特征，让我们自己定义的智能指针可以支持`*`解引用操作符。

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}
```

当解引用 `MyBox` 智能指针时，返回元组结构体中的元素 `&self.0`，有几点要注意的：

- 在 `Deref` 特征中声明了关联类型 `Target`，在之前章节中介绍过，关联类型主要是为了提升代码可读性
- `deref` 返回的是一个常规引用，可以被进行解引用

## *背后的原理

当我们对智能指针 `Box` 进行解引用时，实际上 Rust 为我们调用了以下方法：

```rust
*(y.deref())
```

首先调用 `deref` 方法返回值的常规引用，然后通过 `*` 对常规引用进行解引用，最终获取到目标值。

至于 Rust 为何要使用这个有点啰嗦的方式实现，原因在于所有权系统的存在。如果 `deref` 方法直接返回一个值，而不是引用，那么该值的所有权将被转移给调用者，而我们不希望调用者仅仅只是 `*T` 一下，就拿走了智能指针中包含的值。

需要注意的是，`*` 不会无限递归替换，从 `*y` 到 `*(y.deref())` 只会发生一次，而不会继续进行替换然后产生形如 `*((y.deref()).deref())` 的怪物。

## 隐式Deref转换

隐式Deref转换只存在于函数和方法中，我们不能在表达式中进行隐式Deref转换。

若一个类型实现了 `Deref` 特征，那它的引用在传给函数或方法时，会根据参数签名来决定是否进行隐式的 `Deref` 转换，例如：

```rust
fn main() {
    let s = String::from("hello world");
    display(&s)
}

fn display(s: &str) {
    println!("{}",s);
}
```

以上代码有几点值得注意：

- `String` 实现了 `Deref` 特征，可以在需要时自动被转换为 `&str` 类型
- `&s` 是一个 `&String` 类型，当它被传给 `display` 函数时，自动通过 `Deref` 转换成了 `&str`
- 必须使用 `&s` 的方式来触发 `Deref`（仅引用类型的实参才会触发自动解引用）

### 连续的隐式Deref转换

`Deref` 可以支持连续的隐式转换，直到找到适合的形式为止：

```rust
fn main() {
    let s = MyBox::new(String::from("hello world"));
    display(&s)
}

fn display(s: &str) {
    println!("{}",s);
}
```

这里我们使用了之前自定义的智能指针 `MyBox`，并将其通过连续的隐式转换变成 `&str` 类型：首先 `MyBox` 被 `Deref` 成 `String` 类型，结果并不能满足 `display` 函数参数的要求，编译器发现 `String` 还可以继续 `Deref` 成 `&str`，最终成功的匹配了函数参数。

如果没有隐式转换，我们需要这样调用`display`函数：

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    display(&(*m)[..]);
}
// *m -> String
// &(*m) -> &String
// &(*m)[..] -> &str
```

再来看一下在方法、赋值中自动应用 `Deref` 的例子：

```rust

fn main() {
    let s = MyBox::new(String::from("hello, world"));
    let s1: &str = &s;
    let s2: String = s.to_string();
}
```

对于 `s1`，我们通过两次 `Deref` 将 `&str` 类型的值赋给了它（**赋值操作需要手动解引用**）；而对于 `s2`，我们在其上直接调用方法 `to_string`，实际上 `MyBox` 根本没有没有实现该方法，能调用 `to_string`，完全是因为编译器对 `MyBox` 应用了 `Deref` 的结果（**方法调用会自动解引用**）。

## Deref规则总结

### 引用归一化

Rust 会在解引用时自动把智能指针和 `&&&&v` 做引用归一化操作，转换成 `&v` 形式，最终再对 `&v` 进行解引用：

- 把智能指针（比如在库中定义的，Box、Rc、Arc、Cow 等）从结构体脱壳为内部的引用类型，也就是转成结构体内部的 `&v`
- 把多重`&`，例如 `&&&&&&&v`，归一成 `&v`

关于第二种情况，这么干巴巴的说，也许大家会迷迷糊糊的，我们来看一段标准库源码：

```rust

impl<T: ?Sized> Deref for &T {
    type Target = T;

    fn deref(&self) -> &T {
        *self
    }
}

```

在这段源码中，`&T` 被自动解引用为 `T`，也就是 `&T: Deref<Target=T>`。 按照这个代码，`&&&&T` 会被自动解引用为 `&&&T`，然后再自动解引用为 `&&T`，以此类推，直到最终变成 `&T`。

## 三种Deref转换

在之前，我们讲的都是不可变的 `Deref` 转换，实际上 Rust 还支持将一个可变的引用转换成另一个可变的引用以及将一个可变引用转换成不可变的引用，规则如下：

- 当 `T: Deref<Target=U>`，可以将 `&T` 转换成 `&U`，也就是我们之前看到的例子
- 当 `T: DerefMut<Target=U>`，可以将 `&mut T` 转换成 `&mut U`
- 当 `&mut T: Deref<Target=U>`，可以将 `&mut T` 转换成 `&U`

```rust
struct MyBox<T> {
    v: T,
}

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox { v: x }
    }
}

use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.v
    }
}

use std::ops::DerefMut;

impl<T> DerefMut for MyBox<T> {
    fn deref_mut(&mut self) -> &mut Self::Target {
        &mut self.v
    }
}

fn main() {
    let mut s = MyBox::new(String::from("hello, "));
    display(&mut s)
}

fn display(s: &mut String) {
    s.push_str("world");
    println!("{}", s);
}
```

以上代码有几点值得注意：

- 要实现 `DerefMut` 必须要先实现 `Deref` 特征：`pub trait DerefMut: Deref`
- `T: DerefMut<Target=U>` 解读：将 `&mut T` 类型通过 `DerefMut` 特征的方法转换为 `&mut U` 类型，对应上例中，就是将 `&mut MyBox<String>` 转换为 `&mut String`

# Drop释放资源

## Rust中的资源回收

在 Rust 中，你可以指定在一个变量超出作用域时，执行一段特定的代码，最终编译器将帮你自动插入这段收尾代码。这样，就无需在每一个使用该变量的地方，都写一段代码来进行收尾工作和资源释放。

## 一个不那么简单的Drop例子

```rust
struct HasDrop1;
struct HasDrop2;
impl Drop for HasDrop1 {
    fn drop(&mut self) {
        println!("Dropping HasDrop1!");
    }
}
impl Drop for HasDrop2 {
    fn drop(&mut self) {
        println!("Dropping HasDrop2!");
    }
}
struct HasTwoDrops {
    one: HasDrop1,
    two: HasDrop2,
}
impl Drop for HasTwoDrops {
    fn drop(&mut self) {
        println!("Dropping HasTwoDrops!");
    }
}

struct Foo;

impl Drop for Foo {
    fn drop(&mut self) {
        println!("Dropping Foo!")
    }
}

fn main() {
    let _x = HasTwoDrops {
        two: HasDrop2,
        one: HasDrop1,
    };
    let _foo = Foo;
    println!("Running!");
}
```

上面代码虽然长，但是目的其实很单纯，就是为了观察不同情况下变量级别的、结构体内部字段的 `Drop`，有几点值得注意：

- `Drop` 特征中的 `drop` 方法借用了目标的可变引用，而不是拿走了所有权，这里先设置一个悬念，后边会讲
- 结构体中每个字段都有自己的 `Drop`

输出：

```rust
Running!
Dropping Foo!
Dropping HasTwoDrops!
Dropping HasDrop1!
Dropping HasDrop2!
```

### Drop的顺序

观察以上输出，我们可以得出以下关于 `Drop` 顺序的结论

- **变量级别，按照逆序的方式**，`_x` 在 `_foo` 之前创建，因此 `_x` 在 `_foo` 之后被 `drop`
- **结构体内部，按照顺序的方式**，结构体 `_x` 中的字段按照定义中的顺序依次 `drop`

### 没有实现Drop的结构体

及时移除`HasTwoDrops`的`Drop`特征，依然可以执行内部两个字段的`Drop`代码。

也就是移除下面这段代码：

```rust
impl Drop for HasTwoDrops {
    fn drop(&mut self) {
        println!("Dropping HasTwoDrops!");
    }
}
```

最终输出为：

```rust
Dropping HasDrop1!
Dropping HasDrop2!
```

Rust 自动为几乎所有类型都实现了 `Drop` 特征，因此就算你不手动为结构体实现 `Drop`，它依然会调用默认实现的 `drop` 函数，同时再调用每个字段的 `drop` 方法。

## 手动回收

当使用智能指针来管理锁的时候，你可能希望提前释放这个锁，然后让其它代码能及时获得锁，此时就需要提前去手动 `drop`。 但是在之前我们提到一个悬念，`Drop::drop` 只是借用了目标值的可变引用，所以，就算你提前调用了 `drop`，后面的代码依然可以使用目标值，但是这就会访问一个并不存在的值，非常不安全，好在 Rust 会阻止你：

```rust
#[derive(Debug)]
struct Foo;

impl Drop for Foo {
    fn drop(&mut self) {
        println!("Dropping Foo!")
    }
}

fn main() {
    let foo = Foo;
    foo.drop();
    println!("Running!:{:?}", foo);
}

error[E0040]: explicit use of destructor method
  --> src/main.rs:37:9
   |
37 |     foo.drop();
   |     ----^^^^--
   |     |   |
   |     |   explicit destructor calls not allowed
   |     help: consider using `drop` function: `drop(foo)`

```

如上所示，编译器直接阻止了我们调用 `Drop` 特征的 `drop` 方法，原因是对于 Rust 而言，不允许显式地调用析构函数（这是一个用来清理实例的通用编程概念）。好在在报错的同时，编译器还给出了一个提示：使用 `drop` 函数。

针对编译器提示的 `drop` 函数，我们可以大胆推测下：它能够拿走目标值的所有权。现在来看看这个猜测正确与否，以下是 `std::mem::drop` 函数的签名：

```rust
pub fn drop<T>(_x: T)
```

如上所示，`drop` 函数确实拿走了目标值的所有权，来验证下：

```rust

fn main() {
    let foo = Foo;
    drop(foo);
// 以下代码会报错：借用了所有权被转移的值
// println!("Running!:{:?}", foo);
}
```

<aside>
💡

事实上，能被显式调用的drop(_x)函数只是个空函数，在拿走目标值的所有权后没有任何操作。而由于其持有目标值的所有权，在drop(_x)函数结束之际，编译器会执行_x真正的析构函数，从而完成释放资源的操作。换句话说，drop(_x)函数只是帮助目标值的所有者提前离开了作用域。

</aside>

## Drop的使用场景

对于 Drop 而言，主要有两个功能：

- 回收内存资源
- 执行一些收尾工作

对于第二点，在之前我们已经详细介绍过，因此这里主要对第一点进行下简单说明。

在绝大多数情况下，我们都无需手动去 `drop` 以回收内存资源，因为 Rust 会自动帮我们完成这些工作，它甚至会对复杂类型的每个字段都单独的调用 `drop` 进行回收！但是确实有极少数情况，需要你自己来回收资源的，例如文件描述符、网络 socket 等，当这些值超出作用域不再使用时，就需要进行关闭以释放相关的资源，在这些情况下，就需要使用者自己来解决 `Drop` 的问题。

## 互斥的Copy和Drop

我们无法为一个类型同时实现 `Copy` 和 `Drop` 特征。因为实现了 `Copy` 特征的类型会被编译器隐式的复制，因此非常难以预测析构函数执行的时间和频率。因此这些实现了 `Copy` 的类型无法拥有析构函数。

```rust
#[derive(Copy)]
struct Foo;

impl Drop for Foo {
    fn drop(&mut self) {
        println!("Dropping Foo!")
    }
}
error[E0184]: the trait `Copy` may not be implemented for this type; the type has a destructor
  --> src/main.rs:24:10
   |
24 | #[derive(Copy)]
   |          ^^^^ Copy not allowed on types with destructors

```

实际上，互斥这一点不仅仅是难以预测，更重要的是内存安全（资源正确释放）方面的考虑。

copy可以理解为栈内存的简单复制，通常意义上的浅拷贝，trivial copy。

简单举例来说，有一个结构体只包含一个指针，这个指针指向分配出来的堆内存。

它实现了Drop，作用是释放堆上的内存（类似于c++里面的析构函数）。编译器做的工作是在栈上分配的类或者结构体，在离开作用域时自动插入析构的函数。

copy会把这个指针复制一遍，这时候就有两个结构体在栈上。

结构体离开了作用域，会调用drop，这个时候就会调用两遍析构，但是结构体管理的实际资源（堆上的一段内存）只有一个，此时资源就被释放两遍了。这是一种内存错误。

解决方法就是

1. 不仅仅复制栈上的结构，我复制这个结构体的时候把资源也复制一份。也就是clone trait。
2. 使用智能指针，给资源做一个引用计数，结构体作为引用计数和资源的控制结构。每次出作用域的时候，就检查一遍引用计数，判断此时可以不可以释放。
3. 禁止复制，也就是move语义，资源的控制结构只存在一个，这个控制结构拥有所有权。move语义的赋值其实也是仅仅复制栈上的结构，但是编译器帮我记住现有栈上有效的结构到底是哪一个。我要是用错了，就报错。
4. 有gc的，就用gc来释放。

# Rc与Arc

Rust 所有权机制要求一个值只能有一个所有者，在大多数情况下，都没有问题，但是考虑以下情况：

- 在图数据结构中，多个边可能会拥有同一个节点，该节点直到没有边指向它时，才应该被释放清理
- 在多线程中，多个线程可能会持有同一个数据，但是你受限于 Rust 的安全机制，无法同时获取该数据的可变引用

以上场景不是很常见，但是一旦遇到，就非常棘手，为了解决此类问题，Rust 在所有权机制之外又引入了额外的措施来简化相应的实现：通过引用计数的方式，允许一个数据资源在同一时刻拥有多个所有者。

这种实现机制就是 `Rc` 和 `Arc`，前者适用于单线程，后者适用于多线程。

## Rc<T>

引用计数(reference counting)，顾名思义，通过记录一个数据被引用的次数来确定该数据是否正在被使用。当引用次数归零时，就代表该数据不再被使用，因此可以被清理释放。

而 `Rc` 正是引用计数的英文缩写。当我们**希望在堆上分配一个对象供程序的多个部分使用且无法确定哪个部分最后一个结束时，就可以使用 `Rc` 成为数据值的所有者**

下面是经典的所有权被转移导致报错的例子：

```rust
fn main() {
    let s = String::from("hello, world");
// s在这里被转移给alet a = Box::new(s);
// 报错！此处继续尝试将 s 转移给 blet b = Box::new(s);
}
```

使用 `Rc` 就可以轻易解决：

```rust
use std::rc::Rc;
fn main() {
    let a = Rc::new(String::from("hello, world"));
    let b = Rc::clone(&a);

    assert_eq!(2, Rc::strong_count(&a));
    assert_eq!(Rc::strong_count(&a), Rc::strong_count(&b))
}
```

以上代码我们使用 `Rc::new` 创建了一个新的 `Rc<String>` 智能指针并赋给变量 `a`，该指针指向底层的字符串数据。

智能指针 `Rc<T>` 在创建时，还会将引用计数加1，此时获取引用计数的关联函数 `Rc::strong_count` 返回的值将是 `1`。

### Rc::clone

接着，我们又使用 `Rc::clone` 克隆了一份智能指针 `Rc<String>`，并将该智能指针的引用计数增加到 `2`。

由于 `a` 和 `b` 是同一个智能指针的两个副本，因此通过它们两个获取引用计数的结果都是 `2`。

不要被 `clone` 字样所迷惑，以为所有的 `clone` 都是深拷贝。这里的 `clone` **仅仅复制了智能指针并增加了引用计数，并没有克隆底层数据**，因此 `a` 和 `b` 是共享了底层的字符串 `s`，这种**复制效率是非常高**的。当然你也可以使用 `a.clone()` 的方式来克隆，但是从可读性角度，我们更加推荐 `Rc::clone` 的方式。

实际上在 Rust 中，还有不少 `clone` 都是浅拷贝，例如[迭代器的克隆](https://course.rs/compiler/pitfalls/iterator-everywhere.html)。

### 观察引用计数的变化

使用关联函数 `Rc::strong_count` 可以获取当前引用计数的值，我们来观察下引用计数如何随着变量声明、释放而变化：

```rust
use std::rc::Rc;
fn main() {
        let a = Rc::new(String::from("test ref counting"));
        println!("count after creating a = {}", Rc::strong_count(&a));
        let b =  Rc::clone(&a);
        println!("count after creating b = {}", Rc::strong_count(&a));
        {
            let c =  Rc::clone(&a);
            println!("count after creating c = {}", Rc::strong_count(&c));
        }
        println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
```

有几点值得注意：

- 由于变量 `c` 在语句块内部声明，当离开语句块时它会因为超出作用域而被释放，所以引用计数会减少1，事实上这个得益于 `Rc<T>` 实现了 `Drop` 特征
- `a`、`b`、`c` 三个智能指针引用计数都是同样的，并且共享底层的数据，因此打印计数时用哪个都行
- 无法看到的是：当 `a`、`b` 超出作用域后，引用计数会变成0，最终智能指针和它指向的底层字符串都会被清理释放

### 不可变引用

事实上，`Rc<T>` 是指向底层数据的不可变的引用，因此你无法通过它来修改数据，这也符合 Rust 的借用规则：要么存在多个不可变借用，要么只能存在一个可变借用。

但是实际开发中我们往往需要对数据进行修改，这时单独使用 `Rc<T>` 无法满足我们的需求，需要配合其它数据类型来一起使用，例如内部可变性的 `RefCell<T>` 类型以及互斥锁 `Mutex<T>`。事实上，在多线程编程中，`Arc` 跟 `Mutex` 锁的组合使用非常常见，它们既可以让我们在不同的线程中共享数据，又允许在各个线程中对其进行修改。

### 简单总结

- `Rc/Arc` 是不可变引用，你无法修改它指向的值，只能进行读取，如果要修改，需要配合后面章节的内部可变性 `RefCell` 或互斥锁 `Mutex`
- 一旦最后一个拥有者消失，则资源会自动被回收，这个生命周期是在编译期就确定下来的
- `Rc` 只能用于同一线程内部，想要用于线程之间的对象共享，你需要使用 `Arc`
- `Rc<T>` 是一个智能指针，实现了 `Deref` 特征，因此你无需先解开 `Rc` 指针，再使用里面的 `T`，而是可以直接使用 `T`。

## Arc

`Arc` 是 `Atomic Rc` 的缩写，顾名思义：原子化的 `Rc<T>` 智能指针。

为什么需要Arc，而不是统一使用Rc。

原因在于原子化或者其它锁虽然可以带来的线程安全，但是都会伴随着性能损耗，而且这种性能损耗还不小。因此 Rust 把这种选择权交给你，毕竟需要线程安全的代码其实占比并不高，大部分时候我们开发的程序都在一个线程内。

`Arc` 和 `Rc` 拥有完全一样的 API，修改起来很简单：

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let s = Arc::new(String::from("多线程漫游者"));
    for _ in 0..10 {
        let s = Arc::clone(&s);
        let handle = thread::spawn(move || {
           println!("{}", s)
        });
    }
}
```

对了，两者还有一点区别：`Arc` 和 `Rc` 并没有定义在同一个模块，前者通过 `use std::sync::Arc` 来引入，后者通过 `use std::rc::Rc`。

# Cell和RefCell

但是严格是一把双刃剑，带来安全提升的同时，损失了灵活性，有时甚至会让用户痛苦不堪、怨声载道。因此 Rust 提供了 `Cell` 和 `RefCell` 用于内部可变性，简而言之，可以在拥有不可变引用的同时修改目标数据，对于正常的代码实现来说，这个是不可能做到的（要么一个可变借用，要么多个不可变借用）。

<aside>
💡

内部可变性的实现是因为 Rust 使用了 `unsafe` 来做到这一点，但是对于使用者来说，这些都是透明的，因为这些不安全代码都被封装到了安全的 API 中

</aside>

## Cell

`Cell` 和 `RefCell` 在功能上没有区别，区别在于 `Cell<T>` 适用于 `T` 实现 `Copy` 的情况：

```rust

use std::cell::Cell;
fn main() {
  let c = Cell::new("asdf");
  let one = c.get();
  c.set("qwer");
  let two = c.get();
  println!("{},{}", one, two);
}
```

以上代码展示了 `Cell` 的基本用法，有几点值得注意：

- "asdf"是 `&str` 类型，它实现了 `Copy` 特征
- `c.get` 用来取值，`c.set` 用来设置新值

取到值保存在 `one` 变量后，还能同时进行修改，这个违背了 Rust 的借用规则，但是由于 `Cell` 的存在，我们很优雅地做到了这一点，但是如果你尝试在 `Cell` 中存放`String`：

```rust
 let c = Cell::new(String::from("asdf"));
```

编译器会立刻报错，因为 `String` 没有实现 `Copy` 特征：

```rust
| pub struct String {
| ----------------- doesn't satisfy `String: Copy`
|
= note: the following trait bounds were not satisfied:
        `String: Copy`
```

## RefCell

由于 `Cell` 类型针对的是实现了 `Copy` 特征的值类型，因此在实际开发中，`Cell` 使用的并不多，因为我们要解决的往往是可变、不可变引用共存导致的问题，此时就需要借助于 `RefCell` 来达成目的。

我们可以将所有权、借用规则与这些智能指针做一个对比：

| Rust 规则 | 智能指针带来的额外规则 |
| --- | --- |
| 一个数据只有一个所有者 | `Rc/Arc`让一个数据可以拥有多个所有者 |
| 要么多个不可变借用，要么一个可变借用 | `RefCell`实现编译期可变、不可变引用共存 |
| 违背规则导致**编译错误** | 违背规则导致**运行时`panic`** |

可以看出，`Rc/Arc` 和 `RefCell` 合在一起，解决了 Rust 中严苛的所有权和借用规则带来的某些场景下难使用的问题。但是它们并不是银弹，例如 `RefCell` 实际上并没有解决可变引用和引用可以共存的问题，只是将报错从编译期推迟到运行时，从编译器错误变成了 `panic` 异常：

### RefCell为何存在？

存在即合理，究其根因，在于 Rust 编译期的**宁可错杀，绝不放过**的原则，当编译器不能确定你的代码是否正确时，就统统会判定为错误，因此难免会导致一些误报。

### RefCell简单总结

- 与 `Cell` 用于可 `Copy` 的值不同，`RefCell` 用于引用
- `RefCell` 只是将借用规则从编译期推迟到程序运行期，并不能帮你绕过这个规则
- `RefCell` 适用于编译期误报或者一个引用被在多处代码使用、修改以至于难于管理借用关系时
- 使用 `RefCell` 时，违背借用规则会导致运行期的 `panic`

## Cell or RefCell

根据本文的内容，我们可以大概总结下两者的区别：

- `Cell` 只适用于 `Copy` 类型，用于提供值，而 `RefCell` 用于提供引用
- `Cell` 不会 `panic`，而 `RefCell` 会

### 性能比较

`Cell` 没有额外的性能损耗，`zero cost`。

`RefCell` 有一点运行期开销，原因是它包含了一个字节大小的“借用状态”指示器，该指示器在每次运行时借用时都会被修改，进而产生一点开销。

## 内部可变性

之前我们提到 `RefCell` 具有内部可变性，何为内部可变性？简单来说，对一个不可变的值进行可变借用，但这个并不符合 Rust 的基本借用规则。

但是有些场景，我们希望一个值可以在方法内部被修改，同时对其他代码是不可变的，比如下面这种情况：

```rust
// 定义在外部库中的特征
pub trait Messenger {
    fn send(&self, msg: String);
}

// --------------------------
// 我们的代码中的数据结构和实现
struct MsgQueue {
    msg_cache: Vec<String>,
}

impl Messenger for MsgQueue {
    fn send(&self, msg: String) {
        self.msg_cache.push(msg)
    }
}
```

如上，在外部库中有一个消息发送器`trait`，它只有一个发送消息的功能，因为发送消息不需要修改自身，所以原作者只使用了`&self`，这没问题。

但是我们要在自己的代码中实现时，出于性能的考虑，我们希望先将消息写到本地缓存中，然后批量发送，因此我们先插入到本地缓存`msg_cache`中。那么问题来了，`send`方法的签名是`&self`，因此上述代码会报错，这时候就需要`RefCell`了。

我们可以使用`RefCell`包裹一下`cache` ，然后调用`borrow_mut`方法，就能够将不可变借用变成可变的。

```rust
use std::cell::RefCell;
pub trait Messenger {
    fn send(&self, msg: String);
}

pub struct MsgQueue {
    msg_cache: RefCell<Vec<String>>,
}

impl Messenger for MsgQueue {
    fn send(&self, msg: String) {
        self.msg_cache.borrow_mut().push(msg)
    }
}

fn main() {
    let mq = MsgQueue {
        msg_cache: RefCell::new(Vec::new()),
    };
    mq.send("hello, world".to_string());
}
```

## Rc + RefCell

在 Rust 中，一个常见的组合就是 `Rc` 和 `RefCell` 在一起使用，前者可以实现一个数据拥有多个所有者，后者可以实现数据的可变性：

```rust
use std::cell::RefCell;
use std::rc::Rc;
fn main() {
    let s = Rc::new(RefCell::new("我很善变，还拥有多个主人".to_string()));

    let s1 = s.clone();
    let s2 = s.clone();
// let mut s2 = s.borrow_mut();
    s2.borrow_mut().push_str(", oh yeah!");

    println!("{:?}\n{:?}\n{:?}", s, s1, s2);
}
```

上面代码中，我们使用 `RefCell<String>` 包裹一个字符串，同时通过 `Rc` 创建了它的三个所有者：`s`、`s1`和`s2`，并且通过其中一个所有者 `s2` 对字符串内容进行了修改。

由于 `Rc` 的所有者们共享同一个底层的数据，因此当一个所有者修改了数据时，会导致全部所有者持有的数据都发生了变化。

程序的运行结果也在预料之中：

```
RefCell { value: "我很善变，还拥有多个主人, oh yeah!" }
RefCell { value: "我很善变，还拥有多个主人, oh yeah!" }
RefCell { value: "我很善变，还拥有多个主人, oh yeah!" }
```

### 性能损耗

这两者结合在一起使用的性能其实非常高，大致相当于没有线程安全版本的C++ `std::shared_ptr` 指针

### 内存损耗

从对内存的影响来看，大约是仅仅多分配了三个`usize/isize`，并没有其它额外的负担。

### CPU损耗

从 CPU 来看，损耗如下：

- 对 `Rc<T>` 解引用是免费的（编译期），但是  带来的间接取值并不免费
- 克隆 `Rc<T>` 需要将当前的引用计数跟 `0` 和 `usize::Max` 进行一次比较，然后将计数值加 1
- 释放（drop） `Rc<T>` 需要将计数值减 1， 然后跟 `0` 进行一次比较
- 对 `RefCell` 进行不可变借用，需要将 `isize` 类型的借用计数加 1，然后跟 `0` 进行比较
- 对 `RefCell` 的不可变借用进行释放，需要将 `isize` 减 1
- 对 `RefCell` 的可变借用大致流程跟上面差不多，但是需要先跟 `0` 比较，然后再减 1
- 对 `RefCell` 的可变借用进行释放，需要将 `isize` 加 1

其实这些细节不必过于关注，只要知道 CPU 消耗也非常低，甚至编译器还会对此进行进一步优化！

### CPU缓存Miss

唯一需要担心的可能就是这种组合数据结构对于 CPU 缓存是否亲和，这个我们无法证明，只能提出来存在这个可能性，最终的性能影响还需要在实际场景中进行测试。

总之，分析这两者组合的性能还挺复杂的，大概总结下：

- 从表面来看，它们带来的内存和 CPU 损耗都不大
- 但是由于 `Rc` 额外的引入了一次间接取值（），在少数场景下可能会造成性能上的显著损失
- CPU 缓存可能也不够亲和

## 通过Cell::from_mut解决借用冲突

在 Rust 1.37版本中新增了两个非常实用的方法：

- Cell::from_mut，该方法将 `&mut T` 转为 `&Cell<T>`
- Cell::as_slice_of_cells，该方法将 `&Cell<[T]>` 转为 `&[Cell<T>]`

这里我们不做深入的介绍，但是来看看如何使用这两个方法来解决一个常见的借用冲突问题：

```rust
fn is_even(i: i32) -> bool {
    i % 2 == 0
}

fn retain_even(nums: &mut Vec<i32>) {
    let mut i = 0;
    for num in nums.iter().filter(|&num| is_even(*num)) {
        nums[i] = *num;
        i += 1;
    }
    nums.truncate(i);
}
```

以上代码会报错：

```
error[E0502]: cannot borrow `*nums` as mutable because it is also borrowed as immutable
 --> src/main.rs:8:9
  |
7 |     for num in nums.iter().filter(|&num| is_even(*num)) {
  |                ----------------------------------------
  |                |
  |                immutable borrow occurs here
  |                immutable borrow later used here
8 |         nums[i] = *num;
  |         ^^^^ mutable borrow occurs here
```

很明显，报错是因为同时借用了不可变与可变引用，你可以通过索引的方式来避免这个问题：

```rust
fn retain_even(nums: &mut Vec<i32>) {
    let mut i = 0;
    for j in 0..nums.len() {
        if is_even(nums[j]) {
            nums[i] = nums[j];
            i += 1;
        }
    }
    nums.truncate(i);
}
```

但是这样就违背我们的初衷了，毕竟迭代器会让代码更加简洁，那么还有其它的办法吗？

这时就可以使用 `Cell` 新增的这两个方法：

```rust
use std::cell::Cell;

fn retain_even(nums: &mut Vec<i32>) {
    let slice: &[Cell<i32>] = Cell::from_mut(&mut nums[..])
        .as_slice_of_cells();

    let mut i = 0;
    for num in slice.iter().filter(|num| is_even(num.get())) {
        slice[i].set(num.get());
        i += 1;
    }

    nums.truncate(i);
}
```

此时代码将不会报错，因为 `Cell` 上的 `set` 方法获取的是不可变引用 `pub fn set(&self, val: T)`。

当然，以上代码的本质还是对 `Cell` 的运用，只不过这两个方法可以很方便的帮我们把 `&mut [T]` 类型转换成 `&[Cell<T>]` 类型。