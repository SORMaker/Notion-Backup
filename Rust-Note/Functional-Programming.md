# Functional Programming

# Closure

闭包是**一种匿名函数，它可以赋值给变量也可以作为参数传递给其它函数，不同于函数的是，它允许捕获调用者作用域中的值。**

定义：

```rust
|param1, param2,...| {
语句1;
语句2;
返回表达式
}
或者
let some = |param1, param2,...| {
		语句1;
		语句2;
		返回表达式
		}
```

第二种方法，只是把闭包赋值给变量`some`,并不是将闭包执行后的结果赋值给`some`，因此这里的`some`就相当于闭包函数，可以跟函数一样进行调用：`some(param1, param2, ...)`

如果只有一个返回表达式的话，定义可以简化为：

```rust
|param1| 返回表达式
```

闭包特征`Fn(u32) -> u32`

```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    query: T,
    value: Option<u32>,
}
```

`Fn(u32) -> u32` 是一个特征，用来表示 `T` 是一个闭包类型

**三种 Fn 特征**

闭包捕获变量有三种途径，恰好对应函数参数的三种传入方式：转移所有权、可变借用、不可变借用，因此相应的 `Fn` 特征也有三种：

1. `FnOnce`，该类型的闭包会拿走被捕获变量的所有权。`Once` 顾名思义，说明该闭包只能运行一次，**仅**实现 `FnOnce` 特征的闭包在调用时会转移所有权，所以显然不能对已失去所有权的闭包变量进行二次调用：
    
    ```rust
    fn fn_once<F>(func: F)
    where
        F: FnOnce(usize) -> bool,
    {
        println!("{}", func(3));
        println!("{}", func(4));
    }
    
    fn main() {
        let x = vec![1, 2, 3];
        fn_once(|z|{z == x.len()})
    }
    
    error[E0382]: use of moved value: `func`
     --> src\main.rs:6:20
      |
    1 | fn fn_once<F>(func: F)
      |               ---- move occurs because `func` has type `F`, which does not implement the `Copy` trait
                      // 因为`func`的类型是没有实现`Copy`特性的 `F`，所以发生了所有权的转移
    ...
    5 |     println!("{}", func(3));
      |                    ------- `func` moved due to this call // 转移在这
    6 |     println!("{}", func(4));
      |                    ^^^^ value used here after move // 转移后再次用
      |
    
    ```
    
    这里面有一个很重要的提示，因为 `F` 没有实现 `Copy` 特征，所以会报错，那么我们添加一个约束，试试实现了 `Copy` 的闭包：`func` 的类型 `F` 实现了 `Copy` 特征，调用时使用的将是它的拷贝，所以并没有发生所有权的转移。
    
    ```rust
    fn fn_once<F>(func: F)
    where
        F: FnOnce(usize) -> bool + Copy,// 改动在这里
    {
        println!("{}", func(3));
        println!("{}", func(4));
    }
    
    fn main() {
        let x = vec![1, 2, 3];
        fn_once(|z|{z == x.len()})
    }
    true
    false
    ```
    
    如果你想强制闭包取得捕获变量的所有权，可以在参数列表前添加 `move` 关键字，这种用法通常用于闭包的生命周期大于捕获变量的生命周期时，例如将闭包返回或移入其他线程。
    
    ```rust
    use std::thread;
    let v = vec![1, 2, 3];
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });
    handle.join().unwrap();
    ```
    
