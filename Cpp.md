
# 链接

[C++11中关键字 - 知乎](https://zhuanlan.zhihu.com/p/157523014)

[C++STL中的常用容器总结\_stl容器总结\_SeeDoubleU的博客-CSDN博客](https://blog.csdn.net/SeeDoubleU/article/details/124507029)

[C++模板元编程详细教程（之三）\_c++模板元编程实战\_borehole打洞哥的博客-CSDN博客](https://blog.csdn.net/fl2011sx/article/details/128314495)
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

类中不能存在不知道大小的成员。前向声明并不能提供某个类的大小，但可以取它的指针，因为指针的大小是固定的。

## STL

STL默认以vector为容器、以`operator<`为比较方式。
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
## sort函数

默认从小到大。

重载`operator<`即可重建该对象的排序规则。

也可以添加比较函数：
```cpp
//比较函数，使其从大到小排
int cmp(const int &a, const int &b) { 
	if(a > b) { 
		return true; 
	} 
	return false; 
}
sort(arr, arr+len, cmp);
```


## stable_sort函数（稳定排序）

和sort函数用法一样，但是稳定。

## less greater 比较类

在头文件`functional`中。

如int的比较类就是`greater<int>`，表示取大（`>`）。将其对象传入sort就是从大到小排序：
```cpp
sort(arr, arr+len, greater<int>());
```

## priority_queue 优先级队列

就是个堆。

优先级：排前面（上面）的优先级高。在sort中排在后面的，在priority_queue中就排在前面，也就是优先级高。

默认为大顶堆。

在头文件`queue`中。

```cpp
priority_queue<类型,容器,cmp>

//大顶堆（降序）
priority_queue<int> big_heap;
priority_queue<int,vector<int>,less<int>> big_heap2;

//小顶堆（升序）
priority_queue<int,vector<int>,greater<int>> small_heap;
```

| 方法      | 功能                             |
| --------- | -------------------------------- |
| push()    | 在优先级队列末尾插入新元素并调整 |
| emplace() | 在优先级队列末尾构造新元素并调整 |
| pop()     | 删除优先级最高的元素             |
| top()     | 获取优先级最高的元素             |
| size()    | 获取优先级队列的大小             |
| empty()   | 验证队列是否为空                 |
|   swap()        | 交换两个优先级队列的内容                                 |

## 字符串
### String

| 方法      | 功能                                                                                                                 |
| --------- | -------------------------------------------------------------------------------------------------------------------- |
| Operators | 可以用 == ， >， <， >=， <=， and !=比较字符串。 可以用 + 或者 += 操作符连接两个字符串， 并且可以用[]获取特定的字符 |
| append()  | 在字符串的末尾添加文本（就是+=）                                                                                     |
| compare() | 比较两个字符串                                                                                                       |
| empty()   | 如果字符串为空，返回真                                                                                               |
| erase()   | 删除字符                                                                                                             |
| insert()  | 插入字符，也可以插入另一个字符串的子串                                                                               |
| length()  | 返回字符串的长度                                                                                                     |
| replace() | 替换字符                                                                                                             |
| size()    | 返回字符串中字符的数量（结果等价于length）                                                                           |
| substr()  | 返回某个子字符串                                                                                                     |
| swap()    | 交换两个字符串的内容                                                                                                 |
|  c_str()         |      返回字符数组（const char\*）                                                                                                                |

```cpp
//erase
string s("1234567890123456789012345");//总共25个字符
cout << s << endl;
cout << s.erase(10, 6) << endl;//从下标第10的字符开始删除，一共删除6个。
cout << s.erase(10) << endl;//删除下标为10的字符及之后的 ==保留下标小于10的字符
cout << s.erase();//清空字符串
```

```cpp
//String转字符数组
char c[20];
string s="1234";
strcpy(c,s.c_str());

//数组字符转String
s=c;
```

#### find()

```cpp
string s="this is buaa";
	
int sfy=s.find("is");
cout<< sfy <<" ";
cout<< s.find("is")<<" ";
cout<<"\n";

int sfn=s.find("isj");
cout<< sfn <<" ";
cout<< s.find("isj")<<" ";
cout<<s.npos<<" "<<std::string::npos<<" ";
cout<<(s.find("isj")==s.npos)<<" ";
cout<<(s.find("isj")==-1)<<" ";
cout<<"\n";
```

输出：
```cpp
1
2 2
-1 4294967295 4294967295 4294967295 1 1
```

找到了就返回起始下标值，找不到就返回`std::string::npos`，等于-1。
### C风格字符串操作

都是对char数组的操作（参数为char*）。

- strcpy(p, p1) 复制字符串 
- strncpy(p, p1, n) 复制指定长度字符串 
- strcat(p, p1) 附加字符串 
- strncat(p, p1, n) 附加指定长度字符串 
- strlen(p) 取字符串长度 
- strcmp(p, p1) 比较字符串 
- strcasecmp(p, p1)忽略大小写比较字符串 
- strncmp(p, p1, n) 比较指定长度字符串 
- strchr(p, c) 在字符串中查找指定字符 
- strrchr(p, c) 在字符串中反向查找 
- strstr(p, p1) 查找字符串 
- strpbrk(p, p1) 以目标字符串的所有字符作为集合，在当前字符串查找该集合的任一元素 
- strspn(p, p1) 以目标字符串的所有字符作为集合，在当前字符串查找不属于该集合的任一元素的偏移 
- strcspn(p, p1) 以目标字符串的所有字符作为集合，在当前字符串查找属于该集合的任一元素的偏移

### char字符检查函数

参数都为char。

- isalpha() 检查是否为字母字符 
- isupper() 检查是否为大写字母字符 
- islower() 检查是否为小写字母字符 
- isdigit() 检查是否为数字 
- isxdigit() 检查是否为十六进制数字表示的有效字符 
- isspace() 检查是否为空格类型字符 
- iscntrl() 检查是否为控制字符 
- ispunct() 检查是否为标点符号 
- isalnum() 检查是否为字母和数字 
- isprint() 检查是否是可打印字符 
- isgraph() 检查是否是图形字符，等效于 isalnum() | ispunct()

## iterator迭代器

### distance()

返回两个迭代器之间的元素个数，类似于两者之差。

```cpp
#include <iostream> // std::cout
#include <iterator> // std::distance
#include <list> // std::list
using namespace std;

int main() {
	//创建一个空 list 容器
	list<int> mylist;
	//向空 list 容器中添加元素 0~9
	for (int i = 0; i < 10; i++) {
		mylist.push_back(i);
	}
	//指定 2 个双向迭代器，用于执行某个区间
	list<int>::iterator first = mylist.begin();//指向元素 0
	list<int>::iterator last = mylist.end();//指向元素 9 之后的位置
	//获取 [first,last) 范围内包含元素的个数
	cout << "distance() = " << distance(first, last);
	return 0;
}
```

输出：

```txt
distance() = 10
```

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

## 模板

模板代码本身什么都不能做，它需要被使用后知道要生成哪些实例。并且所有实例都自带inline，不会重复定义。

（在预编译之后）声明和实现必须在同一个文件里面。如果把声明放在头文件、把实现放在cpp文件，则另一个cpp文件只会include头文件而会忽视cpp文件，于是只能通过模板函数声明推导出函数声明语句，而不会推导实现语句，使得实例（如`f<int>()`)会找不到实现。
### 函数模板
#### 基本使用


```cpp
template <typename T> 
void f(const T &t); 

template <typename T> 
void f(const T &t) {} // 当然，文件内部没有声明依赖关系的时候，声明和实现可以合并
```

### 特化

有时候对模板类型的操作不能通用，需要单独特殊处理，则可以用特化。

```cpp
#include <cstring>

template <typename T>
void add(T &t1, const T &t2) {
  t1 += t2;
}

template <> // 模板特化也要用模板前缀，但由于已经特化了，所以参数为空
void add<char *>(char *&t1, char *const &t2) { // 特化要指定模板参数，模板体中也要使用具体的类型
  std::strcat(t1, t2);
}

void Demo() {
  int a = 1, b = 3;
  add(a, b); // add<int>是通过通用模板生成的，因此本质是a += b，符合预期

  char c1[16] = "abc";
  char c2[] = "123";

  add(c1, c2); // add<char *>有定义特化，所以直接调用特化函数，因此本质是strcat(c1, c2)，符合预期
}
```

### 全特化

确定了所有模板参数后即为全特化。全特化已经是个实例，因此要当成普通函数/类来对待。如果要和模板函数/类一起写在头文件的话，要提防重复定义，如手动加上inline。

# 问题、技巧、解决方案

## 两个类互相依赖

类A里面需要B，类B里面需要A。可以在B.h里面别`include "A.h"`，而是前向声明 `class A`；A.h则正常地`include "B.h"`。

## 获取数组长度

```cpp
int len=sizeof(arr)/sizeof(arr[0]);
```

## 计算时间间隔

```cpp
#include <chrono> 

auto start = std::chrono::system_clock::now(); 
//do something 
auto end = std::chrono::system_clock::now(); 
auto elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(end - start); 
std::cout << elapsed.count() <<"ms" << '\n';
```

## 头文件放什么

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

# 编译过程

## 1、预处理（Preprocess）

`.i`: Intermediate file

```bash
gcc -E test.c -o test.i
```

处理`#define`（字符串替换）、`#include`（复制粘贴）、条件编译等，并把注释给去掉。
## 2、编译（Compilation）

`.s`: Assembly file

```bash
gcc -S test.i -o test.s
```

词法语法语义分析，生成汇编代码文件。

## 3、汇编（Assembly）

`.o`: object file

```bash
gcc -c test.s -o test.o
```

把汇编文件汇编为二进制机器码。以机器码的形式包含了编译单元里所有的函数和数据、导出符号表、未解决符号表、地址重定向表等。

每个cpp文件都是个编译单元（Translation Unit），生成一个`.o`目标文件。

## 4、链接（Linking）

```bash
gcc test.o -o test.exe
```

将目标文件与启动文件、其他库文件、其他目标文件进行链接。

可见性：
- 内部链接(internal linkage): 只能在该编译单元使用，如：
	- 常量（const）
	- 内联函数（inline）
	- 静态函数（static）
	- 类（class）
- 外部链接(external linkage): 全局可引用，如：
	- 函数
	- extern声明的变量

全局变量必须唯一不得重名，也因此只能被定义一次。多个文件用同一个全局变量时使用extern声明。

static可以理解成“只能自己用的全局变量”。因此多文件开发时尽量避免全局变量，尽量加上static。


# 编译命令

- `-o xxx` 输出文件的名字 
- `-std=c++11` 切换为cpp11标准

