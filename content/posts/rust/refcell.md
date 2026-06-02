+++
title = "RefCell<T> 与内部可变性"
date = 2026-03-24
description = "解释 RefCell<T> 如何把借用检查从编译期推迟到运行期，以及 Rc<RefCell<T>> 的典型使用场景。"

[taxonomies]
tags = ["rust", "smart-pointers"]
+++
# RefCell<T> - 内部可变性

## 作用

`RefCell<T>` 提供**内部可变性**。

意思是：即使外部拿到的是不可变值，内部仍然可以修改。

它把借用检查从：
- **编译期** → **运行期**

## 基本例子

```rust
use std::cell::RefCell;

fn main() {
    let x = RefCell::new(5);

    *x.borrow_mut() += 1;

    println!("{}", x.borrow());
}
```

## 为什么叫"内部可变性"

通常 Rust 的规则是：
```rust
let x = 5;
// x = 6; // 不允许，x 是不可变的
```

但 `RefCell<T>` 允许你在外部不可变时，修改内部值：
```rust
let x = RefCell::new(5);
*x.borrow_mut() = 10; // 允许！
```

## 借用规则仍然存在

只是从编译器检查改成了运行时检查：

- 可以有多个 `borrow()`（不可变借用）
- 或者一个 `borrow_mut()`（可变借用）
- 不能同时混用

如果违反规则，会 **panic**：

```rust
use std::cell::RefCell;

fn main() {
    let x = RefCell::new(5);

    let a = x.borrow();      // 不可变借用
    let b = x.borrow_mut();  // 运行时 panic！
}
```

## borrow() vs borrow_mut()

- `borrow()`：返回 `Ref<T>`，允许多个同时存在
- `borrow_mut()`：返回 `RefMut<T>`，独占访问

这两个类型都实现了 `Deref`，所以可以直接像普通引用来使用。

## 什么时候用 RefCell

**适合场景：**
- 单线程环境
- 编译器无法静态判断借用合法，但你自己能保证逻辑正确
- 需要内部可变性
- 测试中需要 mock 对象状态

**常见应用场景：**
- 测试中的 mock 对象
- 树/图结构节点修改
- 搭配 `Rc<T>` 使用实现共享可变数据
- 缓存实现
- 计数器或状态跟踪

## Cell<T> vs RefCell<T>

`Cell<T>` 是另一种内部可变性容器：

```rust
use std::cell::Cell;

fn main() {
    let x = Cell::new(5);
    x.set(10);
    println!("{}", x.get());
}
```

**区别：**
- `Cell<T>`：适用于 `Copy` 类型，通过 `get()`/`set()` 操作，不返回引用
- `RefCell<T>`：适用于任何类型，返回引用，支持借用检查

## Rc<RefCell<T>> 组合

这是 Rust 里很经典的组合。

### 为什么组合使用

- `Rc<T>`：解决多个所有者问题
- `RefCell<T>`：解决内部可变性问题

合起来就是：
- 多个地方共享
- 且可以修改内部数据
- 但只限单线程

### 完整示例

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let root = Rc::new(Node {
        value: 1,
        children: RefCell::new(vec![]),
    });
    
    let child1 = Rc::new(Node {
        value: 2,
        children: RefCell::new(vec![]),
    });
    
    let child2 = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });
    
    // 修改 root 的子节点（虽然是 Rc，但内部可变）
    root.children.borrow_mut().push(child1.clone());
    root.children.borrow_mut().push(child2.clone());
    
    // 修改 child1 的子节点
    child1.children.borrow_mut().push(Rc::new(Node {
        value: 4,
        children: RefCell::new(vec![]),
    }));
    
    println!("{:#?}", root);
}
```

### 适用场景

很适合：
- 单线程树结构
- 图结构
- GUI 节点共享状态
- 编译器/解释器内部结构

你可以把它理解成：
**"单线程下的共享可变数据"**

## 注意事项

- **运行时开销**：借用检查在运行时进行，有轻微性能开销
- **panic 风险**：违反借用规则会导致程序 panic
- **单线程限制**：`RefCell<T>` 不是线程安全的
- **调试困难**：运行时错误比编译时错误更难调试

## 最佳实践

1. **优先使用编译时检查**：如果可能，尽量使用普通的可变引用
2. **明确文档说明**：使用 `RefCell` 时，在注释中说明为什么需要内部可变性
3. **避免深层嵌套**：不要过度使用 `RefCell`，保持代码清晰
4. **测试覆盖**：确保测试覆盖了所有可能的借用场景