2. `FnMut`，它以可变借用的方式捕获了环境中的值，因此可以修改该值。在闭包中，我们调用 `s.push_str` 去改变外部 `s` 的字符串值，因此这里捕获了它的可变借用，运行下试试：
注意：想要在闭包内部捕获可变借用，需要把该闭包声明为可变类型，也就是 `update_string` 要修改为 `mut update_string`
    
    ```rust
    fn main() {
        let mut s = String::new();
    
        let mut update_string =  |str| s.push_str(str);
        update_string("hello");
    
        println!("{:?}",s);
    }
    ```
    
    再看一个更复杂的：这段代码中`update_string`没有使用mut关键字修饰，而上文提到想要在闭包内部捕获可变借用，需要用关键词把该闭包声明为可变类型。
    
    这是因为：
    
    exec的参数f已经获得闭包的所有权了，操作是合法的。
    
    类似：let mut f = update_string;
    
    ```rust
    fn main() {
        let mut s = String::new();
    
        let update_string: impl FnMut(&str) =  |str| s.push_str(str);
    
        exec(update_string);
    
        println!("{:?}",s);
    }
    
    fn exec<'a, F: FnMut(&'a str)>(mut f: F)  {
        f("hello")
    }
    ```
    
    我们注意到，上面是将闭包的所有权转移给了`exec`函数，说明闭包没有实现`Copy`特征。这里要注意：闭包自动实现`Copy`特征的规则是，只要闭包捕获的类型都实现了`Copy`特征的话，这个闭包就会默认实现`Copy`特征。
    
3. `Fn` 特征，它以不可变借用的方式捕获环境中的值让我们把上面的代码中 `exec` 的 `F` 泛型参数类型修改为 `Fn(&'a str)`：
    
    ```rust
    fn main() {
        let s = "hello, ".to_string();
    
        let update_string =  |str| println!("{},{}",s,str);
    
        exec(update_string);
    
        println!("{:?}",s);
    }
    
    fn exec<'a, F: Fn(String) -> ()>(f: F)  {
        f("world".to_string())
    }
    ```
    
    在这里，因为无需改变 `s`，因此闭包中只对 `s` 进行了不可变借用，那么在 `exec` 中，将其标记为 `Fn` 特征就完全正确。
    

### move 和 Fn

在上面，我们讲到了 `move` 关键字对于 `FnOnce` 特征的重要性，但是实际上使用了 `move` 的闭包依然可以使用 `Fn` 或 `FnMut` 特征。

因为，**一个闭包实现了哪种 Fn 特征取决于该闭包如何使用被捕获的变量，而不是取决于闭包如何捕获它们**。`move` 本身强调的就是后者，闭包如何捕获变量：

```rust
fn main() {
    let s = String::new();

    let update_string =  move || println!("{}",s);

    exec(update_string);
}

fn exec<F: Fn()>(f: F)  {
    f()
}
```

### 三种 Fn 的关系

实际上，一个闭包并不仅仅实现某一种 `Fn` 特征，规则如下：

- 所有的闭包都自动实现了 `FnOnce` 特征，因此任何一个闭包都至少可以被调用一次
- 没有移出所捕获变量的所有权的闭包自动实现了 `FnMut` 特征
- 不需要对捕获变量进行改变的闭包自动实现了 `Fn` 特征

以下是三个特征的简化版源码：

```rust
pub trait Fn<Args> : FnMut<Args> {
    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
}

pub trait FnMut<Args> : FnOnce<Args> {
    extern "rust-call" fn call_mut(&mut self, args: Args) -> Self::Output;
}

pub trait FnOnce<Args> {
    type Output;

    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
}
```

从特征约束能看出来 `Fn` 的前提是实现 `FnMut`，`FnMut` 的前提是实现 `FnOnce`，因此要实现 `Fn` 就要同时实现 `FnMut` 和 `FnOnce`，这段源码从侧面印证了之前规则的正确性。

从源码中还能看出一点：`Fn` 获取 `&self`，`FnMut` 获取 `&mut self`，而 `FnOnce` 获取 `self`。 在实际项目中，**建议先使用 `Fn` 特征**，然后编译器会告诉你正误以及该如何选择。

闭包作为函数返回值：

用特征对象！只需要用 `Box` 的方式即可实现：

```rust
fn factory(x:i32) -> Box<dyn Fn(i32) -> i32> {
    let num = 5;

    if x > 1{
        Box::new(move |x| x + num)
    } else {
        Box::new(move |x| x - num)
    }
}
```

几个容易混淆的区别：

