[← Rust](Rust.md)

## 泛型

`<T>`中的T是**类型参数**。在**编译时**会被替换为具体的类型，这个过程称为**单态化（monomorphyzation)**。

在函数名、结构体名或枚举名后面加个`<T>`就表示了泛型。

使用泛型时使用`StructName::<TypeName>`来指明T的具体类型TypeName，也可以自动推断。

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
	//自动推断
    let integer = Point { x: 5, y: 10 };
    //手动指定
    let float = Point::<f32> { x: 1.0, y: 4.0 };
    println!("integer: ({}, {}), float: ({}, {})", integer.x, integer.y, float.x, float.y);
}
```

如果要写泛型结构体的impl，则要写成`impl<T> Point<T>{...}`。但针对具体类型的实现就不需要这样写了，如`impl Point<i32>{...}`。（要尽量早地“注册”所有符号名）

泛型T的约束加在impl的后面。

```rust
//这个例子中，ReportCard::<f32>不需要指明具体类型，可自动推断，即不用写::<f32>
pub struct ReportCard<T> {
    pub grade: T,
    pub student_name: String,
    pub student_age: u8,
}

impl<T: std::fmt::Display> ReportCard<T> {
    pub fn print(&self) -> String {
        format!("{} ({}) - achieved a grade of {}",
            &self.student_name, &self.student_age, &self.grade)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
  
    #[test]
    fn generate_numeric_report_card() {
        let report_card = ReportCard::<f32> {
            grade: 2.1,
            student_name: "Tom Wriggle".to_string(),
            student_age: 12,
        };
        assert_eq!(
            report_card.print(),
            "Tom Wriggle (12) - achieved a grade of 2.1"
        );
    }
  
    #[test]
    fn generate_alphabetic_report_card() {
        let report_card = ReportCard::<String> {
            grade: "A+".to_string(),
            student_name: "Gary Plotter".to_string(),
            student_age: 11,
        };
        assert_eq!(
            report_card.print(),
            "Gary Plotter (11) - achieved a grade of A+"
        );
    }
}
```

### const泛型

泛型前面加上const之后，泛型就可以像常数一样被定义类型和值，这样就可以用来限制数组的尺寸。

[泛型 Generics - Rust语言圣经(Rust Course)](https://course.rs/basic/trait/generic.html#const-%E6%B3%9B%E5%9E%8Brust-151-%E7%89%88%E6%9C%AC%E5%BC%95%E5%85%A5%E7%9A%84%E9%87%8D%E8%A6%81%E7%89%B9%E6%80%A7)

### std::borrow::Borrow 与&String

一个函数的参数为`&T`，而T有可能为`String`，此时只会解析成`&String`，而不能使用`&str`。这种接口通用性很差。

```rust
pub fn get<K>(&self, k: &K) -> Option<&V>
    where
        K: OtherTrait,
    {
        // ...
    }
```

使用[Borrow trait](#Borrow%20%26%20AsRef)可以解决问题。

```rust
pub fn get<Q>(&self, k: &Q) -> Option<&V>
    where
        K: Borrow<Q>,
        Q: ?Sized + OtherTrait
    {
        // ...
    }
```

`K: Borrow<Q>`使得对K的借用等价为对Q的借用。如果上下文推导出`K`为`String`，则`Q`就可以是`str`，从而得到`&str`类型的函数参数。

这也要求目标`Q`要满足`OtherTrait`。

副作用是会传参时使用`into()`不能很好地自动推断。
## Trait

trait告诉编译器某种类型有哪些可以与其他类型共享的功能。抽象的定义共享行为。

**trait bound（约束）**: 让泛型的类型参数指定为一个trait，即可规定该类型参数的行为，也就能让以该类型参数为类型的变量能做出一些事情，如比大小。

trait和interface差不多，规定了共用的一组方法签名，但可以没有实现；如果有实现，就相当于是默认实现（java的default）；实现trait的类型必须实现所有未实现的trait方法。

默认实现的方法可以调用其他无默认实现的方法。

```rust
trait Speak {
    fn speak(&self);
}

struct Human;

impl Speak for Human {
    fn speak(&self) {
        println!("Hello, world!");
    }
}

fn main() {
    let person = Human;
    person.speak();
}
```

如果一个方法来自于trait，则使用的时候必须保证当前作用域里面有此trait。

参数的mut与否似乎并不是严格限制的：
```rust
trait AppendBar {
    fn append_bar(self) -> Self;
}
  
impl AppendBar for String {
    fn append_bar(mut self) -> Self{
        self.push_str("Bar");
        self
    }
}
```

可以在某个type上实现某个trait的前提条件是：这个type **或** 这个trait 是**本crate**里定义的。也就是说，无法为外部的类型定义外部的trait。这样可以保证不会出现两个crate分别给同一个struct实现同一个trait的情况。即**孤儿规则（Orphan Rule）**（[孤儿规则](#%E5%AD%A4%E5%84%BF%E8%A7%84%E5%88%99)），防止trait的实现是个孤儿（既不属于定义trait的crate，也不属于定义type的crate）。但也因此可能无法做到为某些外部结构体实现Debug、Display，再套一层结构体即可解决：

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

### trait作为函数参数

trait可以作为参数，当成接口类型使用：

```rust
pub trait Animal {  
    fn speak(&self);  
}  
  
pub struct Dog;  
pub struct Cat;  
  
impl Animal for Dog {  
    fn speak(&self) {  
        println!("Woof!");  
    }  
}  
  
impl Animal for Cat {  
    fn speak(&self) {  
        println!("Meow!");  
    }  
}  

//impl trait 写法 （是trait bound 写法的语法糖）
fn make_animal_speak1(animal: &impl Animal) {  
    animal.speak();  
}  

//dyn trait 写法（动态分发）
fn make_animal_speak1(animal: &impl Animal) {  
    animal.speak();  
}  

//trait bound 写法
fn make_animal_speak2<T: Animal>(animal: &T) {
    animal.speak();
}
  
fn main() {  
    let dog = Dog;  
    let cat = Cat;  
  
    make_animal_speak1(&dog);  
    make_animal_speak2(&cat);  
}
```

也可以使用`+`规定参数需要同时实现多个trait：
```rust
fn action<T: Speak + Move>(animal: T) {
    animal.speak();
    animal.move();
}
```

可以用`where`子句来指定类型参数的约束，以保证函数名到函数参数列表之间不会写太长的东西：
```rust
fn some_function<T, U>(t: T, u: U) -> i32
where
    T: std::fmt::Display + Put,
    U: std::fmt::Debug,
{
    println!("T: {}", t);
    println!("U: {:?}", u);
    42
}
```

### trait作为函数返回值

可以直接`-> impl Speak`，但是函数内的具体的返回值不能为多个类型，即使它们都实现了Speak这个trait。可以理解为编译器会根据具体的返回值去推断返回值类型，推断成功后才又和规定的impl xxx进行对比以确保那唯一的返回值类型实现了该xxx trait。

### 使用trait bound对泛型类型进行有条件的实现

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

### 覆盖实现

覆盖实现（blanket implementation）: 在impl后面对类型参数使用trait bound进行约束，再把for后面的type改成类型参数T。这样的话，满足trait bound约束的type都会被这个impl覆盖实现。

```rust
impl<T: Display> ToString for T {
    // ...
}
```

### 关联类型

在trait定义一个type，具体使用时将其赋值后即可关联到某个类型，如：

```rust
trait Iterator {
    type Item; // 关联类型，表示迭代器的元素类型

