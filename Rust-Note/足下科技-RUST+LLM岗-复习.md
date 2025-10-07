# 足下科技-RUST+LLM岗-复习

## 对Rust所有权的理解

- 单一所有权
 单一所有权是指：每个值在任意时刻有且仅有一个所有者，所有权转移时原所有者失效。这样设计的好处是为了杜绝悬垂指针。
- 内存管理方式
    
    内存管理方式是：无Garbage Collection的自动回收。
    对于栈内存，是随着作用域的结束自动释放；对于堆内存，当所有者离开作用域是，自动调用Drop trait释放内存。
    这样的内存管理方式通过与所有权规则相配合，可以保证编译时的内存安全，兼顾性能与安全。
    
- Copy/Move语义
    
    Move语义是转移所有权；
    Copy语义是复制栈上的数据，不转移所有权；
    
    <aside>
    💡
    
    copy trait vs clone trait
    
    Copy trait：
    
    具备这种trait的类型，一般都是一下简单的基本类型，在进行赋值和函数参数传递时，会直接复制数据，而不会发生所有权转移，这一过程是隐式发生的。
    
    并且copy trait可以通过`#[derive(Copy)]` 宏来简化实现。一般来说，基本类型都自动实现了copy trait，并且当类型的所有成员都实现了Copy trait时，该类型就能自动派生Copy trait。`但是注意`：Copy trait的实现必须是显式实现的，也就是即使类型的所有成员内部都实现了Copy trait，也必须在类型上方调用`#[derive(Copy)]`来显式的生命Copy trait的实现，但是调用时隐式发生的，这点需要注意。
    
    Clone trait：
    
    具备这种trait的类型，需要通过显式的通过`clone()`方法来复制数据。通常意味着复制操作会有较大的开销。
    
    首先，Clone trait也可以通过`#[derive(Clone)]`宏来简化实现，调用的前提是所有类型都实现了Clone trait。
    
    Clone分为深拷贝和浅拷贝。深拷贝是指：当调用clone方法时，会复制所有的数据，包括堆上分配的数据。浅拷贝是指：只复制指针，而不复制指针指向的数据，新对象和原对象会公用一块内存。
    
    这里注意并非所有的智能指针都是浅拷贝：
    
    Rc、Arc是浅拷贝，因为他们用于多个所有者之间的共享数据
    
    Box是深拷贝，它会将堆上分配的内存一起进行拷贝
    
    RefCell和Mutex则取决于内部的类型，如果内部类型T是深拷贝，那RefCell和Mutex就都是深拷贝，否则就是浅拷贝。
    
    并且：实现Copy trait的前提是先实现Clone trait
    
    </aside>
    
- 借用规则
    - 同一个作用域可以存在多个不可变借用
    - 同一个作用域最多可以存在一个可变借用，并且不能与不可变借用共存
    - 借用的生命周期必须小于等于所有者的生命周期
- 如何把借用检查推迟到运行期
    
    可以通过RefCell将编译器的借用检查推迟到运行时。
    
    <aside>
    💡
    
    RefCell<T>：
    
    创建：`let value = RefCell::new(初始值);`
    
    获取不可变借用：`let ref_value = value.borrow();`在ref_value存在期间，无法获得value的可变借用
    
    获取可变借用：`let mut ref_mut_value = value.borrow_mut();`在ref_mut_value存在期间，无法获得可变与不可变借用。
    
    与Rc,Arv结合使用，实现多个所有者对同一个RefCell实例的共享。
    
    </aside>
    
    ## 实现vec![]宏，要求实验my_vec![1,2]和my_vec![1;2]两种形式。
    
    ```rust
    #[macro_export]
    macro_rules! myvec {
        ($($x:expr),*) => {
            {
                let mut my_vec = Vec::new();
                $(my_vec.push($x);)*
                my_vec
            }
        };
        ($value:expr; $n:expr) => {
            {
                let mut my_vec = Vec::new();
                for _ in 0..$n {
                    my_vec.push($value.clone());
                }
                my_vec
            }
        };
    }
    ```
    

## 实现链表中相邻两节点间插入一个节点，插入节点的val是两个节点val的最大公约数。

力扣mid题-2807

Given the head of a linked list `head`, in which each node contains an integer value.

Between every pair of adjacent nodes, insert a new node with a value equal to the **greatest common divisor** of them.

Return *the linked list after insertion*.

The **greatest common divisor** of two numbers is the largest positive integer that evenly divides both numbers.