- `fn do1(c: String) {}`：表示实参会将所有权传递给 `c`
- `fn do2(c: &String) {}`：表示实参的不可变引用（指针）传递给 `c`，实参需带`&`声明
- `fn do3(c: &mut String) {}`：表示实参可变引用（指针）传递给 `c`，实参需带 `let mut` 声明，且传入需带 `&mut`
- `fn do4(mut c: String) {}`：表示实参会将所有权传递给 `c`，且在函数体内 `c` 是可读可写的，**实参无需 `mut` 声明**
- `fn do5(mut c: &mut String) {}`：表示实参可变引用指向的值传递给 `c`，且 `c` 在函数体内部是可读可写的，**实参需带 `let mut` 声明，且传入需带 `&mut`**

**一句话总结：在函数参数中，冒号左边的部分，如：`mut c`，这个 `mut` 是对🪄函数体内部有效🪄；冒号右边的部分，如：`&mut String`，这个 `&mut` 是针对🪄外部实参传入时的形式（声明）说明🪄。**

---

来看看例子：

```rust
fn main() {
    let d1 = "str".to_string();
    do1(d1);

    let d2 = "str".to_string();
    do2(&d2);

    let mut d3 = "str".to_string();
    do3(&mut d3);

    let d4 = "str".to_string();
    do4(d4);

    let mut d5 = "str".to_string();
    do5(&mut d5);
}

fn do1(c: String) {}

fn do2(c: &String) {}

fn do3(c: &mut String) {}

fn do4(mut c: String) {}

fn do5(mut c: &mut String) {}
```

# Iterator

## 惰性初始化

在 Rust 中，迭代器是惰性的，意味着如果你不使用它，那么它将不会发生任何事。

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("{}", val);
}
```

在 `for` 循环之前，我们只是简单的创建了一个迭代器 `v1_iter`，此时不会发生任何迭代行为，只有在 `for` 循环开始后，迭代器才会开始迭代其中的元素，最后打印出来。

这种惰性初始化的方式确保了创建迭代器不会有任何额外的性能损耗，其中的元素也不会被消耗，只有使用到该迭代器的时候，一切才开始。

## next方法

**迭代器之所以成为迭代器，就是因为实现了 `Iterator` 特征**，要实现该特征，最主要的就是实现其中的 `next` 方法，该方法控制如何从集合中取值，最终返回值的类型是关联类型 `Item`。

有两点需要注意：

- `next` 方法返回的是 `Option` 类型，当有值时返回 `Some(i32)`，无值时返回 `None`
- 遍历是按照迭代器中元素的排列顺序依次进行的，因此我们严格按照数组中元素的顺序取出了 `Some(1)`，`Some(2)`，`Some(3)`
- 手动迭代必须将迭代器声明为 `mut` 可变，因为调用 `next` 会改变迭代器其中的状态数据（当前遍历的位置等），而 `for` 循环去迭代则无需标注 `mut`，因为它会帮我们自动完成

总之，`next` 方法对**迭代器的遍历是消耗性的**，每次消耗它一个元素，最终迭代器中将没有任何元素，只能返回 `None`。

## IntoIterator 特征

其实有一个细节，由于 `Vec` 动态数组实现了 `IntoIterator` 特征，因此可以通过 `into_iter` 将其转换为迭代器，那如果本身就是一个迭代器，该怎么办？实际上，迭代器自身也实现了 `IntoIterator`，标准库早就帮我们考虑好了：

```rust
impl<I: Iterator> IntoIterator for I {
    type Item = I::Item;
    type IntoIter = I;