    fn next(&mut self) -> Option<Self::Item>; // next方法返回关联类型的Option值
}
```

使用关联类型而不使用泛型的话无法给一个类型多次实现一个trait。

### 运算符重载

重载`std::ops`里的trait就等价于运算符重载。

```rust
//重载了Meters对Millimeters的+运算符
impl Add<Meters> for Millimeters {
	type Output = Millimeters;
	
	fn add(self, other: Meters) -> Millimeters {
		Millimeters(self.0 + other.0 * 1000)
	}
}
```

Map的Key若想要取消去重机制，只要在Ord实现里面不出现Order::Equal即可：
```rust
extern crate alloc;  
  
use core::cmp::Ordering;  
use core::ops::AddAssign;  
use alloc::collections::BTreeMap;  
  
pub static HALF_BIG_STRIDE: u8 = 127;  
  
#[derive(Clone, Copy, Debug)]  
pub struct Stride(pub u8);  
  
impl PartialOrd for Stride {  
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {  
        if self.0 >= other.0 && self.0 - other.0 <= HALF_BIG_STRIDE  
            || self.0 < other.0 && other.0 - self.0 > HALF_BIG_STRIDE{  
            Some(Ordering::Less)  
        }else{  
            Some(Ordering::Greater)  
        }  
    }  
}  
  
impl PartialEq for Stride {  
    fn eq(&self, other: &Self) -> bool {  
        let _ = other;  
        false    }  
}  
  
impl Eq for Stride{}  
  
impl Ord for Stride{  
    fn cmp(&self, other: &Self) -> Ordering {  
        self.partial_cmp(&other).unwrap()  
    }  
}  
  
impl AddAssign<u8> for Stride{  
    fn add_assign(&mut self, other: u8) {  
        self.0 += other;  
    }  
}  
  
fn main() {  
    let mut map = BTreeMap::new() ;  
    map.insert(Stride(1),1);  
    map.insert(Stride(1),1);  
    map.insert(Stride(2),2);  
    map.insert(Stride(2),2);  
    map.insert(Stride(3),2);  
    map.insert(Stride(2),6);  
    println!("{:?}", map);  
}
```


### 完全限定语法

如果结构体Human实现了Pilot trait，也实现了Wizard trait，且Human自己、两个trait内都实现了不同的fly方法，则：

```rust
let person = Human;
person.fly(); //调用原来的
Pilot::fly(&person); //调用Pilot trait的实现
Wizard::fly(&person); //调用Wizard trait的实现
```

这种情况下，比如对Pilot这个trait来说，由于接收到了&person为&Human，则可以推断出是调用Human对Pilot的实现中的fly。

但是也有情况是推断不出来的，例如要调用的不是方法而是函数，而且参数很朴素：

```rust
Dog::baby_name(); //调用原来的
Animal::baby_name(); //Error: 无法确认到底调用哪个函数
<Dog as Animal> :: baby_name(); //调用Animal trait的实现
```

`<TypeName as TraitName>::func_name()`就叫做完全限定语法。

### supertrait

即trait的继承，`trait TraitName: SuperTraitName{...}`即可使用SuperTraitName的所有特性。

### 关联类型

一个trait里面可以定义一个“成员类型”。当一个结构体实现该trait时，就必须指定这个类型是什么。

### 泛型trait

带泛型参数的trait。谨慎使用，因为不同泛型的trait是完全不同的trait，很多时候编译器无法知道到底使用哪个trait。

### Object Safe

现在官方称 **dyn compatibility**；旧称 object safety。兼容的 trait 可以作 `&dyn TraitName` 或 `Box<dyn TraitName>`，由虚表调用可分派的方法。

记忆：虚表入口的签名不能依赖调用时未知的具体 `Self`。可分派方法不能有类型泛型参数、额外的 `Self` 参数/返回值或 `impl Trait` 返回值；生命周期泛型可以。还须满足接收者等条件。

无接收者的关联函数不能通过虚表调用。不兼容的方法可用 `where Self: Sized` 排除出动态分派，trait 的其余部分仍可能兼容；若 trait 本身要求 `Self: Sized`，就不能创建该 trait object。

`dyn Trait` 自身是非 Sized 类型；上面的 `Sized` 约束是区分“只能由具体类型调用”和“可动态分派”的办法。

[完整条件：Rust Reference](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility)（核验：2026-09-30）。原参考：[trait object - 知乎](https://zhuanlan.zhihu.com/p/23791817)。

### impl trait in trait（RPITIT）

Rust1.75后，支持在trait里面使用impl trait：
```rust
trait MyTrait {
    fn method(&self) -> impl Debug;
}
```

概念上可想成下面的匿名关联类型形式（伪代码，并非可直接编译的声明）：
```rust
trait MyTrait {
    type method<‘a>: Debug; // 一个泛型关联类型（GAT）
    fn method(&self) -> Self::method<‘_>;
}
```

返回 `impl Trait` 的方法不能直接经 trait object 分派；若用 `where Self: Sized` 将该方法排除，trait 其余部分仍可能 dyn compatible。需要动态调用时，可考虑返回合适的 trait object，是否堆分配取决于接口。

## 具体trait
### Deref Trait

唯一一种隐式转化逻辑。

deref trait里面有`fn deref(&self) -> &T`。

- 解引用符号`*y`作用于实现了deref trait的类型的时候，会自动变成`*(y.deref())`，即**隐式解引用转化（Deref Coercion）**。
- 在函数传递的参数类型不匹配的时候，编译器会试图调用deref来进行转换。如`&String`就有可能会转换成`&str`。

- 当`T:Deref<Target=U>`时，允许`&T`或`&mut T`转换为`&U`。
- 当`T:DerefMut<Target=U>`时，允许`&mut T`转换为`&mut U`。

### Drop Trait

drop trait里面有`fn drop(&mut self)`。会在变量离开作用域时自动调用，被当做析构函数，可用于释放资源。

不能直接地显式调用，但可以调用`std::mem::drop(value)`，以提前释放变量（调用drop）。

Drop Check机制对实现了Drop Trait的类型的签名做出了约束，从而防止drop函数体实现出现非法访问：[Drop Check - Drop in std::ops - Rust](https://doc.rust-lang.org/std/ops/trait.Drop.html#drop-check)。

### Eq & PartialEq

生成相等的相关逻辑。

PartialEq不实现反身性，即没有`a == a`。

浮点类型只实现了PartialEq，因为`NaN!=NaN`。

### Ord & PartialOrd

生成比大小（Order）的相关逻辑。

PartialOrd不实现确定性（必定存在`>`或\=\=或`<`其中的一个关系）。它允许存在无法比较的两数，返回None。

PartialOrd继承了PartialEq。PartialOrd完成后提供`lt()`，`le()`，`gt()`，`ge()`。

Ord继承了Eq和PartialOrd。Ord完成后提供`max()`，`min()`，`clamp()`。

### Send & Sync

- 实现`Send`的类型可以在线程间安全的传递其所有权
- 实现`Sync`的类型可以在线程间安全的共享(通过引用)

- 若`&T`是`Send`，则要求`T`是`Sync`。
- 若`&mut T`是`Send`，则要求`T`是`Send`。（因为mut引用每个瞬间只能被一个线程持有，不存在并发问题）

[基于 Send 和 Sync 的线程安全 - Rust语言圣经(Rust Course)](https://course.rs/advance/concurrency-with-threads/send-sync.html)

`T: !Sync` 表示不能直接跨线程共享 `&T`；若它是 Send，仍可以转移所有权，或放进合适的同步容器。不能据此说 Arc 都可换成 Rc：`Arc<Mutex<T>>` 就可在 `T: Send` 时用于线程间共享。

包含 `&T` 的结构体要自动实现 Send，需要 `T: Sync`；包含 `&mut T` 时则需要 `T: Send`。共享借用与独占借用的条件不同。

这两个trait都是自动实现的，如果手动实现的话，相当于一个简单的声明，并且是unsafe的。换句话说，如果一个结构体实现了Send，那么就理应可以任意转移；如果实现了Sync，就理应可以任意共享访问。

锁的条件：`Mutex<T>` 的 Send 与 Sync 都只要求 `T: Send`；`RwLock<T>: Send` 要求 `T: Send`，而 `RwLock<T>: Sync` 要求 `T: Send + Sync`。读写锁允许同时取得多个共享借用；内部可变性意味着“共享读”也可能写内存。[标准库 RwLock](https://doc.rust-lang.org/std/sync/struct.RwLock.html)。

### Borrow & AsRef

借用类型`Q`时，可以被视为借用类型`T`，如`String`借用变为`str`借用、`Box<T>`借用变为`T`借用。这需要使用`Borrow` trait来实现。`Q impl Borrow<T>`。[Borrow in std::borrow - Rust](https://doc.rust-lang.org/std/borrow/trait.Borrow.html)

[编写参数为泛型且有可能为&str时非常有用](#std%3A%3Aborrow%3A%3ABorrow%20%E4%B8%8E%26String)。

`AsRef` trait与此功能十分相似，但也有区别。[AsRef in std::convert - Rust](https://doc.rust-lang.org/std/convert/trait.AsRef.html)

## 孤儿规则


目的：
- 防止两个库给同一个库进行拓展实现，导致这两个库之间冲突。
- 防止下游实现破坏上游功能。

[2451-re-rebalancing-coherence - The Rust RFC Book](https://rust-lang.github.io/rfcs/2451-re-rebalancing-coherence.html)

被拓展的孤儿规则允许为外部类型实现外部trait，但要求本地trait出现在泛型参数中（且顺序靠前）。这使得可以为本地类型实现`Into`或`From`外部类型。

## 动态分发与静态分发

- 动态分发: dyn trait，运行时调用对应实现的方法。使用胖指针（两倍指针大小）来描述传入的指针对应的类型，这样调用方法时就能正确找到函数位置。**胖指针是虚表的替代品**。
- 静态分发: impl trait，会在编译期单态化，因此并不能同时适配多种类型，如作为函数返回值类型的之后返回两种类型。

智能指针`Box<TraitName>`逻辑上没问题，但已被弃用，应当使用`Box<dyn TraitName>`。

