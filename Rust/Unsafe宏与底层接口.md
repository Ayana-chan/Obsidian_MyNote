[← Rust](Rust.md)

## PhantomData

`PhantomData`不占用任何内存，它只在编译期有用。[PhantomData in std::marker - Rust](https://doc.rust-lang.org/std/marker/struct.PhantomData.html)

Rust禁止结构体有没被内部变量使用的生命周期参数或类型参数，遇到这种情况时可以用`PhantomData<&'a T>`等凑数。

作为结构体的内部变量，`PhantomData`会影响型变和Auto Trait的实现，如禁止Sync。

`PhantomData`还会影响Drop Check机制，这在unsafe Rust里面可能会经常考虑到。这篇文章对其Drop Check相关影响做了更多讨论：[PhantomData - The Rustonomicon](https://doc.rust-lang.org/nomicon/phantom-data.html)。

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

当原始指针作为一个safe函数的结果时，由于safe不能解引用，因此使用起来不方便。

可以将其转化为引用，这样就不需要unsafe也能使用该数据。当然也可以通过指定生命周期参数（或自动推导）来创建非static的引用。

核心思想就是使用unsafe来获取指针指向的数据的左值，然后对其取引用。这样的左值不会被Borrow checker管理。

>下面例子取自**rCore**。

例子的`get_current_mem_set`中，先获取了result的可变原始指针，然后在unsafe里面重新获取其左值并且进行static的可变引用。这样就实现了“对一个局部变量进行static的可变引用”。这种引用的功能是最强的，没有任何使用限制。

例子的`get_ref`中，addr是一个简单的数值，是一个地址的值。先将其转为`*const T`，即指向类型T的不可变指针；这样就能通过`*(addr as *const T)`得到类型T的数据（左值），对其取引用就得到了其不可变引用。

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

官方提供了转换的函数，不过Safety依然需要手动管理。

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

### Unbounded Lifetime

unsafe代码中很可能会在签名中出现一个与其他东西完全无关的lifetime，使得该lifetime基本没法约束任何东西。

```rust
fn get_str<'a>(s: *const String) -> &'a str {
    unsafe { &*s }
}

fn main() {
    let soon_dropped = String::from("hello");
    let dangling = get_str(&soon_dropped);
    drop(soon_dropped);
    // 非法访问，但是编译的时候无法探知
    println!("Invalid str: {}", dangling);
}
```

可以通过在参数中传入一些借用来产生生命周期关系。

结构体也一样，没使用的生命周期参数几乎没有约束结构体的价值，因此需要`PhantomData`来构建约束关系。例如结构体只需要在T的读引用合法期间使用，那么就可以放一个`PhantomData<&'a T>`。
## Unique

`NonNull`是对`*mut T`的封装，并且确保指针非空。使用`new_unchecked`可以在不检查空指针的情况下构建`NonNull`。使用`Option<NonNull<T>>`来表示可以为空的`*mut T`，且在内存中依然是一个指针。

`NonNull`被Drop的时候不会调用目标的Drop, 因此需要手动调用`drop_in_place`.

`NonNull`假设目标数据是有可能被共享的, 因此不实现Sync和Send.

## 宏 macro

- 使用`macro_rules!`构建的**声明宏（declarative macros）**。
- 三种**过程宏（procedural macros）**，接收并操作输入的rust代码，然后返回新的rust代码：
	- 自定义`#[derive]`宏，只能用于struct或enum,可以为其指定随derive属性添加的代码。
	- 类似属性的宏，在任何条日上添加自定义属性。
	- 类似函数的宏，看起来像函数调用，对其指定为参数的token进行操作。

宏定义前的代码无法使用此宏，要注意宏定义顺序。
### Declarative macros (macro_rules!)

用macro_rules!书写的宏的语法类似于match语句。定义时名字不用加感叹号。

用起来很像C语言的宏。

[Macros By Example - The Rust Reference](https://doc.rust-lang.org/reference/macros-by-example.html)

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

用例，快速定义特定结构体的静态变量：
```rust
macro_rules! define_static_error {  
    ($name:ident, $code:expr, $message:expr) => {  
        pub static $name: ResponseErrorStatic = ResponseErrorStatic {  
            code: $code,  
            message: $message,  
        };  
    };  
}  
  
define_static_error!(NET_COMMUCATION_FAIL, "C0601", "Err communication");  
define_static_error!(NET_UNKNOWN_ERROR, "C0602", "Err unknown");  
```

### Derive macros

写出`#[derive(SomeName)]`后，会匹配标注了`#[proc_macro_derive(SomeName)]`的函数，并把`#[derive(SomeName)]`标注的代码段作为输入传入，生成的输出会贴到`#[derive(SomeName)]`所在的地方的下面。

### Attribute-like macros

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

### Function-like macros

```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);

#[proc_macro] 
pub fn sql(input: TokenStream) -> TokenStream {
	...
}
```

把括号里面的字符都传到TokenStream，进行处理。可以看做其参数可无视rust语法限制的函数。

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

## 内存布局

[蚂蚁集团 ｜ Rust 数据内存布局 - Rust精选](https://rustmagazine.github.io/rust_magazine_2021/chapter_6/ant-rust-data-layout.html)

[浅聊 Rust 程序内存布局 - Rust语言中文社区](https://rustcc.cn/article?id=98adb067-30c8-4ce9-a4df-bfa5b6122c2e)
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

[生成文档](%E6%A8%A1%E5%9D%97Cargo%E4%B8%8E%E6%B5%8B%E8%AF%95.md#%E7%94%9F%E6%88%90%E6%96%87%E6%A1%A3)。

使用`include_str`来包含markdown文件来作为文档。且上下也能接着写文档（外面的文档可能不太好链接到具体项，这就需要在内部补充）。

```
#![doc = include_str!("../README.md")]  
  
//! # Usage  
//!  
//! Just look at the [`AsyncTasksRecorder`](AsyncTasksRecorder).  
//!
```

[Linking to items by name - The rustdoc book](https://doc.rust-lang.org/nightly/rustdoc/write-documentation/linking-to-items-by-name.html)
使用`[path]`即可按名字构建文档内超链接。也可以使用`[your_name](path)`以隐藏复杂的path：
```
//! Errors for API's response. \  
//! Use [`api::ApiResponse<T>`](crate::api::ApiResponse) as return type is more convenient, which is declared as `Result<Json<T>, ResponseError>`. \  
//! Look at [`ResponseError`] for more usage.
```

文档中的代码段默认为rust，也默认会被编译、运行。使用一些attributes写在语言处即可调整设置：[Documentation tests - Attributes - The rustdoc book](https://doc.rust-lang.org/nightly/rustdoc/write-documentation/documentation-tests.html#attributes)