    #[inline]
    fn into_iter(self) -> I {
        self
    }
}
```

## into_iter, iter, iter_mut

在之前的代码中，我们统一使用了 `into_iter` 的方式将数组转化为迭代器，除此之外，还有 `iter` 和 `iter_mut`，聪明的读者应该大概能猜到这三者的区别：

- `into_iter` 会夺走所有权
- `iter` 是借用
- `iter_mut` 是可变借用

其实如果以后见多识广了，你会发现这种问题一眼就能看穿，`into_` 之类的，都是拿走所有权，`_mut` 之类的都是可变借用，剩下的就是不可变借用。

有两点需要注意的是：

- `.iter()` 方法实现的迭代器，调用 `next` 方法返回的类型是 `Some(&T)`
- `.iter_mut()` 方法实现的迭代器，调用 `next` 方法返回的类型是 `Some(&mut T)`

# **消费者与适配器**

消费者是迭代器上的方法，它会消费掉迭代器中的元素，然后返回其类型的值，这些消费者都有一个共同的特点：在它们的定义中，都依赖 `next` 方法来消费元素，因此这也是为什么迭代器要实现 `Iterator` 特征，而该特征必须要实现 `next` 方法的原因。

### 消费者适配器

只要迭代器上的某个方法 `A` 在其内部调用了 `next` 方法，那么 `A` 就被称为**消费性适配器**：因为 `next` 方法会消耗掉迭代器上的元素，所以方法 `A` 的调用也会消耗掉迭代器上的元素。

```rust
fn main() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);

    // v1_iter 是借用了 v1，因此 v1 可以照常使用
    println!("{:?}",v1);

    // 以下代码会报错，因为 `sum` 拿到了迭代器 `v1_iter` 的所有权
    // println!("{:?}",v1_iter);
}

fn sum<S>(self) -> S
    where
        Self: Sized,
        S: Sum<Self::Item>,
    {
        Sum::sum(self)
    }

```

### 迭代器适配器

既然消费者适配器是消费掉迭代器，然后返回一个值。那么迭代器适配器，顾名思义，会返回一个新的迭代器，这是实现链式方法调用的关键：`v.iter().map().filter()...`。

`map` 会对迭代器中的每一个值进行一系列操作，然后把该值转换成另外一个新值，该操作是通过闭包 `|x| x + 1` 来完成：最终迭代器中的每个值都增加了 `1`，从 `[1, 2, 3]` 变为 `[2, 3, 4]`。

与消费者适配器不同，迭代器适配器是惰性的，意味着你**需要一个消费者适配器来收尾，最终将迭代器转换成一个具体的值**：

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

### collect

上面代码中，使用了 `collect` 方法，该方法就是一个消费者适配器，使用它可以将一个迭代器中的元素收集到指定类型中，这里我们为 `v2` 标注了 `Vec<_>` 类型，就是为了告诉 `collect`：请把迭代器中的元素消费掉，然后把值收集成 `Vec<_>` 类型，至于为何使用 `_`，因为编译器会帮我们自动推导。

为何 `collect` 在消费时要指定类型？是因为该方法其实很强大，可以收集成多种不同的集合类型，`Vec<T>` 仅仅是其中之一，因此我们必须显式的告诉编译器我们想要收集成的集合类型。

### zig

`zip` 是一个迭代器适配器，它的作用就是将两个迭代器的内容压缩到一起，形成 `Iterator<Item=(ValueFromA, ValueFromB)>` 这样的新的迭代器,

在此处就是形如 `[(name1, age1), (name2, age2)]` 的迭代器。

然后再通过 `collect` 将新迭代器中`(K, V)` 形式的值收集成 `HashMap<K, V>`，同样的，这里必须显式声明类型，然后 `HashMap` 内部的 `KV` 类型可以交给编译器去推导，最终编译器会推导出 `HashMap<&str, i32>`，完全正确！

```rust
use std::collections::HashMap;
fn main() {
    let names = ["sunface", "sunfei"];
    let ages = [18, 18];
    let folks: HashMap<_, _> = names.into_iter().zip(ages.into_iter()).collect();

    println!("{:?}",folks);
}
```

### 闭包作为适配器参数

之前的 `map` 方法中，我们使用闭包来作为迭代器适配器的参数，它最大的好处不仅在于可以就地实现迭代器中元素的处理，还在于可以捕获环境值：

```rust

struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}
```

`filter` 是迭代器适配器，用于对迭代器中的每个值进行过滤。 它使用闭包作为参数，该闭包的参数 `s` 是来自迭代器中的值，然后使用 `s` 跟外部环境中的 `shoe_size` 进行比较，若相等，则在迭代器中保留 `s` 值，若不相等，则从迭代器中剔除 `s` 值，最终通过 `collect` 收集为 `Vec<Shoe>` 类型。