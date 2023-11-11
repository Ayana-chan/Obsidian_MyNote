
# 链接

[Rust语言圣经(Rust Course)](https://course.rs/about-book.html)

[命名规范 - Rust语言圣经(Rust Course)](https://course.rs/practice/naming.html)
# 安装与起步

[Rust使用国内Crates 源、 rustup源 |字节跳动新的 Rust 镜像源以及安装rust\_rust镜像-CSDN博客](https://blog.csdn.net/inthat/article/details/106742193)

[Rust安装配置\_rust下载到d盘\_Andreby的博客-CSDN博客](https://blog.csdn.net/andrewby/article/details/75139736)

Clion中，安装Rust插件，进行如下配置：

- Toolchain Location: `...\.rustup\toolchains\stable-x86_64-pc-windows-gnu\bin`
- Standard Library: `...\.rustup\toolchains\stable-x86_64-pc-windows-gnu\lib\rustlib\src\rust`

>这里不知道什么时候可以自动填入了，且toolchain路径为`...\.cargo\bin`。可能是因为我下了且在用nightly版本。

如果Standard Library找不到的话，运行下方命令即可：
```bash
rustup component add rust-src --toolchain stable-x86_64-pc-windows-gnu
```

`rustup doc`可以离线查看教程。

换源，`...\.cargo\config`：

```config
[source.crates-io]
# 替换成你偏好的镜像源
replace-with = 'tuna'
 
# 清华大学 5mb
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
 
# 中国科学技术大学 2mb
[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"
# 上海交通大学 2mb
[source.sjtu]
registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index"
 
# rustcc社区 2mb
[source.rustcc]
registry = "https://crates.rustcc.cn/crates.io-index"
# 字节跳动 10mb
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
```

# 基础

Rust是静态强类型的。

代码包或库被称为crate。

变量默认为immutable，即`不可变变量`，不可进行二次赋值；想要让它可变需要在声明时加上`mut`。

- 不可变变量可以声明赋值分开，即赋值是不确定的，仅仅是无法二次赋值；
- 而const常量必须进行一次初始化，且必须指定类型。

use很像cpp的using namespace。有的crate是默认导入（prelude，预导入模块）的，如std::io。

```rust
use std::io;  
  
fn main() {  
    println!("Input: ");  
    let mut input=String::new(); //声明并用String的new赋值一个可变变量
    io::stdin().read_line(&mut input).expect("Fail to read line");  
    //read_line返回 io::Result，交由expect处理
    println!("Your input is: {}",input)  
}
```

trait: 可视为接口，提供了一些方法，如`rand::Rng`。

不应当使用`while true{...}`来表示无限循环，而是使用`loop{...}`，因为rust在哲学上追求高度一致，while仅用于while condition。

数字可以使用下划线分割来增加可读性，如100_000。

变量名、函数名使用snack_case命名规范，其他都是大驼峰。

有些东西在预导入模块（prelude）里，不需要use。

T代表所有类型，包括&T和&mut T；而&T和&mut T完全没有交集。

## shadowing

rust允许定义同名的新变量来shadow（隐藏）旧变量，常用于类型转换。新旧的变量的类型可以完全不同，也可以都是不可变变量。

## 数据类型

`let valName: type = value`可以显式定义变量类型。

类型推断可以做到从下往上推断。而所有变量应当在编译期就能推断出确定的类型。

### 标量类型

#### 整数


| Length  | Signed | Unsigned |
| ------- | ------ | -------- |
| 8-bit   | i8     | u8       |
| 16-bit  | i16    | u16      |
| 32-bit  | i32    | u32      |
| 64-bit  | i64    | u64      |
| 128-bit | i128   | u128     |
| arch    | isize  | usize    | 

isize和usize由计算机架构决定，64位计算机就是64bit。主要用于对某种集合进行索引操作。

整数默认类型是i32。

字面值：

| Number Literals | Example     |
| --------------- | ----------- |
| Decimal         | 98_222      |
| Hex             | 0xff        |
| Octal           | 0077        |
| Binary          | 0b1111_0000 |
| Byte(u8 only)   | b'A'            |

除Byte以外都能加类型后缀，如57u8。

在调试模式下发生溢出则会panic；发布模式下256u8会变成0,257u8变成11。

#### 浮点

IEEE-754

- f32，32位，单精度
- f64，64位，双精度（默认）
#### 布尔

bool

占1个字节。

#### 字符

char

占4个字节。使用unicorn编码。


### 复合类型

Rust提供了两个基本复合类型：元组Tuple和数组。

#### 元组

使用点-索引进行取值。

```rust
let tp: (i32, i64, f64) = (45, 32, 6.7);  
println!("tp: {},{},{}", tp.0, tp.1, tp.2); 
```

可以使用解构赋值来拆分：

```rust
let tp: (i32, i64, f64) = (45, 32, 6.7);  
let (t0, t1, t2) = tp;  
println!("tp: {},{},{}", t0, t1, t2);
```

也可以以此让函数返回多个值：

```rust
fn calculate_values() -> (i32, i32) {
    let x = 5;
    let y = 10;
    (x, y)
}
```

#### 数组

长度是固定的。

用数组而不用vector通常是为了保存固定长度的数据或者让数据存在栈而非堆上。

```rust
let array = [12,34,63];
```

标注类型时可以标注长度，如：
```rust
let arr: [i32:5] = [1,2,3,4,5]
```

`let arr = [3;5]`等价于`let arr = [3,3,3,3,3]`。

raft禁止数组越界访问，会在运行时报错。编译时只能检查出简单的越界错误。


## 语句与表达式

>表达式加了分号就算是语句了。

语句的返回值为空的Tuple：`()`，而表达式返回其值。块的最后一个表达式或语句会成为块的返回值：

```rust
let y = {
	let x=2;
	x+1
};

let z = if 99>88{ 1 } else { -1 }; //为了满足一致性，以此替代三目表达式
```
## 函数

函数不需要关心前后关系，不需要前向声明。

```rust
fn fun_name(x: i32,y: i32) -> i32 {
	println!("fun {} {}",x,y);
	x+y
}
```

## Range类型

`(1..4)`就是个Range类型，表示 $[1,4)$ ，即1,2,3。`(1..4).rev()`则翻转变成3,2,1。而`(1..=4)`则为 $[1,4]$ ，包含4。

可以被for循环遍历，也可使用迭代器方法。

```rust
//阶乘
pub fn factorial(num: u64) -> u64 {
	(1..=num).fold(1, |acc, e| acc * e)
}
```
## for循环

```rust
let arr = [11, 22, 33, 44, 55];  
//使用迭代器
for ele in arr.iter() {  
    println!("for1: {}", ele);  
}
//使用Range
for index in 0..5 {  
    println!("for2: {}", arr[index]);  
}
//enumerate对迭代器进行包装，返回Tuple
for (i,&ele) in arr.iter().enumerate(){  
    println!("for3: {} {}", i, ele);  
}
```

for不仅可以结构，还能通过`&item`接收来进行解引用，可以理解为`&item=&i32`即为`item=i32`。

```rust
fn func1(list: &[i32]){
	let mut largest: i32 = 0;
	for &item in list{
		if item > largest {
			largest = item;
		}
	}
	largest
}
//等价于
fn func2(list: &[i32]){
	let mut largest: i32 = 0;
	for item in list{
		if *item > largest {
			*largest = item;
		}
	}
	largest
}
```

## 结构体

```rust
//定义
struct StructName{
	variable_name: type,
	...
}
```

```rust
//实例化
//缺少字段会报错
let object_name = StructName{
	variable_name: value,
	...
};
```

整个struct要么都可变，要么都不可变。

像js一样，字段值和给它赋值的变量名一样时可简写，如`v_name: v_name`可直接写成`v_name`。

struct的所有引用变量都要进行生命周期标注。标注后，该变量的生命周期必须要长于struct。

### struct更新语法

如果S的两个实例s1和s2只有若干字段不同，则可以先给这几个字段赋值，然后加上`..s2`：

```rust
let person = Person {
	name: String::from("Alice"),
	age: 20,
};
let updated_person = Person {
	name: String::from("Bob"),
	..person
};
```

### Tuple Struct

类似于对Tuple的typedef。

```rust
struct Color(i32,i32,i32);
let black = Color(0,0,0);
```

### Unit-Like Struct

没有定义任何字段是struct，与`()`（空Tuple）（单元类型）相似。想实现trait但不需要存储数据的时候会用到。


### 方法

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

方法的第一个参数总是self，非引用、不可变引用或可变引用都行。

调用方法时，会自动给调用方法的对象加上`&`、`&mut`或`*`，以匹配方法中的self。

>一个结构体的可以写多个impl块。

### 关联函数

如果定义的时候没有传入self，则称为关联方法。对象无法调用关联方法，只能使用`结构体名::关联函数名`的形式调用，类似于其他语言的静态函数。

```rust
struct Circle {
    radius: f64,
}

impl Circle {
    fn new(radius: f64) -> Circle {
        Circle { radius }
    }
}
```

### 私有性

给struct加上pub后，可以被上级模块访问到，但其内部的成员默认都是私有的。想让成员公有则必须在成员的前面也加上pub。

### 结构化绑定

以结构体的方式解构变量。

```rust
let mut a = A {
        x: 1,
        y: 2,
    };
let A { x, y } = &mut a; //x和y都是&mut引用且生命周期相同
```

## 枚举

枚举的值种类叫做变体。枚举的访问使用`MyEnum::V1`。

```rust
enum Direction {
    North,
    South,
    East,
    West,
}

let direction = Direction::North;
```

变体可以携带数据，且各个变体能有不同的类型：

```rust
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
}

let msg = Message::ChangeColor(0, 160, 255);
```

枚举也能和方法一样用impl定义方法。

### 私有性

给enum加上pub后，可以被上级模块访问到，且其内部的成员默认都为公共。


## Option枚举

是为了去除null而诞生的，表示该类型变量可空。不用`Option<T>`的都可以直接视为不可能空，可放心使用。

在预导入模块中，`Option<T>`、`Some(T)`、`None`都是能直接用的。

```rust
//标准库中的定义
enum Option<T>{
	Some(T),
	None, //表示变量为空
}
```

`Option<T>`和`T`被视为两种类型，不能混用。若想把`Option<T>`当成`T`用，则需要先进行类型转换（相当于强制检测一次null）。

```rust
let num1: Option<i32> = None;
let num2: Option<i32> = Some(5);
```

使用match对空进行判断：

```rust
match x {
	None => println!("Empty"),
	Some(i) => println!("{}",i),
}
```

### unwrap_or_default()

使用`unwrap_or_default()`，在Some时返回提取的内容，在None时返回Default trait的`default()`的结果。

### and() and_then()

`and(x)`: 如果是Some，则返回`Some(x)`，否则直接返回None。

`and_then(|x| {...})`: 如果是Some，则将内部数据当做闭包参数调用闭包，且闭包返回值类型是Option；否则直接返回None。
## 错误

Rust没有异常系统。

- 可恢复错误: `Result<T,E>`
- 不可恢复错误: `panic!`宏

### 不可恢复错误 Panic

panic发生时，默认会**展开（unwind）** 调用栈，往回走，把遇到的所有函数的数据都给清理掉。可以选择不展开，而是**中止（abort）**，不清理而直接停止程序，由操作系统回收内存。中止也可以让二进制文件更小。

```toml
[profile.release]
panic = 'abort'
```

为了让panic发生时打印回溯错误信息，需要：

```bash
#windows
set RUST_BACKTRACE=1 && cargo run
#linux
RUST_BACKTRACE=1 cargo run
```

### 可恢复错误 Result

```rust
enum Result<T,E> {
	Ok(T),
	Err(E),
}
```

可以把T或E写成`()`表示不关心该返回值。
#### 错误处理

可以提取出Err的参数热然后调用kind()以进一步match错误类型：

```rust
use std::fs::File;
use std::io::ErrorKind;

let file = File::open("non_existent_file.txt");

match file {
    Ok(_) => println!("File opened successfully"),
    Err(e) => match e.kind() {
        ErrorKind::NotFound => println!("File not found"),
        ErrorKind::PermissionDenied => println!("Permission denied"),
        other => println!("Some other error: {:?}", other),
    },
}
```

也可以直接调用**unwrap**方法，如果是Ok则返回Ok的数据，否则调用panic!。但无法自定义panic信息。

```rust
let file = File::open("non_existent_file.txt").unwrap();
```

若调用**expect**方法即可自定义panic信息。

```rust
let file = File::open("non_existent_file.txt").expect("Expect Panic Msg");
```

#### 错误传递

只要返回值为result，就可以传递错误，如可以写为`Result<String,io::Error>`。

使用`?`运算符可以在输出为Result::Err的时候自动将其return。

但是使用`?`时要注意，它在成功的时候返回的并不是Result，因此正确地返回时应当用Ok再做包装。

在`?`返回的Err与要求的返回值不匹配时，会隐式地调用from函数进行转换，前提是转化的目标错误对原错误实现了from函数。

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_file() -> io::Result<String> { //等价于Result<String,io::Error>
    let mut f = File::open("file.txt")?;
    let mut buffer = String::new();

    f.read_to_string(&mut buffer)?; //等价于match到Err(e)后return Err(e)

    Ok(buffer)
}

fn main() {
    match read_file() {
        Ok(content) => println!("File content: {}", content),
        Err(e) => println!("An error occurred: {}", e),
    }
}
```

#### Result转Option

使用`ok()`或`err()`方法可以将Result枚举转换为Option枚举：
```rust
let x: Result<u32, &str> = Ok(2);
assert_eq!(x.ok(), Some(2));

let x: Result<u32, &str> = Err("Nothing here");
assert_eq!(x.ok(), None);
```
### 错误使用指南

如果确定必然是Ok的，则直接使用unwrap获取结果值。

## 解构赋值

```rust
let p = Point{ x: 1 , y: 3 };
let Point{ x_num , y_num } = p;
let Point{ x , y } = p; //解构的变量名和结构体的成员变量名相同的时候可简写
```

## 静态变量 static

```rust
static MY_STATIC_VA: &str = "Hello World";
static mut COUNT: u32 = 0;
```

- 使用SCREAMING_SNAKE_CASE来命名
- 必须指定类型
- 存储引用时，生命周期只能是`'static`

即使是可变静态变量，对其的修改也是不安全的，要在unsafe中进行。[Unsafe Rust](Rust.md#Unsafe%20Rust)

不能直接用RefCell，因为不能保证线程安全（有内部可变性的类型也都受此约束），必须unsafe地实现Sync trait，对其做一层封装。例如：
```rust
pub struct UPSafeCell<T> {
    /// inner data
    inner: RefCell<T>,
}

unsafe impl<T> Sync for UPSafeCell<T> {}

impl<T> UPSafeCell<T> {
    /// User is responsible to guarantee that inner struct is only used in
    /// uniprocessor.
    pub unsafe fn new(value: T) -> Self {
        Self { inner: RefCell::new(value) }
    }
    /// Panic if the data has been borrowed.
    pub fn exclusive_access(&self) -> RefMut<'_, T> {
        self.inner.borrow_mut()
    }
}
```

当一个静态变量需要在运行时进行赋值时，可以使用`lazy_static`外部库，让`lazy_static!`宏中的静态变量在首次被使用时才赋值。
## main函数

main函数的默认返回值为`()`，即单元类型。但也可以改成`Result<String, Box<dyn std::error::Error>>`，表示可以接收**任意错误**，但要在main末尾添上`Ok(())`。

```rust
use std::fs::File;
use std::io::Read;

fn read_file() -> Result<String, Box<dyn std::error::Error>> {
    let mut f = File::open("file.txt")?;
    let mut buffer = String::new();

    f.read_to_string(&mut buffer)?;

    Ok(buffer)
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let content = read_file()?;
    println!("File content: {}", content);
    Ok(())
}
```

# 所有权

对于某个值（在堆中），当拥有它的变量走出作用范围时，立刻会自动调用drop函数将此堆内存释放。

当一个值以Move的形式被赋给另一个值的时候，先前的值会失效。这样可以防止在调用drop的时候出现二次释放。**标量类型及其组合（如标量组合的Tuple）不会触发Move**。

```rust
let s1=String::from("abc");
let s2=s1; //此时s1完全不可用
```

想触发拷贝则需要使用clone函数：

```rust
let s1=String::from("abc");
let s2=s1.clone(); //s1依旧可用，且堆内存也被拷贝了（深拷贝）
```

对于完全存放在栈上的数据类型，它们不需要Move，因此如果一个数据类型实现了Copy这个trait，则赋值的时候**会进行克隆而非Move**。但是，如果实现了Drop trait，就**禁止实现Copy trait**，以防止二次drop。TODO：似乎工程意义上要先实现Clone trait？

函数的传参与返回都会触发所有权的转移。因此若String被作为普通的参数传入函数的话，会在函数结束时失效。

## 引用

```rust
let ref = &v;
```

引用此时有了新的解释：能获取值，但不转移所有权。把引用作为函数参数的行为叫做**借用**。有借有还。引用想可变也需要mut。

- **在同一个作用域内，对同一块数据最多只能有一个可变引用。** 以防范数据竞争。
- **当存在可变引用时，不能有不可变引用。** 以保证不可变引用的语义一致性。实际上，当存在不可变引用时，就能保证不能对数据发生更改，因为很多更改实际上是传了`&mut v`作为参数，即可变借用也会判为可变引用。

引用的作用域是**声明->最后一次使用**。因此，所有不可变引用不再使用后就马上可以进行修改。

**悬空引用（Dangling Reference）**（可理解为野指针）**永远不会发生**，因为编译时，若引用未离开作用域（如被函数返回）但数据离开作用域的话就会报错。

## 借用变量转化为所有权变量 `to_owned()`

对一个实现了Clone trait的引用（如`&str`）进行`to_owned()`，就会通过拷贝得到一个带有所有权的变量（如`String`）。



# 生命周期

[深入生命周期 - Rust语言圣经(Rust Course)](https://course.rs/advance/lifetime/advance.html)

生命周期的目的是为了避免悬空引用。

编译器在编译时会列出各个变量生命周期的长度，保证不发生数据失效后的使用。但是有些情况无法在编译时得知生命周期：
```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
//即使这样也会报错
fn longest(x: &str, y: &str) -> &str {
    x
}
```

编译器不看函数内部，而**只看函数签名**来推断返回值的生命周期。

## 函数生命周期标注

`<'a>`为泛型生命周期参数。而生命周期标注（`&'a str`）仅仅用于描述关系，而不**直接**影响生命周期长度。

下面的代码给所有参数和返回值都进行了同一个标注，表示在该函数范围内它们的生命周期完全一样。从因果逻辑上来看，`'a`首先获得s1和s2的生命周期，并取交集，即取比较短的部分，然后规定返回值的生命周期必须在这部分里有效。

```rust
// 这是一个函数，它接受两个字符串切片，并返回一个字符串切片。
// 'a 是一个生命周期参数，它表示输入参数和返回值的生命周期必须“相同”。
fn longest<'a>(s1: &'a str, s2: &'a str) -> &'a str {
    if s1.len() > s2.len() {
        s1
    } else {
        s2
    }
}

//正常的代码
fn main() {
    let string1 = String::from("long string is long");
    let result;
    let string2 = String::from("xyz");
    result = longest(string1.as_str(), string2.as_str());
    println!("The longest string is {}", result);
}

//会报错，因为result的生命周期取为string2，不能超出下一个花括号。
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

参数的交互、借用的时间也会受到生命周期标注的影响：

```rust
fn main(){
	let mut my_vec: Vec<&i32> = vec![];
	let val1 = 1;
	let val2 = 2;
	
	insert_value(&mut my_vec , &val1);
	insert_value(&mut my_vec , &val2);
}

//错误，因为push需要保证value要活的比my_vec久，但在此函数内无法推断
fn insert_value(my_vec: &mut Vec<& i32> , value: & i32){  
    my_vec.push(value);  
}

//错误，因为这会使得my_vec的可变引用被延迟到val1或val2被释放，导致多个可变引用同时存在
fn insert_value<'a>(my_vec: &'a mut Vec<&'a i32> , value: &'a i32){
	my_vec.push(value);
}

//正确，让Vec与其值的生命周期独立开，只要求值见的生命周期同步。当然也要求Vec的值比Vec活得久
fn insert_value<'r , 'val>(my_vec: &'r mut Vec<&'val i32> , value: &'val i32){
	my_vec.push(value);
}

//正确，无关的生命周期标注也可直接省略
fn insert_value<'val>(my_vec: &mut Vec<&'val i32> , value: &'val i32){
	my_vec.push(value);
}
```

## 生命周期自动推断

如果返回值的生命周期可以被推断，则可以省略。推断规则：
1. 每一个是引用的参数都有它自己的生命周期参数
2. 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数
3. 如果方法有多个输入生命周期参数并且其中一个参数是 &self 或 &mut self， 那么所有输出生命周期参数被赋予 self 的生命周期

## struct生命周期标注

struct的所有引用变量都要进行生命周期标注。标注后，该变量的生命周期必须要长于struct。标注了生命周期的struct必须在impl的时候在impl后面也标注一个，即`impl<'a> ST<'a>{...}`。

```rust
struct MyStruct<'a> {
    my_ref: &'a i32,
}

impl<'a> MyStruct<'a> {
    fn new(ref_to_i32: &'a i32) -> MyStruct<'a> {
        MyStruct { my_ref: ref_to_i32 }
    }

    fn print(&self) {
        println!("{}", self.my_ref);
    }
}

fn main() {
    let value = 5;
    let my_struct = MyStruct::new(&value);
    my_struct.print();
}
```

但如果impl里面并没有任何函数要用到生命周期标注的话，可以使用`impl Mystruct<'_>{...}`。其他推荐显式标注生命周期、但完全可以由编译器推导的情况下，也可以使用`'_`。

## 静态生命周期

`'static`是特殊的生命周期，表示整个程序的运行时间。如所有字符串字面值都有此生命周期。

`<T: 'static>`让T成为`'static`的子类，只有当T为引用的时候才要求必须是`'static`，否则是有所有权的变量的话不做任何要求。

## 型变

[Subtyping and Variance - The Rust Reference](https://doc.rust-lang.org/stable/reference/subtyping.html)

目的：让子可以完全处理父。

若有一个类型构造器`Foo(Class)`，以类型为参数来产生另一个类型。设A是B的父类。
1. 若`Foo(A)`与`Foo(B)`*无关*，则称为**不变（invariant）**
2. 若`Foo(A)`是`Foo(B)`的*父类*，则称为**协变（covariant）**
3. 若`Foo(A)`是`Foo(B)`的*子类*，则称为**逆变（contra-variant）**

原文：
- `F<T>` is _covariant_ over `T` if `T` being a subtype of `U` implies that `F<T>` is a subtype of `F<U>` (subtyping "passes through")
- `F<T>` is _contravariant_ over `T` if `T` being a subtype of `U` implies that `F<U>` is a subtype of `F<T>`
- `F<T>` is _invariant_ over `T` otherwise (no subtyping relation can be derived)

>不变实际上是要求A和B必须完全相同才能在转化后兼容。

若A 是B的父类，B是C的父类，则`fn A => C`是`fn B => B`的父类，因为参数为A的函数必然能处理参数B，而返回值为C的函数必然也能返回B（即`fn A => C`无论是入参还是返回值都可以直接在外部视为`fn B => B`）。因此fn类型对参数类型是逆变的，对返回值类型是协变的。

|Type|Variance in `'a`|Variance in `T`|
|---|---|---|
|`&'a T`|covariant|covariant|
|`&'a mut T`|covariant|invariant|
|`*const T`||covariant|
|`*mut T`||invariant|
|`[T]` and `[T; n]`||covariant|
|`fn() -> T`||covariant|
|`fn(T) -> ()`||**contravariant**|
|`std::cell::UnsafeCell<T>`||invariant|
|`std::marker::PhantomData<T>`||covariant|
|`dyn Trait<T> + 'a`|covariant|invariant|

可见只有函数参数是逆变的；涉及可变的类型（而非生命周期）基本都是不变；剩下都是协变。

如果T不变，但T形如`TypeName<X>`，则X也不变。

```rust
struct Bar<'r>{
	_phantom: PhantomData<fn(&'r ())>, //逆变套协变=逆变
}
fn bar<'short, 'long: 'short>(
	mut short_bar: Bar<'short>,
	mut long_bar: Bar<'long>)
{
	//short_bar=long_bar; //编译不通过
	long_bar=short_bar; //编译通过
}

//如果两个参数都改成mut short_bar: &mut Bar<'short>，就会导致short_bar和long_bar互相都不能赋值
```

下面是一个比较典型的例子：

```rust
struct Manager<'val>{  
    text: &'val str,
}

//不好的写法，因为&'a是协变，但<'a>是不变，因此Interface<'a>是不变
//这导致此结构体没有子类型，无法使其生命周期短于Manager<'a>
//struct Interface<'a>
//	manager: &'a mut Manager<'a>
//}

//好的写法，解耦使'r保持协变，可以让Interface提前消亡而不影响Manager
struct Interface<'r, 'val>{
	manager: &'r mut Manager<'val> //隐式定义'val: 'r
}

//这种解耦可以有效防止出现过长的、不可测的借用
//因此生命周期标注可以看做是缩短、切割、独立而不是延长、接续、衔接
struct List<'val>{
	manager: Manager<'val>,
}

impl<'val> List<'val>{
	//解耦了：1.该方法对self的借用；2.self本身（所有权变量）的生命周期
	//相当于要求这个借用一直持续到Interface消亡（由于'val: 'r所以不用管'val）
	pub fn get_interface<'r>(&'r mut self) -> Interface<'r, 'val> {
		Interface{
			manager: &mut self.manager,
		}
	}

	//也可以省略'r，让编译器自己推断
	pub fn get_interface_2(&mut self) -> Interface<'_, 'val> {  
	    Interface{  
	        manager: &mut self.manager,  
	    }  
	}
}

fn main(){  
    let mut list = List{  
        manager: Manager{  
            text: "abc",  
        }  
    };  
  
    let i1 = list.get_interface();  
    println!("1: {}",i1.manager.text);  
    let i2 = list.get_interface();  
    println!("2: {}",i2.manager.text);  
    
    //然而下面这个是过不了的，因为i1这个interface还没消亡，因此对list的借用没结束
	//let i1 = list.get_interface();  
    //let i2 = list.get_interface();
    //println!("1: {}",i1.manager.text);  
    //println!("2: {}",i2.manager.text);
}
```

## reborrow

>&T实现了Copy，但&mut T没有

下面的代码中，`let y: &mut i32 = x;`并不会发生Move，而是等价于`let y: &mut i32 = &mut *x;`，将x解引用后再次创建可变引用，即reborrow。当y消亡后，x又可以继续使用。如果提前使用x，就会提示x已经被借用了；也就是说，reborrow相当于对引用进行一次借用。

```rust
fn main(){
	let mut i = 42;
	let x = &mut i;
	let y: &mut i32 = x;
	//y对x重引用后，y成为了i的唯一可变引用
	*y=43;
	println!("y {}",*y);
	//此处y消亡，x重新变成i的唯一可变引用
	*x=44;
	println!("x {}",*x);
}
```

实际上，将一个可变引用传参到`func_name(val_name: &mut T)`也是一次重引用。

## 对引用的引用

`&'b &'a StructName`即为`&'b (&'a StructName)`。解引用时先把`'b`给解了。

`&'b &'a StructName`=>`'a: 'b`，因为`&'a StructName`作为一个`&T`收到了`'b &T'`的制约，因此`'a`不得短于`'b`。

`*rb`产生`'c`，`'c: 'b`，因为`'c`是`&'b &'a`解引用出来的，因此`&'b &'c`也要合法。

>可以简单记忆成*返回`'a`和`'b`中生命周期短的那个*。

```rust
//返回为&'b时，参数无论怎样mut都是合法的，证略
//返回的'b并不是指'b层的引用，而是返回的'c: 'b，因此返回'b必然合法
fn func_name<'b, 'a>(rb: &'b &'a StructName) -> &'b StructName {
	*rb
}

//返回为&'b mut时，仅当参数为&'b mut &'a mut时才是合法的，实际上是进行一次reborrow
//&'b &'a mut是不行的，因为&'b不可变，可以拷贝多份，因此显然不能随意拿出它里面的可变引用
fn func_name<'b, 'a>(rb: &'b mut &'a mut StructName) -> &'b mut StructName {
	*rb
}

//返回为&'a时，参数只能是&'b &'a或&'b mut &'a，即要求参数的&'a不能是mut，以进行copy
fn func_name<'b, 'a>(rb: &'b mut &'a StructName) -> &'a StructName {
	*rb
}

//返回为&'a时，参数是什么都不行，只有如下情况可以使用
fn func_name<'b: 'a, 'a>(rb: &'b mut &'a mut StructName) -> &'a StructName {
	*rb
}
```

## 延长函数内新变量的生命周期

涉及到高阶trait绑定。

下面这段代码无法通过编译，因为如果在函数上定义了`'a`，则`'a`默认比函数体要长，因此不接纳zero。

```rust
fn call_on_ref_zero<'a, F>(f: F)
where
	F: Fn(&'a i32),
{
	let zero = 0;
	f(&zero);
}
```

将代码改成如下形式即可：

```rust
fn call_on_ref_zero<F>(f: F)
where
	F: for <'a> Fn(&'a i32), //表示类型参数F可以接纳任意的'a
	//F: Fn(&'a i32) 也可以直接这样写，因为Fn本身就是高阶的生命周期绑定。
{
	//zero的生命周期依然只有这两行
	let zero = 0;
	f(&zero);
}
```



# 库与语法

## expect

可能失败的方法会返回Result，是个枚举类，为`Ok(val)`或`Err(_)`，可以交由expect处理（`result.expect(msg)`）。若Result为Ok，则返回Result中存储的结果值val；否则，打印expect的参数msg并停止程序。
## match

匹配到符合的结果，执行其后的语句，且可以有返回值。分支称为arm。

编译时要求必须处理所有情况。使用`_`通配符或者用一个变量接住即可匹配剩余所有情况。

```rust
let value = 5;

match value {
	1 => println!("one") //arm
	2 => {...}
	_ => {...}
}

match value {
    1 => println!("one"),
    2 => println!("two"),
    //用变量接住
    other => println!("The value is: {}", other),
}
```

```rust
match guess.cmp(&secret_number){
	Ordering::Less=>println!("Too small!"), 
	Ordering::Greater=>println!("Too big!"),
	Ordering::Equal=>println!("You win!"),
}
```

对于枚举类型、结构体、Tuple等等，也可以在模式匹配的时候将其绑定的值赋值到变量中。即使存在嵌套，也可以解构成功。只想占位的话使用`_`来防止发生不必要的所有权转移。

```rust
let guess: u32 = match str.trim().parse(){
	OK(num)=>num*2,
	Err(_)=>{
		println!("You Wrong!"),
		println!("OOPS"),
	}
};
```

```rust
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
}

fn process_message(msg: Message) {
    match msg {
        Message::Quit => {
            println!("The Quit variant was passed in.");
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change the color to red {}, green {}, and blue {}", r, g, b);
        }
        Message::Move { x, y: new_y } => {
            println!("Move in the x direction {} and in the y direction {}", x, new_y);
        }
        Message::Write(text) => {
            println!("Text message: {}", text);
        }
    }
}
```

### match 守卫

在箭头之前加上if就能进一步判断以进行过滤：

```rust
fn main() {
    let x = 5;

    match x {
        // 如果x是1、2或3时，打印"Small"
        1 | 2 | 3 if is_even(x) => println!("Small and even"),
        1 | 2 | 3 => println!("Small"),
        // 如果x是4到10之间的偶数，打印"Medium and even"
        y @ 4..=10 if is_even(y) => println!("Medium and even"),
        // 如果x是4到10之间的数，打印"Medium"
        4..=10 => println!("Medium"),
        // 其他情况，打印"Large"
        _ => println!("Large"),
    }
}

fn is_even(num: u32) -> bool {
    num % 2 == 0
}
```

### if let

当要操作的情况只有一个，其他情况都要忽略时，就能用if let简化：

```rust
let option = Some(5);

if let Some(5) = option {
    println!("Five");
} else if let Some(6) = option {
    println!("Six");
} else {
    println!("Not five or six");
}
```

和if的区别是，if let使用了match的模式匹配，可以提取绑定值。可以搭配else、else if使用。

if let可以和普通if语句（包括else if）混写，只是带let的判断会利用模式匹配。

## 切片 slice

切片是对数组型数据结构的一部分的引用，不具有所有权。字符串切片类型记为`&str`。

```rust
let mut s = String::from("abcdef");  
let s1=&s[2..4];  
let s2=&s[2..=4];  
println!("slice: {} {}",s1,s2);
//输出：slice: cd cde
//即Range包含的数字都会被提取
//可以省略前置0或后置字符串长度。因此&str[..]即是对整个数组的切片
```

对`i32`字符串的切片类型为`&[i32]`。

使用`len()`方法可以获得切片的元素数量。

## 字符串

String是Byte的集合，采用UTF-8编码，是在标准库中的。本质是对`Vec<u8>`的封装。

字符串字面值（本质是字符串切片，不可变引用）是硬编码的，访问快但本身不可变。其对应的变量就是个字符串切片，是个不可变引用，类型为`&str`。`&str`转String类型要使用`String::from`。

只有`&str`可以转`String`，反之不行。

```rust
let mut str = String::from("abc");
str.push_str("def");
let mut str1 = "abc1".parse().unwrap();
```

加法运算合并字符串时，只有第一个参数可以是String：
```rust
let s1 = "Hello";
let s2 = "World";
let merged = s1 + s2;

let s11 = String::from("Hello");
let s22 = String::from("World");
let mergedd = s1 + &s2;
```

函数参数如果是字符串的话，最好使用`&str`类型来表示参数，即强制要求传切片。这使得在传参为切片的时候可以直接调用，传参为String的时候则要求创建完整切片（Rust会自动转换）后传入，更加通用了。这也可以使得字符串传参没有任何的副作用。

使用`+`即可连接两个String。但`+`本质还是调用函数，因此如果`let s=s1+&s2`，会导致调用完毕后s2还能接着用，但s1的所有权被剥夺了。

`formit!`用法类似于`println!`，可以用于字符串拼接：
```rust
let name = "Alice";
let age = 30;
let s = format!("{} is {} years old.", name, age);
println!("{}", s);
```

**不支持索引访问字符**。因为UTF-8编码下**某些字符会占多个字节**，难以识别。因此直接调用len方法返回的是占用的字节数，也就不一定等于字符个数。

- bytes方法: 将字符串看成纯粹的Byte数组，返回byte类型的迭代器。
- chars方法: 将字符串看成标量值数组，返回char类型的迭代器。
- 标准库中尚未提供按字符簇访问String的方法。

字符串切片的时候，切片的索引是按byte来的。如果切片的索引不是char的边界（相当于把字符给分裂了），则会panic。

```rust
fn string_slice(arg: &str) {
    println!("{}", arg);
}

fn string(arg: String) {
    println!("{}", arg);
}

fn main() {
    string_slice("blue");
    string("red".to_string());
    string(String::from("hi"));
    string("rust is fun!".to_owned());
    string("nice weather".into());
    string(format!("Interpolation {}", "Station"));
    string_slice(&String::from("abc")[0..1]); //切片
    string_slice("  hello there ".trim()); //去掉前后缀空白
    string("Happy Monday!".to_string().replace("Mon", "Tues")); //文本替换
    string("mY sHiFt KeY iS sTiCkY".to_lowercase()); //大小写替换
}
```
## Vector

```rust
let v1: Vec<i32> = Vec::new();
let v2 = vec![1,2,3];
```

```rust
//泛型可以通过第一个插入值来自动推断
let mut v = Vec::new();
v.push(1); //自动推断出i32
```

可以使用索引和get两种方式访问值。但是索引如果越界，会直接报错；get会返回`Option<T>`，如果越界了会返回None：

```rust
let v = vec![1, 2, 3, 4, 5];

match v.get(2) {
    Some(value) => println!("The third element is {}", value),
    None => println!("There is no third element."),
}
```

遍历vector：

```rust
let mut v = vec![2,5,7];
for i in &mut v {
	*i += 50;
}
```

使用附带数据的枚举可以实现用vector存储不同类型的数据。编译器推导出vector的泛型是枚举后，就相当于得知了它的有限个类型种类，也就可以生成对应的操作应对方案。

```rust
enum Data {
    Integer(i32),
    Float(f64),
    Text(String),
}

fn main() {
    let data = vec![
        Data::Integer(10),
        Data::Float(3.14),
        Data::Text(String::from("Hello")),
    ];

    for item in &data {
        match item {
            Data::Integer(i) => println!("Integer: {}", i),
            Data::Float(f) => println!("Float: {}", f),
            Data::Text(t) => println!("Text: {}", t),
        }
    }
}
```

### swap_remove

通过和最后一个元素交换来实现$O(1)$地删除任意一个元素，但这样会破坏顺序。要保持顺序的话需要用remove，是$O(n)$
## HashMap

相当于unordered_map。

不在预导入模块中，需要`use std::collections::HashMap`。

可以使用collect来从形如(k,v)的Tuple数组中构造HashMap。collect方法适用于很多集合，因此需要给返回值显式指定类型`HashMap<_, _>`。

```rust
use std::collections::HashMap;

let vec = vec![("key1", 1), ("key2", 2), ("key3", 3)];
let map: HashMap<_, _> = vec.into_iter().collect();
println!("{:?}", map);
```

```rust
//利用zip可以从两个vector构造HashMap
use std::collections::HashMap;

let keys = vec!["key1", "key2", "key3"];
let values = vec![1, 2, 3];

let map: HashMap<_, _> = keys.into_iter().zip(values.into_iter()).collect();

println!("{:?}", map);
```

HashMap直接传值会剥夺所有权。传引用的话要注意数据不能提前失效。

遍历：
```rust
for (k,v) in &map {
	...
}
```

insert同一个key会导致覆盖。可以用`contains_key()`来检测一个key是否存在于HashMap中

entry(key)方法可以返回一个枚举Entry，是key对应的值的入口，可以用于表示key是否存在。Entry有or_insert(value)方法，如果key存在则返回key对应的值的可变引用，如果不存在则插入参数value然后返回对应的值的可变引用。

默认的HashMap安全但不是最快的，可以替换hasher。

## BTreeMap

使用B树实现的Map，是cpp的红黑树map的替代品，可以通过next和next_back来迅速访问最小值和最大值。

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

可以在某个type上实现某个trait的前提条件是：这个type **或** 这个trait 是**本crate**里定义的。也就是说，无法为外部的类型定义外部的trait。这样可以保证不会出现两个crate分别给同一个struct实现同一个trait的情况。即**孤儿规则（Orphan Rule）**，防止trait的实现是个孤儿（既不属于定义trait的crate，也不属于定义type的crate）。但也因此可能无法做到为某些外部结构体实现Debug、Display，再套一层结构体即可解决：

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

### 典型trait
#### deref trait

唯一一种隐式转化逻辑。

deref trait里面有`fn deref(&self) -> &T`。

- 解引用符号`*y`作用于实现了deref trait的类型的时候，会自动变成`*(y.deref())`，即**隐式解引用转化（Deref Coercion）**。
- 在函数传递的参数类型不匹配的时候，编译器会试图调用deref来进行转换。如`&String`就有可能会转换成`&str`。

- 当`T:Deref<Target=U>`时，允许`&T`或`&mut T`转换为`&U`。
- 当`T:DerefMut<Target=U>`时，允许`&mut T`转换为`&mut U`。

#### drop trait

drop trait里面有`fn drop(&mut self)`。会在变量离开作用域时自动调用，被当做析构函数，可用于释放资源。

不能直接地显式调用，但可以调用`std::mem::drop(value)`，以提前释放变量（调用drop）。

### 可直接derive的经典trait

#### Eq & PartialEq

生成相等的相关逻辑。

PartialEq不实现反身性，即没有`a == a`。

浮点类型只实现了PartialEq，因为`NaN!=NaN`。

#### Ord & PartialOrd

生成比大小（Order）的相关逻辑。

PartialOrd不实现确定性（必定存在`>`或\=\=或`<`其中的一个关系）。它允许存在无法比较的两数，返回None。

PartialOrd继承了PartialEq。PartialOrd完成后提供`lt()`，`le()`，`gt()`，`ge()`。

Ord继承了Eq和PartialOrd。Ord完成后提供`max()`，`min()`，`clamp()`。

## 动态分发与静态分发

- 动态分发: dyn trait，运行时调用对应实现的方法。使用胖指针（两倍指针大小）来描述传入的指针对应的类型，这样调用方法时就能正确找到函数位置。**胖指针是虚表的替代品**。
- 静态分发: impl trait，会在编译期单态化，因此并不能同时适配多种类型，如作为函数返回值类型的之后返回两种类型。

智能指针`Box<TraitName>`逻辑上没问题，但已被弃用，应当使用`Box<dyn TraitName>`。

## 闭包

```rust
fn main() {
    // 定义一个闭包，它接受两个i32类型的参数，返回它们的乘积
    let multiply = |x: i32, y: i32| -> i32 {
        let result = x * y;
        println!("The product of {} and {} is {}", x, y, result);
        result
    };

    // 调用闭包
    let product = multiply(4, 5);
    println!("Product: {}", product);
}
```

闭包不要求显式指定参数和返回值的类型，可以依靠编译器进行上下文推断。

表达式只有单个的时候可以省略花括号，比如`let add_x = |y| y + x;`。

每个闭包类型都有独特的类型，即使它们长得完全一样。

>如果想让一个复杂运算只执行一次（延迟计算），则可以维护一个结构体，里面含有一个闭包，和一个存储**参数->返回值**的HashMap。

闭包至少实现了下面三个trait中的一个，且实现了前面的之后就必然也实现了所有后面的：
- Fn: 对定义所在作用域中的变量进行不可变借用。
- FnMut: 对定义所在作用域中的变量进行可变借用。
- FnOnce: 用到定义所在作用域中的变量时就夺取所有权（也因此这个闭包只能调用一次）。

一个闭包可以被`struct STName<T> where T: Fn(i32) -> i32 {...}`使用。

在参数列表前使用move关键字，就可以强制获取所使用的所有环境值的所有权。可用于将闭包传递给新线程并把数据都给新线程使用的时候。

由于闭包是trait，是动态的，无法直接作为变量类型、返回值等，但可以使用`Box<dyn Fn(i32) -> i32>`。

## 函数指针

函数指针类型为`fn(i32)->i32`。其中fn为**类型**而非trait。

函数指针**必然**同时实现了Fn、FnMut、FnOnce三个trait。因此函数指针也能当闭包用。

对一个数组的每一项都做同一件事的时候，我们可以用map然后传入一个闭包，也可以传入一个函数指针，此时遍历到的项会作为参数传入函数：
```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> =
list_of_numbers.iter().map(|i| i.to_string()).collect();
let list_of_strings: Vec<String> = list_of_numbers.iter().map(ToString::to_string).collect();
```

在带数据的枚举中，构造枚举的构造器也是个函数，因此也能传入map：
```rust
enum Status {
	Value(u32),
	Stop,
}

//变为Status::Value数组
let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
```
## 迭代器

迭代器实现了Iterator trait。需要指定type Item并实现一个next方法。next返回None意味着迭代结束。

>迭代器循环遍历会做循环展开优化。

```rust
pub trait Iterator {
    type Item; //迭代器的返回类型
    fn next(&mut self) -> Option<Self::Item>; //遍历方法
    // methods with default implementations elided
}
```

- `iter`: 遍历的时候获取到的都是不可变引用。
- `iter_mut`: 遍历的时候获取到的都是可变引用。
- `into_iter`: 遍历的时候会获取所有权。

next方法本质是对迭代器的消耗（因为不能遍历第二遍）。

Iterator trait中有一些带默认实现的方法，如果方法实现里面调用了next，就称为**消耗型适配器**，如sum方法。还有些方法将迭代器转化为不同的迭代器，称为**迭代器适配器**，如map方法。

如果next不配调用（或者说没有消耗型适配器被调用）的话，迭代器**不会做任何事**，包括调用的迭代器适配器。

可以使用`next_back`反向迭代：
```rust
let numbers = vec![1, 2, 3, 4, 5, 6];

let mut iter = numbers.iter();

assert_eq!(Some(&1), iter.next());
assert_eq!(Some(&6), iter.next_back());
assert_eq!(Some(&5), iter.next_back());
assert_eq!(Some(&2), iter.next());
assert_eq!(Some(&3), iter.next());
assert_eq!(Some(&4), iter.next());
assert_eq!(None, iter.next());
assert_eq!(None, iter.next_back());
```

[Iterator in std::iter - Rust](https://doc.rust-lang.org/std/iter/trait.Iterator.html)

迭代器函数可能是lazy的，不一定会立刻执行！

```rust
let a = [1, 2, 3];
//map对所有元素作某事后，返回新的迭代器
//因此闭包的传入与输出类型不同时可以实现类型转换
let mut iter = a.iter().map(|x| 2 * x);

assert_eq!(iter.next(), Some(2));
assert_eq!(iter.next(), Some(4));
assert_eq!(iter.next(), Some(6));
assert_eq!(iter.next(), None);
```

```rust
//使用for_each可以用闭包替代for循环
//不返回新东西，且立即执行
(0..5).flat_map(|x| x * 100 .. x * 110)
      .enumerate()
      .filter(|&(i, x)| (i + x) % 3 == 0)
      .for_each(|(i, x)| println!("{i}:{x}"));
```

```rust
let numbers = vec![10, 20, 30, 40, 50];

//enumerate为迭代值标上索引号
for (index, number) in numbers.iter().enumerate() {
    println!("Index: {}, Number: {}", index, number);
}
```

```rust
//剔除不小于maxLen的元素
dataList.into_iter().filter(|x| -> x.dataLen < maxLen).collect();
```

```rust
//阶乘
pub fn factorial(num: u64) -> u64 {
	//acc: accumulator, e: element
	//初始时acc=第一个参数，然后和所有元素依次执行闭包
	(1..=num).fold(1, |acc, e| acc * e)
	//如果第一个参数完全可以当成迭代器的第一个元素的话，可以改用reduce，但要求迭代器不为空
}
```

```rust
let a = [0, 1, 2, 3, 4, 5];
//step_by改变next的行为
let mut iter = a.iter().step_by(2);

assert_eq!(iter.next(), Some(&0));
assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), Some(&4));
assert_eq!(iter.next(), None);
```
## 类型别名

```rust
type newTypeName = i32;
type Thunk = Box<dyn Fn() + Send + 'static>;
type Result<T> = Result<T, std::io::Error>;
type Result<T> = std::io::Result<T>;
```

## Never类型

`!`即为Never类型，可以被转换为其他任意类型。并不是void，不返回并不是Never。

## 动态大小类型

动态大小类型（DST）只有在运行时才能知道占多少内存空间，且一旦确定就不能再变化。

比如str类型，如果用str类型赋值了长为3的字符串，就不能再用于长为4的字符串。

所有trait都是动态大小类型，要么使用引用来访问，要么放在指针里面。

Sized trait: 要求编译时已知大小的类型都会隐式地实现这个trait。泛型函数都会隐式添加该trait bound，因此泛型函数的实现类型必须是编译时已知大小的，但可以加`?Sized trait`来解除限制。

?Sized trait: 表示T可能是也可能不是确定大小的，可以强制让泛型函数使用动态大小类型。

## 智能指针

### Box

最简单的智能指针，指向堆上的内存，没有开销或额外功能。

- 编译时无法确定大小，但使用的时候又要求知道大小。
- 有大量数据要移交所有权，且要保证不能被复制。
- 使用某个值，只关心其是否实现了某trait，而不关心具体类型。

使用`Box::new(value)`创建Box指针，且会转移所有权。

### Rc std::rc::Rc

引用计数（Reference Count）的智能指针，可共享数据。只能用于单线程。不在预导入模块，在`std::rc::Rc`。

Rc全都是不可变引用，用于共享只读，因为有多个可变引用的话是违背借用规则的。

`Rc::new(value)`创建Rc指针，且会转移所有权。

`Rc::clone(&rc_pointer)`可以复制一个Rc指针，令引用计数器+1。

>`rc_pointer.clone()`似乎完全等价于`Rc::clone(&rc_pointer)`，但语义上有歧义。

`Rc::strong_count(&rc_pointer)`会返回引用计数。

#### Weak

为了防止循环引用导致引用计数永远不为0、因此出现内存泄漏的情况，可以使用`Rc::downgrade(&rc_pointer)`来创建弱引用的智能指针`Weak<T>`，并使弱引用计数+1。

`Rc::weak_count(&rc_pointer)`会返回弱引用计数。

**弱引用计数不为0也不影响数据清理**，当强引用计数为0的时候，弱引用会自动断开。

弱引用有方法`weak_pointer.upgrade()`，返回`Option<Rc<T>>`，以检测数据是否被清理了，若没有被清理则返回强引用。

```rust
fn main() {
    let strong = Rc::new("Hello, world!".to_string());
    let weak = Rc::downgrade(&strong);

    if let Some(weak_ref) = Weak::upgrade(&weak) {
        // 强引用还存在，可以安全地访问
        println!("Strong: {}", *weak_ref);
    } else {
        // 强引用已断开
        println!("Strong reference has been dropped.");
    }
}
```

### RefCell std::cell::RefCell

RefCell指针支持可变引用和不可变引用，而其安全检查在运行时；拥有数据的所有权，不共享。不在预导入模块，在`std::cell::RefCell`。即使是不可变引用，也能修改其值。

>**内部可变性（interior mutability）** **可变地借用一个不可变的值**，从而允许在持有不可变引用的情况下修改数据。指针本身对外是不可变的数据，但依旧提供接口来改变内部数据。

方法（安全接口）：
- `borrow()`: 返回`Ref<T>`（不可变），实现了deref。
- `borrow_mut()`: 返回`RefMut<T>`（可变），实现了deref。

>因为实现了deref，因此borrow到了之后解引用即可直接使用。

会**分别**对borrow出的Ref和RefMut进行引用计数，以此保证**运行时**满足**借用规则**（只有多个不可变借用，或只有一个可变借用）。

使用`Rc<RefCell<T>>`可以使得数据可以被多方共享且能修改，只要保证读写完之后引用的`RefCell<T>`马上消失即可。

>核心库中也有类似的指针core::cell::RefCell，因为它不需要多少运行时环境

### Cell

通过复制而非借用来访问数据，有内部可变性。

### Cow std::borrow::Cow

使用`Cow::from(xxx)`来对xxx建立Clone-On-Write的访问机制。当Cow不具有xxx的所有权时，其类型为`Cow::Borrowed(xxx)`；具有所有权时为`Cow::Owned(xxx)`。可以使用`is_borrowed()`和`is_owned()`或者模式匹配来判断其类型。

使用`to_mut()`来获得可变引用。如果此时未拥有所有权，则对值进行复制，变成`Cow::Owned(xxx)`。

```rust
use std::borrow::Cow;
  
fn abs_all<'a, 'b>(input: &'a mut Cow<'b, [i32]>) -> &'a mut Cow<'b, [i32]> {
    for i in 0..input.len() {
        let v = input[i];
        if v < 0 {
            // Clones into a vector if not already owned.
            input.to_mut()[i] = -v;
        }
    }
    input
}
  
#[cfg(test)]
mod tests {
    use super::*;
  
    #[test]
    fn reference_mutation() -> Result<(), &'static str> {
        // Clone occurs because `input` needs to be mutated.
        let slice = [-1, 0, 1];
        let mut input = Cow::from(&slice[..]);
        match abs_all(&mut input) {
            Cow::Owned(_) => Ok(()),
            _ => Err("Expected owned value"),
        }
    }
  
    #[test]
    fn reference_no_mutation() -> Result<(), &'static str> {
        // No clone occurs because `input` doesn't need to be mutated.
        let slice = [0, 1, 2];
        let mut input = Cow::from(&slice[..]);
        match abs_all(&mut input) {
            Cow::Borrowed(_) => Ok(()),
            _ => Err("Expected borrowed value"),
        }
    }
  
    #[test]
    fn owned_no_mutation() -> Result<(), &'static str> {
        // We can also pass `slice` without `&` so Cow owns it directly. In this
        // case no mutation occurs and thus also no clone, but the result is
        // still owned because it was never borrowed or mutated.
        let slice = vec![0, 1, 2];
        let mut input = Cow::from(slice);
        match abs_all(&mut input) {
            Cow::Owned(_) => Ok(()),
            _ => Err("Expected owned value"),
        }
    }
  
    #[test]
    fn owned_mutation() -> Result<(), &'static str> {
        // Of course this is also the case if a mutation does occur. In this
        // case the call to `to_mut()` returns a reference to the same data as
        // before.
        let slice = vec![-1, 0, 1];
        let mut input = Cow::from(slice);
        match abs_all(&mut input) {
            Cow::Owned(_) => Ok(()),
            _ => Err("Expected owned value"),
        }
    }
}
```

### Arc std::sync::Arc

类似于Rc\<T\>，但使用原子操作来进行引用计数，是线程安全的。

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let numbers: Vec<_> = (0..100u32).collect();
    let shared_numbers = Arc::new(numbers);
    let mut joinhandles = Vec::new();
  
    for offset in 0..8 {
        let child_numbers = shared_numbers.clone();
        joinhandles.push(thread::spawn(move || {
            let sum: u32 = child_numbers.iter().filter(|&&n| n % 8 == offset).sum();
            println!("Sum of offset {} is {}", offset, sum);
        }));
    }
    for handle in joinhandles.into_iter() {
        handle.join().unwrap();
    }
}
```

### Mutex std::sync::Mutex

用于实现跨线程情形下的内部可变性模式，有内部可变性。

结合Arc使用即可进行线程安全的读写共享：
```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
	        //对Mutex进行lock后unwrap就能获得值，并且在离开作用域后自动解锁
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

>Arc和Mutex的正确配合可以达到支持多线程安全读写数据对象。如果需要多线程共享所有权的数据对象，则只用Arc即可。如果需要修改 `T` 类型中某些成员变量 `member` ，那直接采用 `Arc<Mutex<T>>` ，并在修改的时候通过 `obj.lock().unwrap().member = xxx` 的方式是可行的，但这种编程模式的同步互斥的粒度太大，可能对互斥性能的影响比较大。为了减少互斥性能开销，其实只需要在 `T` 类型中的**需要被修改的成员变量**上加 `Mutex<_>` 即可。如果成员变量也是一个数据结构，还包含更深层次的成员变量，那应该继续下推到最终需要修改的成员变量上去添加 `Mutex`
## 类型转换

>解引用 `Deref` Trait 是 Rust 编译器唯一允许的一种隐式类型转换，而对于其他的类型转换，我们必须手动调用类型转化方法或者是显式给出转换前后的类型。
### as

用于类型转换和import的重命名。

### From/Into trait

[From in std::convert - Rust](https://doc.rust-lang.org/std/convert/trait.From.html)

当我们为`U`实现了`From<T>`之后，Rust会自动为`T`实现`Into<U>`Trait：
```rust
// Use the `from` function
let p1 = Person::from("Mark,20");
// Since From is implemented for Person, we should be able to use Into
let p2: Person = "Gerald,70".into();
```

### AsRef AsMut

```rust
// Obtain the number of bytes (not characters) in the given argument.
// TODO: Add the AsRef trait appropriately as a trait bound.
fn byte_counter<T: AsRef<str>>(arg: T) -> usize {
    arg.as_ref().as_bytes().len()
}
  
// Obtain the number of characters (not bytes) in the given argument.
// TODO: Add the AsRef trait appropriately as a trait bound.
fn char_counter<T: AsRef<str>>(arg: T) -> usize {
    arg.as_ref().chars().count()
}
  
// Squares a number using as_mut().
// TODO: Add the appropriate trait bound.
fn num_sq<T: AsMut<u32>>(arg: &mut T) {
    // TODO: Implement the function body.
    *arg.as_mut() *= *arg.as_mut();
}
  
#[cfg(test)]
mod tests {
    use super::*;
  
    #[test]
    fn different_counts() {
        let s = "Café au lait";
        assert_ne!(char_counter(s), byte_counter(s));
    }
  
    #[test]
    fn same_counts() {
        let s = "Cafe au lait";
        assert_eq!(char_counter(s), byte_counter(s));
    }
  
    #[test]
    fn different_counts_using_string() {
        let s = String::from("Café au lait");
        assert_ne!(char_counter(s.clone()), byte_counter(s));
    }
  
    #[test]
    fn same_counts_using_string() {
        let s = String::from("Cafe au lait");
        assert_eq!(char_counter(s.clone()), byte_counter(s));
    }
  
    #[test]
    fn mult_box() {
        let mut num: Box<u32> = Box::new(3);
        num_sq(&mut num);
        assert_eq!(*num, 9);
    }
}
```

## 并发编程

Rust标准库只提供1:1模型，即通过调用OS API来创建线程，有较小的运行时（宿主环境），可以方便与C交互。

使用`std::thread::spawn`传入一个闭包即可创建一个新线程，返回值为一个handle，对handle调用join即可让当前线程阻塞直到handler其完成。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handler = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handler.join().unwrap();
}
```

### move闭包

如果线程需要获得某个变量的所有权，则在闭包前面加上`move`。这样的话，直到该线程结束之前，其他线程都不能使用此变量。

```rust
fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

### channel

使用`std::sync::mpsc::channel()`即可创建一个channel，返回元组`(发送端tx,接收端rx)`。

>mpsc = mutiple producer, single consumer

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

使用`try_recv()`方法即可不阻塞地尝试接收一次，如果接收失败则返回Err。

对发送方tx进行clone即可建立多个发送方。

## 宏 macro

- 使用`macro_rules!`构建的**声明宏（declarative macros）**（弃用）。
- 三种**过程宏（procedural macros）**，接收并操作输入的rust代码，然后返回新的rust代码：
	- 自定义`#[derive]`宏，只能用于struct或enum,可以为其指定随derive属性添加的代码。
	- 类似属性的宏，在任何条日上添加自定义属性。
	- 类似函数的宏，看起来像函数调用，对其指定为参数的token进行操作。

宏定义前的代码无法使用此宏，要注意宏定义顺序。
### macro_rules!

用macro_rules!书写的宏的语法类似于match语句。定义时名字不用加感叹号。

```rust
macro_rules! my_macro {
    () => {
        println!("Check out my macro!");
    };
}
  
fn main() {
    my_macro!();
}
```

>in this situation, the value is the literal Rust source code passed to the macro; the patterns are compared with the structure of that source code; and the code associated with each pattern, when matched, replaces the code passed to the macro. This all happens during compilation

>The `#[macro_export]` annotation indicates that this macro should be made available whenever the crate in which the macro is defined is brought into scope. Without this annotation, the macro can’t be brought into scope.

```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

```rust
mod macros {
    #[macro_export]
    macro_rules! my_macro {
        () => {
            println!("Check out my macro!");
        };
    }
}
  
fn main() {
    my_macro!();
}
```

可以有多个分支情况：
```rust
macro_rules! my_macro {
    () => {
        println!("Check out my macro!");
    };
    ($val:expr) => {
        println!("Look at this other macro: {}", $val);
    };
}
  
fn main() {
    my_macro!();
    my_macro!(7777);
}
```
### derive宏

写出`#[derive(SomeName)]`后，会匹配标注了`#[proc_macro_derive(SomeName)]`的函数，并把`#[derive(SomeName)]`标注的代码段作为输入传入，生成的输出会贴到`#[derive(SomeName)]`所在的地方的下面。

### 类似属性的宏

```rust
#[route(GET,"/")]
fn index(){
	...
}

#[proc_macro_attribute] 
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
	...
}
```

和derive一样进行匹配和传参，但会额外传TokenStream,即`GET, "/"`。

### 类似函数的宏

```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);

#[proc_macro] 
pub fn sql(input: TokenStream) -> TokenStream {
	...
}
```

把括号里面的字符都传到TokenStream，进行处理。可以看做其参数可无视rust语法限制的函数。

## Unsafe Rust

被unsafe修饰的代码块允许：
- 解引用原始指针（raw pointer）
- 调用unsafe函数或方法
- 访问或修改可变的静态变量
- 实现unsafe trait

### 原始指针 raw pointer

原始指针：
- 可变：`*mut T`
- 不可变：`*const T` （不能通过它对其所指的数据赋值）

目的：
- 与C语言进行接口
- 构建借用检查器无法理解的安全抽象

可变指针不算引用，因此可变不可变**不受借用规则约束**，也因此打破了借用规则（但直接用引用的话依旧无法打破）。

可能空指针、野指针，也不自动释放。

在安全代码块里面也能定义原始指针，但禁止解引用。

原始指针也分可变和不可变。

```rust
let mut num = 5;
let r1 = &num as *const i32;

let address = 0x012345usize;
let r2 = &num as *mut i32;

let slice: &mut [i32] = ...;
let r3 = slice.as_mut_ptr();
```

#### 原始指针转为引用

当原始指针作为结果时，可以将其转化为`&'static mut`，这样就不需要unsafe也能使用该数据：
```rust
fn get_current_mem_set(&self) -> &'static mut MemorySet {
	let ptr = &mut result as *mut MemorySet;
	unsafe{&mut *ptr}
}

pub fn get_ref<T>(&self, offset: usize) -> &T where T: Sized {
	let type_size = core::mem::size_of::<T>();
	assert!(offset + type_size <= BLOCK_SZ);
	let addr = self.addr_of_offset(offset);
	unsafe { &*(addr as *const T) }
}
```
#### 原始指针和Box互相转换

```rust
/// # Safety
///
/// The `ptr` must contain an owned box of `Foo`.
unsafe fn raw_pointer_to_box(ptr: *mut Foo) -> Box<Foo> {
    // SAFETY: The `ptr` contains an owned box of `Foo` by contract. We
    // simply reconstruct the box from that pointer.
    let mut ret: Box<Foo> = unsafe { Box::from_raw(ptr) };
    ret.b = Some("hello".to_owned());
    ret
}
  
#[cfg(test)]
mod tests {
    use super::*;
    use std::time::Instant;
  
    #[test]
    fn test_success() {
        let data = Box::new(Foo { a: 1, b: None });
  
        let ptr_1 = &data.a as *const u128 as usize;
        // SAFETY: We pass an owned box of `Foo`.
        let ret = unsafe { raw_pointer_to_box(Box::into_raw(data)) };
  
        let ptr_2 = &ret.a as *const u128 as usize;
  
        assert!(ptr_1 == ptr_2);
        assert!(ret.b == Some("hello".to_owned()));
    }
}
```

### 安全抽象

在安全的函数里面可以使用unsafe代码块，因此要在unsafe代码块的外面确保unsafe是safe的。这个必须人为保证。

### unsafe trait

定义trait的时候加上unsafe，则它只能在unsafe的代码块中实现，一般直接`unsafe impl`。

## extern

[Unsafe Rust - The Rust Programming Language](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#using-extern-functions-to-call-external-code)

[External blocks - The Rust Reference](https://doc.rust-lang.org/reference/items/external-blocks.html)

extern简化创建和使用外部函数接口（FFI，foreign function interface）的过程。

使用类似`extern "C" {...}`可以声明外部的不同语言的函数；`"C"`就是ABI（application binary interface），通过指定汇编规则来指定语言。**extern代码块里的所有东西都是unsafe的**。

在fn前加上`extern "C"`即可创建接口供其它语言调用。若不加则等价于`extern "Rust"`。添加`#[no_mangle]`注解可以防止编译期间发生名称更改。

```rust
extern "Rust" {
    fn my_demo_function(a: u32) -> u32;
    #[link_name = "my_demo_function"]
    fn my_demo_function_alias(a: u32) -> u32;
}
  
mod Foo {
    // No `extern` equals `extern "Rust"`.
    #[no_mangle]
    fn my_demo_function(a: u32) -> u32 {
        a
    }
}
  
#[cfg(test)]
mod tests {
    use super::*;
  
    #[test]
    fn test_success() {
        // The externally imported functions are UNSAFE by default
        // because of untrusted source of other languages. You may
        // wrap them in safe Rust APIs to ease the burden of callers.
        //
        // SAFETY: We know those functions are aliases of a safe
        // Rust function.
        unsafe {
            my_demo_function(123);
            my_demo_function_alias(456);
        }
    }
}
```

## 文档注释

文档注释也分为单行注释和块注释，但又有内外之分：

- 内部文档注释（Inner doc comment）
	- 单行注释（以 /// 开头）
	- 块注释（用 /** ... */ 分隔）
- 外部文档注释（Outer doc comment）
	- 单行注释（以 //! 开头）
	- 块注释（用 /*! ... */ 分隔）

二者的区别：
- 内部文档注释是对它之后的项做注释，与使用 `#[doc="..."]` 是等价的。
- 外部文档注释是对它所在的项做注释，与使用 `#![doc="..."]` 是等价的。

另外，在文档注释中可以使用 Markdown 语法。

## 汇编

[Inline assembly - The Rust Reference](https://doc.rust-lang.org/reference/inline-assembly.html)

`include_str!`可以将一个文件的内容给复制粘贴（如c语言的include）进来；而`global_asm!`可以嵌入全局汇编代码。

```rust
use core::arch::global_asm;
global_asm!(include_str!("entry.asm"));
```

相比 `global_asm!` ， `asm!` 宏可以获取上下文中的变量信息并允许嵌入的汇编代码对这些变量进行操作。由于编译器的能力不足以判定插入汇编代码这个行为的安全性，所以我们需要将其包裹在 unsafe 块中自己来对它负责。

```rust
 2use core::arch::asm;
 3fn syscall(id: usize, args: [usize; 3]) -> isize {
 4    let mut ret: isize;
 5    unsafe {
 6        asm!(
 7            "ecall",
 8            inlateout("x10") args[0] => ret,
 9            in("x11") args[1],
10            in("x12") args[2],
11            in("x17") id
12        );
13    }
14    ret
15}
```
# 问题、技巧、解决方案

## 完整地打印到输出

使用`#[derive(Debug)]`派生宏，使得ST由Debug trait派生；打印时使用`{:?}`即可行内打印，使用`{:#?}`即可换行打印。这样可以完整地打印结构体或枚举。

```rust
#[derive(Debug)]
struct ST{
	v1: i32,
	v2: f64,
}

let st = ST{
	v1: 1,
	v2: 3.5,
};
println!("{:?}",st)
println!("{:#?}",st)
//输出：
// ST { v1: 1, v2: 3.5 }  
// ST {  
//     v1: 1,  
//     v2: 3.5,  
// }
```

## 变量生命周期太长导致其借用太久

适当使用drop来消解这些持久的变量来进行重新借用。
## 生成随机数

```rust
//use rand::Rng;
let mut rng = rand::thread_rng();  
let random_number: u32 = rng.gen_range(0,5); //生成[0,5)的随机数
println!("random_num: {}", random_number);  
```

## 多个char拼接字符串

```rust
let chars = vec!['H', 'e', 'l', 'l', 'o'];
let string: String = chars.into_iter().collect();
```

不要使用String的push，因为String每次push都会重新分配内存，而vector的容量是指数倍增长的，重新分配的次数少。

## 函数传递多个参数

```rust
fn func(&[i32]){...}

fn main(){
	func(&[2,3,4]);
}
```

## 引用存活时间被意外延长

`match xxx {...}` 在`xxx`处的返回结果不会发生拷贝，导致里面的某些引用被延长到match结束。因此复杂的`xxx`取值最好在外面赋给一个变量然后再match。在if等语句中应该都会出现。

## 将结构体的成员变量Move出来

有时候一个结构体要销毁了，但它的成员变量还有用，又不好使用智能指针，也不应当clone时，可以考虑将其类型设为`Option<T>`，要Move走时就进行`obj.variable_name.take().unwrap()`将`T`取出。

## 可变引用一个结构体下的多个成员变量

```rust
#[derive(Debug, PartialEq, Eq)]
struct A {
    x: i32,
    y: i32,
}

impl A {
    /// 这个方法将self这个可变引用的生命周期转移到了字段x上。
    ///
    /// 这种函数相当于在这个生命周期范围内，对该结构体的
    /// 可访问范围从整个结构体**缩小**到了其中的某个字段上，
    /// 而舍弃了在这个生命周期内访问其他字段的权利。
    fn x(&mut self) -> &mut i32 {
        &mut self.x
    }

    /// 同理同上
    #[allow(unused)]
    fn y(&mut self) -> &mut i32 {
        &mut self.y
    }

    /// 这个方法将self可变引用的生命周期**拆分并分别转移**到了x和y上。
    fn x_and_y(&mut self) -> (&mut i32, &mut i32) {
        (&mut self.x, &mut self.y)
    }
}

fn main() {
    let mut a = A {
        x: 1,
        y: 2,
    };
    {
        let (x, y) = a.x_and_y();
        *x += 2;
        *y += 2;
        assert_eq!(a, A { x: 3, y: 4 });
    }
    {
        // 这里使用了另一种方法：结构化绑定。这种语法是模式匹配的一种，与在match中列举enum的各种枚举值的表达式是同一种语法。
        let A { x, y } = &mut a;
        *x += 2;
        *y += 2;
        assert_eq!(a, A { x: 5, y: 6 });

        match &mut a {
            A { x: 5, y } => assert_eq!(y, &mut 6),
            _ => panic!("x != 5"),
        }

        match &mut a {
            // 忽略了y。作用相当于通配符“_”。
            A { x, .. } => assert_eq!(x, &mut 5),
        }
    }
    // 错误用法。
    {
        let x = a.x();
        // let y = a.y();
        *x += 2;
        // *y += 2;
    }
}
```

## 将一段连续内存视为某一个类型的切片

core::slice::from_raw_parts

## 克隆`Option<Arc<T>>`

```rust
pub fn get_t_ref(&self) -> Option<Arc<T>> {
	self.t_var.as_ref().map(Arc::clone)
}
```

## 花式指定self类型

```rust
pub fn spawn(self: &Arc<Self>, elf_data: &[u8]) -> Arc<Self> {
	...
}
```


# 模块系统

按层级从高到低为：
- Package: 包，项目级别。Cargo的特性，可以构建、测试、共享Crate。
- Crate: 单元包，组件级别。一个模块树，可以生成library或可执行文件。
- Module: 模块，功能级别。控制代码的组织、作用域、私有路径。
- Path: 路径，为struct、function、module等项命名的方式。

## Package

Package包含：
- 1个Crate.toml，描述了如何构建这些Crates。
- 至多1个library Crate
- 不限量的binary Crate

在Crate.toml里的dependencies下可以添加外部包，以到`https://crates.io/`下载包。

std标准库是一个自动导入的外部包。
## Crate

Crate将相关功能组合到一个作用域内用于共享，避免冲突。

Crate类型：
- library
- binary

Crate Root: 是个源代码文件，Cargo将它交给编译器，编译器由此开始构建Crate，组成Crate的根Module。

Crate默认：
- 在binary中，main.rs作为crate root。
- 在library中，lib.rs作为crate root。
- Crate名与Package名相同。

binary crate可视为独立运行，无法共享功能给其他crate，也就无法写集成测试。一般逻辑都写在library crate里面，然后由简单的binary crate调用。

## Module

Module在一个Crate内将代码进行分组，增加可读性，同时控制项目（item）的私有性（private，public）。

Module可以嵌套，包含子Module。

子mod默认无法被上级发现，若想暴露需要使用`pub mod`。

使用mod关键字来定义模块：

```rust
mod outer {
    pub mod inner {
        pub fn inner_function() {
            println!("This is an inner function!");
        }
    }
}

fn main() {
    outer::inner::inner_function();
}
```

私有边界（Privacy Boundary）：默认所有条目（包括子模块）都是私有的，在条目或子模块前面加pub后才能被其他模块发现。父不能访问子的私有条目，但子可以访问所有的父的条目；同级之间也可以随意访问。

### 分文件存放模块

假如有模块`outer`和`outer::inner`，则需要构建如下文件结构：

```
src/
  main.rs
  outer/
    mod.rs
    inner.rs
```

```rust
//main.js
mod outer; //会去找../outer.rs或者../outer/mod.rs

fn main() {
	outer::outer_function();
    outer::inner::inner_function();
}
```

```rust
//outer/mod.rs
pub mod inner; //会去找../inner.rs或者../inner/mod.rs

pub fn outer_function() {
    println!("This is an outer function!");
}
```

```rust
//outer/inner.rs
pub fn inner_function() {
    println!("This is an inner function!");
}
```
## Path

Path用于在模块系统中定位项（如结构体、函数、模块等）

- 绝对路径: `use crate::module::SubModule::function;`
- 相对路径: `use module::SubModule::function;`

- 用`crate`表示根模块。
- 用`self`表示自己。
- 用`super`来表示父模块，功能类似于`../`。

### use

use将path导入到作用域内，但仍旧遵守私有规则，类似于cpp的using namespace。

对于函数，一般引入到其模块为止，否则直接引入到函数的话容易看不清函数归属于哪个模块；
但对于其他条目，则一般直接引入到条目本身，除非出现重名。

使用as（`use xxx as yy`）可以指定别名。

use过来的东西默认为私有的，无法被上级访问到。在use前面加pub即可让use的模块也能被上级使用。

可以使用嵌套路径来导入多个前缀相同的路径：
```rust
mod outer {
    pub mod inner {
        pub fn function1() {
            println!("This is function 1!");
        }

        pub fn function2() {
            println!("This is function 2!");
        }
    }
}

// 使用嵌套路径来导入inner、function1和function2
use outer::inner::{self, function1, function2};

fn main() {
	inner::function1();
    function1();
    function2();
}
```

`*`是通配符，表示“所有”。

### pub use

类似export，可以将某个Path条目以最少层数的Path对外暴露。
# Cargo

## 基本命令

- `cargo new ProjectName`: 创建项目。
- `cargo build`: 构建项目。加上`--release`则以发布方式构建，运行更快。会先更新crates.io，并生成（修改）lock文件使其符合toml。
- `cargo run`: 先编译（有必要的话）后执行。
- `cargo check`: 检查代码保证能过编译，但不会生成可执行文件（比build快）。
- `cargo test`: 执行项目中的所有测试。

## 依赖

`Cargo.toml`是对package或者workspace的description，称为manifast。virtual manifast则是仅仅描述一个workspace，且不包含任何一个package。

>toml = Tom's Obvious,Minimal Language

可以在`[dependencies]`后面添加依赖。等号后面可省略。`^`表示和所指版本API兼容的都可用。

```toml
[dependencies]  
rand = "^0.3.14"
```

lock文件（`Cargo.lock`）在build后生成，表示目前所用的确切版本。似乎默认使用最新小版本。

调用`cargo update`可以对各进行更新，且只更新小版本号，如0.3.14只会被更新到0.3.23而不是0.4.0。

## 工作空间

有多个crate时，可以在项目根目录创建一个用于聚合的toml，指定各个crate的路径，然后在对应路径中创建对应的crate（cargo new）。

```toml
[workspace]
members = [
    "package1",
    "package2",
]
```

crate之间的依赖需要在crate内的toml的dependencies通过路径指定。

## Build Script

[Build Scripts - The Cargo Book](https://doc.rust-lang.org/cargo/reference/build-scripts.html)

在跟目录下的`build.rs`中的rust代码会在包构建前运行。println出来的字符串会被用于告知cargo。

如打印`cargo:rustc-env=VAR=VALUE`就会建立环境变量`VAR`，其值为`VALUE`，可以在程序里使用`std::env::var("TEST_FOO").unwrap();`访问。

如打印`cargo:rustc-cfg=feature="pass"`就会使得`#[cfg(feature = "pass")]`为真。

用途：
- Building a bundled C library.
- Finding a C library on the host system.
- Generating a Rust module from a specification.
- Performing any platform-specific configuration needed for the crate.

```rust
// build.rs

use std::env;
use std::fs;
use std::path::Path;

fn main() {
    let out_dir = env::var_os("OUT_DIR").unwrap();
    let dest_path = Path::new(&out_dir).join("hello.rs");
    fs::write(
        &dest_path,
        "pub fn message() -> &'static str {
            \"Hello, World!\"
        }
        "
    ).unwrap();
    println!("cargo:rerun-if-changed=build.rs");
}

// src/main.rs

include!(concat!(env!("OUT_DIR"), "/hello.rs"));

fn main() {
    println!("{}", message());
}
```

```rust
fn main() {
    let timestamp = std::time::SystemTime::now()
        .duration_since(std::time::UNIX_EPOCH)
        .unwrap()
        .as_secs();
    let your_command = format!(
        "rustc-env=TEST_FOO={}",
        timestamp
    );
    println!("cargo:{}", your_command);

    let your_command = "rustc-cfg=feature=\"pass\"";
    println!("cargo:{}", your_command);
}
```
# 测试

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```

`#[cfg(test)]`的cfg是configuration，表示有指定的配置选项（test）才会执行。

使用`cargo test`来执行项目中的所有测试。

每个测试都在独立的线程里面。

一个测试panic了（让线程挂掉）则表示其失败。

使用`assert_eq!`和`assert_ne!`断言相等性，断言失败后会用Debug的形式打印两个参数（要求参数实现Debug trait）。

断言后可以继续加多个参数，使用类似`format!`的形式，作为自定义错误输出。

又是需要检测代码是否能在某个情况下成功触发panic。需要在测试函数（`#[test]`）下面再加上`#[should_panic]`，则panic的时候才算通过。为了防止不正常的panic也导致测试通过，可以使用`#[should_panic(expected = "abc abc")]`来检查panic信息中是否包含expected参数内的字符串，只有当包含了的时候才算测试通过。

测试函数也可以改成以Result为返回值，若返回的是Err则测试失败。

使用`#[ignore]`忽略测试。

## 测试组织

- 单元测试: 一次对一个module进行测试，可以测试到private接口。
- 集成测试: 在整个库的外部，像外部代码一样进行调用，只能测试public接口。

上面的写在代码crate内的`#[cfg(test)]` module就是单元测试。

集成测试文件要写在`tests`文件夹（与src文件夹同级）内。`tests`下的文件夹内的文件不会被视为集成测试文件。每个测试文件都是独立的crate。不用写mod，直接写`#[test]`测试函数即可。只有`cargo test`时才会编译。

`cargo test --test FileName`可以运行某个集成测试文件内的所有测试。

集成测试中如果有些文件只是为了提供帮助而未含有`#[test]`测试函数，则应该将其写到子文件夹里面去，防止出现无意义的空输出pass。

**binary crate意味着独立运行，无法构造集成测试。**


# 外部库

[日常开发三方库精选 - Rust语言圣经(Rust Course)](https://course.rs/practice/third-party-libs.html)
## log

[日志门面 log - Rust语言圣经(Rust Course)](https://course.rs/logs/log.html)

提供了几个宏来输出不同层级的log。

```rust
use log::{info, trace, warn};

pub fn shave_the_yak(yak: &mut Yak) {
    trace!("Commencing yak shaving");

    loop {
        match find_a_razor() {
            Ok(razor) => {
                info!("Razor located: {}", razor);
                yak.shave(razor);
                break;
            }
            Err(err) => {
                warn!("Unable to locate a razor: {}, retrying", err);
            }
        }
    }
}
```

但是log并不会直接输出到某个地方，需要配合日志库使用，如`env_logger`。

```rust
use log::{debug, error, log_enabled, info, Level};

fn main() {
    // 注意，env_logger 必须尽可能早的初始化
    env_logger::init();

    debug!("this is a debug {}", "message");
    error!("this is printed by default");

    if log_enabled!(Level::Info) {
        let x = 3 * 4; // expensive computation
        info!("the answer was: {}", x);
    }
}
```

## lazy static

当static不可变，但最初的初始化不太简单，是一个运行时执行的语句，就需要lazy static来进行延迟初始化。

```rust
lazy_static! {
    pub static ref SYSCALL_TIMES: UPSafeCell<[u32; MAX_SYSCALL_NUM]> = unsafe{UPSafeCell::new([0; MAX_SYSCALL_NUM])};
}
```


## bitflags

[bitflags - Rust](https://docs.rs/bitflags/1.2.1/bitflags/)

专门用于做比特标志位的crate，可以直接访问一个标志位数据结构中的各个bit标志。

```rust
bitflags! {
    pub struct PTEFlags: u8 {
        const V = 1 << 0;
        const R = 1 << 1;
        const W = 1 << 2;
        const X = 1 << 3;
        const U = 1 << 4;
        const G = 1 << 5;
        const A = 1 << 6;
        const D = 1 << 7;
    }
}
```