```
Input: head = [18,6,10,3]
Output: [18,6,6,2,10,1,3]
Explanation: The 1st diagram denotes the initial linked list and the 2nd diagram denotes the linked list after inserting the new nodes (nodes in blue are the inserted nodes).
- We insert the greatest common divisor of 18 and 6 = 6 between the 1st and the 2nd nodes.
- We insert the greatest common divisor of 6 and 10 = 2 between the 2nd and the 3rd nodes.
- We insert the greatest common divisor of 10 and 3 = 1 between the 3rd and the 4th nodes.
There are no more adjacent nodes, so we return the linked list.
```

```rust
// Definition for singly-linked list.
// #[derive(PartialEq, Eq, Clone, Debug)]
// pub struct ListNode {
//   pub val: i32,
//   pub next: Option<Box<ListNode>>
// }

// impl ListNode {
//   #[inline]
//   fn new(val: i32) -> Self {
//     ListNode {
//       next: None,
//       val
//     }
//   }
// }
fn gcd(a: i32, b: i32) -> i32 {
    let mut temp;
    let mut a = a;
    let mut b: i32 = b;
    while b != 0 {
        temp = b;
        b = a % b;
        a = temp;
    }
    a
}

pub fn insert_greatest_common_divisors(mut head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    let mut current = &mut head;
    while current.as_ref().unwrap().next.is_some(){
        let x = current.as_mut().unwrap();
        let next = x.next.take();
        x.next = Some(Box::new(ListNode {
            val: gcd(x.val, next.as_ref().unwrap().val),
            next,
        }));
        current = &mut current.as_mut().unwrap().next.as_mut().unwrap().next;
    }
    head
}
```

<aside>
💡

as_ref()：获取不可变引用

as_mut()：获取可变引用

不能从不可变引用中获取可变引用

Option的take方法：获取Option包裹的内容，并将原来的Option中的内容变为None

</aside>

## 利用Vec实现栈，要求实现top、pop、push、iter、into_iter

```rust
#[derive(Debug)]
struct Stack<T> {
    size: usize,
    data: Vec<T>,
}

impl<T> Stack<T> {
    fn new() -> Self {
        Self {
            size: 0,
            data: Vec::new(),
        }
    }
    fn is_empty(&self) -> bool {
        0 == self.size
    }
  
    fn push(&mut self, val: T) {
        self.data.push(val);
        self.size += 1;
    }

    fn pop(&mut self) -> Option<T> {
        if self.is_empty() {
            return None;
        }
        self.size -= 1;
        self.data.pop()
    }

  
    fn top(&self) -> Option<&T>  {
        self.data.last()
    }

    fn into_iter(self) -> IntoIter<T> {
        IntoIter(self)
    }

    fn iter(&self) -> Iter<T> {
        let mut iterator = Iter {
            stack: Vec::new()
        };
        for item in self.data.iter() {
            iterator.stack.push(item);
        }
        iterator
    }
}

struct IntoIter<T>(Stack<T>);
impl<T: Clone> Iterator for IntoIter<T> {
    type Item = T;
    fn next(&mut self) -> Option<Self::Item> {
        if !self.0.is_empty() {
            self.0.size -= 1;self.0.data.pop()
        }
        else {
            None
        }
    }
}

struct Iter<'a, T: 'a> {
    stack: Vec<&'a T>,
}

impl<'a, T> Iterator for Iter<'a, T> {
    type Item = &'a T;
    fn next(&mut self) -> Option<Self::Item> {
        self.stack.pop()
    }
}
```

<aside>
💡

pop：弹出的是一个Option包裹的数据

top：因为并不会弹出数据，所以返回的是一个Option包裹的引用

Iter中的stack字段是一个包含引用的Vec，Vec的iter方法返回的是其内部值的引用，而不是数据。并且Vec的iter返回的顺序是栈底到栈顶，也就是从索引0开始。

</aside>

| **方法** | **作用描述** | **迭代器类型** | **元素类型** |
| --- | --- | --- | --- |
| `.iter()` | 借用集合元素的不可变引用（`&T`），不消耗集合。 | `Iter<'a, T>` | `&T` |
| `.iter_mut()` | 借用集合元素的可变引用（`&mut T`），允许修改元素。 | `IterMut<'a, T>` | `&mut T` |
| `.into_iter()` | 转移集合所有权，消耗集合并生成拥有元素值的迭代器（`T`）。 | `IntoIter<T>` | `T` |

## 快排的非递归实现

递归的可视化流程可以看这个视频[三分钟学会快速排序_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1j841197rQ/?spm_id_from=333.337.search-card.all.click&vd_source=6988d2e0512788e824dd750150bd2b5d)

### 快排递归实现

