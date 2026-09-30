[← Rust](Rust.md)

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

## toml字段

[Cargo.toml 清单详解 - Rust语言圣经(Rust Course)](https://course.rs/cargo/reference/manifest.html)

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
# 整个toml没有其他东西
[workspace]
members = [
    "package1",
    "package2",
]
```

crate之间的依赖需要在crate内的toml的dependencies通过路径指定。

```toml
[workspace]
resolver = "2"
members = [
    "crates/*",
]
```

## 生成文档

`cargo doc`: 生成文档（直接使用rustdoc似乎不会考虑依赖）。

[注释和文档 - Rust语言圣经(Rust Course)](https://course.rs/basic/comment.html)

使用`cargo doc --no-deps --open`即可不生成依赖项的文档，且构建完后打开页面。

## 条件编译

使用cfg则可以指定条件，只有条件为true时才能进行编译。

一般使用feature来进行条件编译：
```txt
#[cfg(feature = "no_gateway")]
```

如果想要实现“不存在此feature时才编译”，则使用not来反转条件：
```
#[cfg(not(feature = "no_gateway"))]
```

在`Cargo.toml`中声明feature:
```toml
[features]  
# 括号内是相关项, 开启本项后自动开启它们
no_gateway = []
```

## Build Script

[Build Scripts - The Cargo Book](https://doc.rust-lang.org/cargo/reference/build-scripts.html)

在根目录下的`build.rs`中的rust代码会在包构建前运行。println出来的字符串会被用于告知cargo。也可以在cargo里面指定要用的Build Script：

```toml
[package]
name = "your_project"
version = "0.1.0"
build = "build.rs"
```

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

## 其它

### 工具：代码内文档转README

其实就是删除每一行的前四个字符`//! `，使用以下工具代码：

```c
#include <iostream>
#include <fstream>
#include <string>

int main() {
    std::ifstream input("temp.txt");
    std::ofstream output("output.txt");
  
    if (input.is_open() && output.is_open()) {
        std::string line;
        while (std::getline(input, line)) {
            if (line.length() > 4) {
                output << line.substr(4) << std::endl;
            } else {
                output << "" << std::endl;
            }
        }
        input.close();
        output.close();
    } else {
        std::cout << "Unable to open file";
    }
  
    return 0;
}
```

# 测试

## 测试组织

- 单元测试: 一次对一个module进行测试，可以测试到private接口。
- 集成测试: 在整个库的外部，像外部代码一样进行调用，只能测试public接口。

上面的写在代码crate内的`#[cfg(test)]` module就是单元测试。

集成测试文件要写在`tests`文件夹（与src文件夹同级）内。`tests`下的文件夹内的文件不会被视为集成测试文件。每个测试文件都是独立的crate。不用写mod，直接写`#[test]`测试函数即可。只有`cargo test`时才会编译。

`cargo test --test FileName`可以运行某个集成测试文件内的所有测试。

集成测试中如果有些文件只是为了提供帮助而未含有`#[test]`测试函数，则应该将其写到子文件夹里面去，防止出现无意义的空输出pass。

**binary crate意味着独立运行，无法构造集成测试。**

## 单元测试与基本使用

```rust
#[cfg(test)]
mod tests {
	use super::*;
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```

`#[cfg(test)]`的cfg是configuration，表示有指定的配置选项（test）才会执行。

使用`cargo test`来执行项目中的所有测试。

每个测试都在独立的线程里面进行并发测试（因此设置了全局变量的话，要保证以肉眼顺序和并发量执行测试也不出问题）。

一个测试panic了（让线程挂掉）则表示其失败。

使用`assert_eq!`和`assert_ne!`断言相等性，断言失败后会用Debug的形式打印两个参数（要求参数实现Debug trait）。

断言后可以继续加多个参数，使用类似`format!`的形式，作为自定义错误输出。

又是需要检测代码是否能在某个情况下成功触发panic。需要在测试函数（`#[test]`）下面再加上`#[should_panic]`，则panic的时候才算通过。为了防止不正常的panic也导致测试通过，可以使用`#[should_panic(expected = "abc abc")]`来检查panic信息中是否包含expected参数内的字符串，只有当包含了的时候才算测试通过。
```rust
#[test]  
#[should_panic(expected = "Timeout before success")]  
fn test_once_fail() {  
    do_async_test(  
        RuntimeType::MultiThread,  
        test_once_fail_core(),  
    );  
}
```

测试函数也可以改成以Result为返回值，若返回的是Err则测试失败。

使用`#[ignore]`忽略测试。

## 集成测试

[单元测试和集成测试 - Rust语言圣经(Rust Course)](https://course.rs/test/unit-integration-test.html#%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95)

## 测试时打印输出

默认不会输出print的东西，要加个选项才能看到输出：
```sh
cargo test -- --nocapture ${test_name}
```

```sh
cargo test -- --show-output ${test_name}
```

但使用日志系统的话直接test就可以看到输出。日志的capture功能也可以通过[with_test_writer](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/fmt/struct.Layer.html#method.with_test_writer)启用。

# Docker 部署

在工作空间中的一个Dockerfile如下：

```dockerfile
FROM rust:latest as builder  
ARG APP_NAME  
WORKDIR /usr/src/${APP_NAME}  
COPY . .  
RUN cd ./crates/${APP_NAME} && \  
    cargo build --release  
  
FROM debian:buster-slim  
ARG APP_NAME  
COPY --from=builder /usr/src/${APP_NAME}/target/release/${APP_NAME} /usr/local/bin/${APP_NAME}  
CMD ["sh", "-c", "/usr/local/bin/$APP_NAME"]
```

由于app里面的toml可能有额外的配置，直接的在外部的cargo build就会需要使用`--manifest-path`来指定要使用的toml路径。因此使用cd直接进入对应app进行build会比较方便。

## chef 加速构建镜像

chef可以让依赖的下载与编译与源码解耦。只要源码的依赖不变，就不会重新构建依赖层。

```dockerfile
# cargo with source replacement  
# Comment the `RUN` if you are not in China.  
FROM rust:1.75.0-bookworm as source-replaced-cargo  
  
RUN mkdir -p /usr/local/cargo/registry \  
    && echo '[source.crates-io]' > /usr/local/cargo/config.toml \  
    && echo 'replace-with = "rsproxy"' >> /usr/local/cargo/config.toml \  
    && echo '[source.rsproxy]' >> /usr/local/cargo/config.toml \  
    && echo 'registry = "https://rsproxy.cn/crates.io-index"' >> /usr/local/cargo/config.toml  
  
# cargo-chef  
FROM source-replaced-cargo as chef  
  
RUN cargo install cargo-chef  
  
# Computes the recipe file  
FROM chef as planner  
  
WORKDIR /app  
COPY . .  
  
RUN cargo chef prepare --recipe-path recipe.json  
  
# Build app  
FROM chef as builder  
  
WORKDIR /app  
  
COPY --from=planner /app/recipe.json recipe.json  
# Build dependencies - this is the caching Docker layer.  
# No source code here, so as long as the dependency remains unchanged, it will not be rebuilt.  
RUN cargo chef cook --release --recipe-path recipe.json  
  
ARG APP_NAME  
  
COPY . .  
  
# Build  
RUN cd ./crates/${APP_NAME} \  
    && cargo build --release  
  
# Run app  
FROM debian:bookworm-slim  
  
EXPOSE 3000 4000  
  
ARG APP_NAME  
COPY --from=builder /app/target/release/${APP_NAME} /usr/local/bin/${APP_NAME}  
  
ENV APP_NAME_ENV=${APP_NAME}  
CMD ["sh", "-c", "/usr/local/bin/$APP_NAME_ENV"]
```

