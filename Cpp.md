
# 链接

[C++11中关键字 - 知乎](https://zhuanlan.zhihu.com/p/157523014)

# 规范与注意事项

## 类

成员首先按public, protected, private的顺序分块。

每一块中成员的顺序如下：

1. typedef，enum，struct，class 定义的嵌套类型
2. 常量
3. 构造函数
4. 析构函数
5. 成员函数,含静态成员函数
6. 数据成员,含静态数据成员

构造函数的初始化列表中的每个项，应独占一行。

类中不能存在不知道大小的成员。前向声明并不能提供某个类的大小，但可以取它的指针，指针的大小是固定的。

# 库与语法

## 空指针

空指针指向特定范围的存储空间，这个空间永远不会与物理存储器有映射。

对空指针的delete不会报错。

>delete一个指针后，它不会指向空指针，而是指向非法空间，造成野指针错误。因此可以考虑delete一个指针后将其赋值为nullptr。

**nullptr是有类型的，且仅可以被隐式转化为指针类型。**

NULL可能会被定义为0，若有两个同名不同参的函数（重载），一个的参数类型为指针、另一个的参数类型为int，则传入NULL时可能执行参数类型为int的那一个函数。
## 命名空间

`using namespace xxx`表示使用xxx整个命名空间；而`using xxx:fx`表示使用xxx命名空间下的fx。

## explicit

[C++11 新特性 之 explicit关键字 - 显示构造与隐式构造\_explicit 构造\_cpp\_learners的博客-CSDN博客](https://blog.csdn.net/cpp_learner/article/details/117883912)

定义一个类的构造函数时，默认可以通过赋值的形式经由此构造函数构建对象：

```cpp
Student S1(22); // 显示构造 
Student S2 = 23; // 隐式构造
```

而在构造函数前加上`explicit`即可禁用该构造函数的隐式构造。
## 模板
### using

```cpp
// 重定义unsigned int
typedef unsigned int uint_t;
using uint_t = unsigned int;
// 重定义std::map
typedef std::map<std::string, int> map_int_t;
using map_int_t = std::map<std::string, int>;
```

```cpp
template <typename Val> 
using str_map_t = std::map<std::string, Val>; 
// ... 
str_map_t<int> map1;
```

```cpp
/* C++98/03 */
template <typename T>
struct func_t
{
    typedef void (*type)(T, T);
};
// 使用 func_t 模板
func_t<int>::type xx_1;
/* C++11 */
template <typename T>
using func_t = void (*)(T, T);
// 使用 func_t 模板
func_t<int> xx_2;//模板别名（alias template）
```

### 函数模板的默认模板参数
```cpp
template <typename T = int>
void func() { 
	// ... 
}
```

然后即可直接像函数一样调用，不需要指定类型（如果是类模板的话即使有默认模板参数也要在使用时传入类型）。

```cpp
int main() { 
	func(); //T = int 
	return 0; 
}
```

>没有必须写在参数表最后的限制。甚至于，根据实际场景中函数模板被调用的情形，编译器还可以自行推导出部分模板参数的类型。

```cpp
template <typename R = int, typename U>
R func(U val){
    return val;
}
int main(){
    func(97);               // R=int, U=int
    func<char>(97);         // R=char, U=int
    func<double, int>(97);  // R=double, U=int
    return 0;
}
```

## std::move

把一个左值强行变成右值以进行赋值，则相当于将左值对应的内存的所有权进行转交，原左值变成零值。

## 后置返回值类型
[模板函数——后置返回值类型（trailing return type）\_模板函数返回\_HerofH\_的博客-CSDN博客](https://blog.csdn.net/qq_28114615/article/details/100553186)

```cpp
template <class T>  
auto Get(std::string_view key) const -> const T *;
```

auto为占位符，实际上是箭头后的`T*`。

## 智能指针
### unique_ptr转shared_ptr

```cpp
std::shared_ptr<T>(std::move(ptr))
```

### make_unique新建对象

泛型为类，参数会被传入类的构造函数。

```cpp
std::make_unique<TrieNodeWithValue<T>>(children_, value_)
```

## std::ref

传入参数时就指定使用引用输入（平时都是在接收的时候才决定是引用）。
## string_view
对string的一个引用，不发生拷贝。

## 右值相关

[CPP11-右值引用 - 简书](https://www.jianshu.com/p/06b0b17c62bc)

要把右值引用看成用于接住右值、给它续命的东西。

## 完美转发

函数接收参数的时候，右值等可能会发生变化导致右值变左值等问题。使用`std::forward<T>(x)`可以保证其不发生改变。

## 函数（谓词）
### 仿函数（闭包）

若一个类重载了`operater()`，则其实现是个可调用对象，也叫仿函数、闭包。

### Lambda表达式

```cpp
[ 捕获 ] ( 形参 ) -> ret { 函数体 };
```

式子会返回一个匿名、右值的闭包。

当编译器可以推导返回值类型的时候（包括无返回值时），可以不写`-> ret`。

底层逻辑其实就是仿函数，即编译器自动生成一个重载了`operater()`的类。

#### 捕获
在调用lambda表达式时不需要把捕获中的变量当成参数输入，就可以把它们传进去（相当于在定义时而非在调用时传入变量）。

```txt
[]：默认不捕获任何变量；
[=]：默认以复制捕获所有变量；
[&]：默认以引用捕获所有变量；
[x]：仅以复制捕获x，其它变量不捕获；
[x…]：以包展开方式复制捕获参数包变量；
[&x]：仅以引用捕获x，其它变量不捕获；
[&x…]：以包展开方式引用捕获参数包变量；
[=, &x]：默认以复制捕获所有变量，但是x是例外，通过引用捕获；
[&, x]：默认以引用捕获所有变量，但是x是例外，通过复制捕获；
[this]：通过引用捕获当前对象（其实是复制指针）；
[*this]：通过复制方式捕获当前对象；
```

### std::function （包装器）

`function<int(int, int) >`可以统一所有的函数类型，它可以是函数指针、仿函数和Lambda表达式。

使用样例：作为map的键的类型，这样就可以用Lambda表达式来传值。

### std::bind

将函数包装器与参数进行绑定（也可使用`std::placeholders`进行缺省），返回类型也是包装器，可以直接调用。

```cpp
#include <iostream>
#include <functional>
​
void func(int i)
{
    std::cout << "i = " << i << std::endl;
}
​
int main()
{
    auto f1 = std::bind(func, 222);
    f1();
    auto f2 = std::bind(func, std::placeholders::_1);
    f2(333);
    return 0;
}

```

## Sort

重载`operator<`即可重建该对象的排序规则。

## 随机数

```cpp
#include <random>

std::random_device rd;  
auto gen = std::default_random_engine(rd());  
std::uniform_int_distribution<int> dis(0,10);  
  
for (int i=0; i<10; ++i) {  
    std::cout<<"rand: " << dis(gen) << " ";  
}
```

# 计算时间间隔

```cpp
#include <chrono> 

auto start = std::chrono::system_clock::now(); 
//do something 
auto end = std::chrono::system_clock::now(); 
auto elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(end - start); 
std::cout << elapsed.count() <<"ms" << '\n';
```


# 问题与技巧

## 两个类互相依赖

类A里面需要B，类B里面需要A。可以在B.h里面别`include "A.h"`，而是前向声明 `class A`；A.h则正常地`include "B.h"`。

## 其他

>以下所有内容都可以放入头文件中，并且可以多次包含在不同的翻译单元中。   类类型（第9节），枚举类型（7.2），带有外部链接的内联函数（7.1.2），类模板（第14节），非静态函数模板（14.5.5）可以有多个定义，类模板的静态数据成员（14.5.1.3），类模板的成员函数（14.5.1.1），或者在程序中未指定某些模板参数（14.7,14.5.4）的模板特化，前提是每个模板定义出现在不同的翻译单元中，并且定义满足以下要求。 要求基本上归结为每个定义必须相同。请注意，如果您的枚举类型本身没有名称，那么该规则不会涵盖它。另一个翻译单元中的每个定义都定义了一个新的枚举类型，并且不会相互冲突。 将它们放入标题是一个好地方，如果它应该是公开的。将它们放在实现文件中是一个好地方，如果它应该是对该单个文件的私有。在后一种情况下，要么将它们放入未命名的命名空间，要么将它们命名为未命名（就像枚举示例中的情况一样），以便它不会与具有相同名称的另一个枚举冲突。

# CMake

## CMakeList

### 复杂目录项目构建

`add_library`会将当前目录编译为库供其他项目使用，因此必须要自己就能编译通过，所以不太适合用来将一个项目的多个目录进行整合。

利用`target_sources`，可以很方便地管理各子目录。

父目录CMakeList示例：

```cmake
#建立项目，可以使用add_executable
add_library(MyPro MainController.cpp)  
#指定该项目的include目录为./include。只用include_directories会导致该项设置无法传递到子目录
target_include_directories(MyPro PUBLIC include)   

#添加各子目录。要写在创建项目的后面 
add_subdirectory(ui)  
add_subdirectory(util)
```

子目录CMakeList示例：

```cmake
#将子目录的cpp文件加载到项目中。
#若父目录里面的add_subdirectory写在建立项目之前，则会报重复定义错误。也就是说target_sources里面的target（第一项）匹配不到的话会自己建立一个项目
target_sources(
	MyPro  
    PUBLIC        
	    a.cpp
	    b.cpp
)
```