```rust
fn partition(arr: &mut [i32], low: usize, high: usize) -> usize {
    let pivot = low;

    let (mut left, mut right) = (low, high);
    while left < right {
        while left < right && arr[right] >= arr[pivot]  {
            right -= 1;
        }
        while left < right && arr[left] <= arr[pivot] {
            left += 1;
        }

        if left != right {
            arr.swap(left, right);
        }
    }

    arr.swap(pivot, left);

    left
}

fn quick_sort(arr: &mut [i32]) {
    if 1 >= arr.len() {
        return ;
    }
    
    let pivot = partition(arr, 0, arr.len() - 1);

    let (left, right) = arr.split_at_mut(pivot);

    quick_sort(left);
    quick_sort(&mut right[1..]);
}
```

### 快排的非递归实现

```rust
fn partition(arr: &mut [i32], low: usize, high: usize) -> usize {

        let pivot = low;
        let mut left = low;
        let mut right: usize = high;
        while left < right {
            while left < right && arr[right] >= arr[pivot] {
                right -= 1;
            }

            while left < right && arr[left] <= arr[pivot] {
                left += 1;
            }
            if left != right {
                arr.swap(left, right);
             }
        }
         arr.swap(pivot, left);
         left
    }
    
fn quicksort_non_recursive(arr: &mut [i32]) {
    if arr.is_empty() {
        return;
    }
    let mut stack = Vec::with_capacity(arr.len());
    stack.push((0, arr.len() - 1));
    while let Some((left, right)) = stack.pop() {
        let pivot = partition(arr, left, right);
                if pivot > left {
                    stack.push((left, pivot - 1));
                }
                if pivot < right {
                    stack.push((pivot + 1, right));
                }
            }
}
```

<aside>
💡

使用while let 和栈替换了递归实现

</aside>

## 链表实现get、add、remove

```rust
use std::fmt::Debug;
use std::ptr::NonNull;

#[derive(Debug)]
struct Node<T> {
    val: T,
    next: Option<NonNull<Node<T>>>,
    prev: Option<NonNull<Node<T>>>,
}

impl<T> Node<T> {
    fn new(value: T) -> Box<Node<T>> {
        Box::new(Node { val: value, next: None, prev: None })
    }
}

#[derive(Debug)]
struct LinkedList<T> {
    length: u32,
    start: Option<NonNull<Node<T>>>,
    end: Option<NonNull<Node<T>>>,
}

impl<T: std::cmp::PartialEq> LinkedList<T> {
    fn new() -> Self {
        Self { length: 0, start: None, end: None }
    }

    fn get_ith_node(&self, node: Option<NonNull<Node<T>>>, index: i32) -> Option<&T> {
        match node {
            None => None,
            Some(next_node_ptr) => {
                unsafe {
                    match index {
                        0 => Some(&(*next_node_ptr.as_ptr()).val),
                        _ => self.get_ith_node((*next_node_ptr.as_ptr()).next, index - 1),
                    }
                }
            }
        }
    }

    fn get(&self, index: i32) -> Option<&T> {
        self.get_ith_node(self.start, index)
    }

    fn add(&mut self, new_value: T) {
        let new_node = Node::new(new_value);
        let new_node_ptr = NonNull::new(Box::into_raw(new_node)).unwrap();

        match self.end {
            None => {
                self.start = Some(new_node_ptr);
                self.end = Some(new_node_ptr);
            }

            Some(end_ptr) => {
                unsafe {
                    (*end_ptr.as_ptr()).next = Some(new_node_ptr);
                    (*new_node_ptr.as_ptr()).prev = Some(end_ptr);
                }
                self.end = Some(new_node_ptr);
            }
        }
        self.length += 1;
    }

    fn remove(&mut self, value: T) -> bool {
        let mut current = self.start;
        while let Some(current_ptr) = current {
            unsafe {
                if (*current_ptr.as_ptr()).val == value {
                    let prev = (*current_ptr.as_ptr()).prev;
                    let next = (*current_ptr.as_ptr()).next;
                    if let Some(prev_ptr) = prev {
                        (*prev_ptr.as_ptr()).next = next;
                    } else {
                        self.start = next;
                    }
                    if let Some(next_ptr) = next {
                        (*next_ptr.as_ptr()).prev = prev;
                    }else {
                        self.end = prev;
                    }
                    let _ = Box::from_raw(current_ptr.as_ptr());
                    self.length -= 1;
                    return true;
                }
                current = (*current_ptr.as_ptr()).next;
            }
        }
        false
    }
}
```

<aside>
📢

 `let _ = Box::from_raw(current_ptr.as_ptr());`这句代码充分利用了rust的内存管理机制。

</aside>