
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

# 面向对象

[第13章-cpp类继承\_cpp 继承\_itzyjr的博客-CSDN博客](https://blog.csdn.net/itzyjr/article/details/103424450)

## 三大特征

1. **封装（Encapsulation）**：封装隐藏对象的属性，并且外界只能通过对外提供的接口进行访问。
2. **继承（Inheritance）**：子类可以复用父类的成员和方法，并且可以在现有代码的基础上进行功能扩展。
3. **多态（Polymorphism）**：
	1. 编译时多态：同名函数有不同的参数列表，则能够在编译时识别出调用的是哪个函数。处理函数**重载**。
	2. 运行时多态：运行时，根据传入的对象的类型决定调用哪个类的某个函数。这个函数是虚函数，被子类重写，函数名、参数列表完全一样。处理函数**重写**。

## 访问控制

同一个类的两个不同对象是可以互相访问private成员的。因为访问级别是编译时概念而非运行时。对象a和b都为Class Cls，它们都能知道Cls下有私有成员x，则a里面就可以访问b.x。

对于外部来说，protected的行为与private相似；但对于派生类来说，protected的行为与public相似。
## 向上兼容

可以用基类指针或引用指向派生类对象，如`Base *b = new Derived;`或`Derived d; Base &b = d;`。

注意，不能用`Base b = Derived();`，这会导致对象切割，本质还是在存Base。

若`Base *b = new Derived;`，则`*b`（对指针解引用）返回的是Base类型，似乎会发生对象切割。

可以把派生类指针赋值给基类指针。

[对象切割](Cpp.md#对象切割（object%20slicing）)

## 调用基类构造函数

**派生类在构造时必须调用基类的构造函数**. 

如果基类有默认构造函数(参数值全有默认值的构造函数或无参构造函数), 那么子类的构造函数就会默认调用此函数, 这称为**隐式调用**.

其他情况下必须**显式调用**. 在<u>初始化列表</u>的**任意位置**调用基类构造函数即可:
```cpp
Student::Student(char *name, int age, float score): 
m_score(score), People(name, age){ }
```

**总是**会先调用基类构造函数再干其他所有事情(与其在初始化列表中的顺序无关).

> [!tip]
> 如果基类没有默认构造函数, 那么派生类就不得不每次都进行显式调用.

> [!info]
> 如果想直接使用父类构造函数的话, 可以使用[继承构造函数](Cpp/Cpp.md#继承构造函数).

## 重写 & virtual

虚函数可以被重写。被重写的虚函数自动成为虚函数，但一般最好手动加virtual。

使用了virtual之后程序将根据**对象类型**而不是引用或指针的类型来选择方法版本，或者说会挑选该方法的**可获取的最“后代”的**版本，即**动态联编**；否则直接根据**指针类型**去判断要调用哪个函数，即**静态联编**。

这是**运行时多态**。

```cpp
class Base {
public:
    void func() { cout << "Base::func\n"; }
    virtual void vfunc() { cout << "Base::vfunc\n"; }
};

class Derived : public Base {
public:
    void func() { cout << "Derived::func\n"; }
    void vfunc() override { cout << "Derived::vfunc\n"; }
};

Base* b = new Derived;
b->func();  // Outputs "Base::func" 由指针类型决定
b->vfunc(); // Outputs "Derived::vfunc" 由对象类型决定
```

### override & final

写在函数末尾, `override`声明当前函数为重写, 使编译器去检查这是否真的发生了重写. 而`final`声明当前虚函数不可被重写.


### 纯虚函数 & 抽象基类

```cpp
virtual T func(args) = 0;
```

当类声明中包含纯虚函数时，不能创建该类的对象。包含纯虚函数的类只用作基类。

包含纯虚函数的类成为**抽象基类（abstract base class，ABC）**。ABC的一个目的是实现接口约定。

`= 0`的作用仅仅是说明**可以在此类中不提供其实现**，但是依然是**可以进行实现**的。因此，一个拥有纯虚函数func的类作为一个不可实例化的基类，而其所有派生类的func代码完全一样时，不妨就在基类中实现它。

### 重写注意点

虚函数被重写时，会把所有名字一样的都给覆盖，即使参数列表和返回值不一样。若不一样，编译器可能发出警告，但不会编译失败。因此，如果基类有三个同名虚函数，则派生类最好要么都不重写，要么都重写。如下所示：

```cpp
class Dwelling {
public:
    // 三个重载的showperks()
    virtual void showperks(int a) const;
    virtual void showperks(double x) const;
    virtual void showperks() const;
    ...
};
class Hovel : public Dwelling {
public:
    // 三个重新定义的showperks()
    virtual void showperks(int a) const;
    virtual void showperks(double x) const;
    virtual void showperks() const;
    ...
};
```

另外，在设计上，允许在重写时仅对返回值进行修改：如果返回类型是基类引用或指针，则可以修改为指向派生类的引用或指针。这种特性被称为**返回类型协变（covariance of return type）**，因为允许返回类型随类类型的变化而变化。如下所示：

```cpp
class Dwelling {
public:
    // a base method
    virtual Dwelling &build(int n);
    ...
};
class Hovel : public Dwelling {
public:
    // 一个派生类方法，返回类型协变
    virtual Hovel &build(int n); // 相同的参数特征标
    ...
};
```

### 具体函数

**析构函数**一般要设为虚函数，因为若类被继承，则新的成员变量很可能需要特殊的析构函数去管理；有虚析构函数后，任意派生类都会先执行最子类的析构函数，然后自动执行父类的析构函数。只有析构函数有全自动的调用基类的机制。

构造函数不设为虚函数，因为派生类不继承基类的构造函数，而是隐式调用默认构造函数或显式调用某个构造函数。

友元不能是虚函数，因为友元不是类成员，而**只有成员才能是虚函数**。

### 虚函数处理原理

给每个对象添加一个隐藏成员是一个指向函数地址数组的指针，且尽可能靠前。这种数组称为虚函数表（virtual function table，vtbl）。vtbl和类进行绑定，每个对象的指针隐藏指针都指向同一个vtbl。

虚函数表中存储了为类对象进行声明的虚函数的地址。例如，基类对象包含一个指针，该指针指向基类中所有虚函数的地址表。派生类对象将包含一个指向另一个属于自己的vtbl的指针。如果派生类提供了虚函数的新定义，该虚函数表将保存新函数的地址；如果派生类没有重新定义虚函数，该vtbl将保存函数原始版本的地址。如果派生类定义了新的虚函数，则该函数的地址也将被添加到vtbl中。

不管一个派生类被什么指针指着，那个指针都能访问到这个唯一的隐藏指针，也就定位到了基于对象真实类型的虚函数表。

![500](assets/uTools_1689420178949.png)


## 重载

同名方法不同参数列表，即可根据参数在编译期自动选择哪个方法。

这是**编译时多态**。

先把所有同名函数作为候选者，然后匹配：
1. 精确匹配。
2. 通过默认参数能够匹配。
3. 通过默认类型转换能够匹配。

若出现二义性或者完全找不到，则报错。

```cpp
// Function overload #1
void display(int num) {
    std::cout << "Displaying int: " << num << std::endl;
}

// Function overload #2
void display(double num) {
    std::cout << "Displaying double: " << num << std::endl;
}

// Function overload #3
void display(char const *str) {
    std::cout << "Displaying string: " << str << std::endl;
}

int main() {
    display(5); // Calls function overload #1
    display(5.5); // Calls function overload #2
    display("Hello"); // Calls function overload #3

    return 0;
}
```

## 对象切割（object slicing）

把子类对象赋值给父类对象（形如`Father f = Son();`）时，会把子的多余成员变量给切割掉。

```cpp
class Father {
public:
    int a;
};

class Son : public Father {
public:
    int b;
};

int main() {
    Son s;
    s.a = 1;
    s.b = 2;

    Father f = s;
    // f now only has 'a', 'b' is sliced off
}
```

## 确定运行时对象的类

基类指针指向派生类对象时，我们有可能要根据派生类的真正类型来判断要调用什么方法，或想要调用派生类有而基类没有的方法。

### dynamic_cast

可以实现**指针安全转换**。`dynamic_cast<D*>(B)`在B能转成D时返回转换后的指针，否则返回nullptr。

转换前后的指针指向同一个对象。

```cpp
Base *bp = /* ... */;
Derived1 *d1 = dynamic_cast<Derived1*>(bp);
if (d1 != nullptr) {
    // bp points to an object of type Derived1
} else {
    Derived2 *d2 = dynamic_cast<Derived2*>(bp);
    if (d2 != nullptr) {
        // bp points to an object of type Derived2
    }
}
```

### 利用virtual性质

若`Base *b=new Derived1();`且Base中有virtual函数func、Derived1中重写了func，则`b->func()`会调用Derived1的函数。利用这个性质，可以在Derived::func中反馈“这个指针b指向的是Derived1类而不是Derived2或者Base”。

**单分派（single dispatch）、双分派（double dispatch）**：传入一个（两个）参数，根据参数的运行时类型确定该使用哪个方法处理它们。

双分派例子：父类Shape，有三个子类Circle、Square、Triangle。要求定义一个函数intersect(Shape* s1, Shape* s2)，使得输入两个子类对象之后，能判断两个对象是否属于同一个类。

#### 解法1

令Shape包含：

```cpp
virtual bool intersect_sec(Shape* const s) = 0;
virtual bool judgeIntersect(Circle& s) = 0;
virtual bool judgeIntersect(Square& s) = 0;
virtual bool judgeIntersect(Triangle& s) = 0;
```

子类将实现它们。每个judgeIntersect函数都在参数类型与自己类型一样时返回true，否则返回false。比如`Square::judgeIntersect(Square& s)`恒返回true，其他两个返回false。

intersect函数先调用`s1->intersect_sec(s2)`，这样就调用了s1内的intersect_sec，也就相当于知道了s1的类型。而每个子类里面的intersect_sec函数都是这样实现的：

```cpp
bool intersect_sec(Shape* const s){
	return s->judgeIntersect(*this);
}
```

`*this`将s1自己以确切的类型传给s2，使用s2的judgeIntersect很快就判断出是否是同类型了。

#### 解法2

先化为两个单分派问题，获取人为规定的类型标识，然后进行判断即可。

```cpp
enum ShapeType { CIRCLE, SQUARE, TRIANGLE };

class Shape {
public:
    virtual ~Shape() {}
    virtual ShapeType getType() const = 0; // Pure virtual function
};

class Circle : public Shape {
public:
    ShapeType getType() const override { return CIRCLE; }
};

class Square : public Shape {
public:
    ShapeType getType() const override { return SQUARE; }
};

class Triangle : public Shape {
public:
    ShapeType getType() const override { return TRIANGLE; }
};

bool intersect(const Shape* s1, const Shape* s2) {
    return s1->getType() == s2->getType();
}
```

## 继承访问控制

继承时可以指定访问等级，使得继承后基类成员的访问等级不得高于继承访问等级。

| 继承类型 \\ 存取模式 | public    | protected | private |
| ------------ | --------- | --------- | ------- |
| public       | public    | protected | private |
| protected    | protected | protected | private |
| private      | private   | private   | private |


## 调用基类函数/运算符

派生类中，如果想调用基类的函数，但这个函数在派生类中被重写了或正在重写中，则要显式地指定基类函数，如`Base::func()`。如果是运算符的话，则用函数表示法写出，如`Base::operator=(arg)`。

```cpp
hasDMA & hasDMA::operator=(const hasDMA & hs) {
    if (this == &hs) 
        return *this;
    baseDMA::operator=(hs); // 用[函数表示法]赋值基类部分，这样才不会形成递归调用！
    delete [] style; // 为新style作准备
    style = new char[std::strlen(hs.style) + 1];
    std::strcpy(style, hs.style);
    return *this;
}
```

## 类间互相依赖问题

首先明确，两个文件是不能互相include的，编译也是有执行顺序的（只有类内部才有类似拉链回填的机制）。

#### 成员变量类型相互依赖

两个类的其中一个成员变量的类型为对方。编译时只能先编译其中一个类，此时它就不知道另一个类，导致编译失败。

使用前向声明可以解决一部分问题，因为声明是廉价的，声明之后就可以得知存在另一个类，即使不知道它有什么内容。

但是，一个类编译完成时，其大小是确定的。但是此时不知道另一个类的内容，也就不知道另一个类的大小，使得总大小不确定。将类型换成指针即可，因为指针虽然可以指向各种不同类型，但其本身的大小是固定的（如64bit）。


```cpp
class ClassB; // Forward declaration

class ClassA {
    private:
        ClassB* classBInstance; // Use pointer instead of object
};

class ClassB {
    private:
        ClassA* classAInstance; // Use pointer instead of object
};
```

#### 静态变量类型相互依赖

**警告：这个疑似没什么用**

两个类各有一个静态成员变量的类型为对方。由于static是必须立刻初始化的，因此我们无法做到真正初始化它们。

可以都初始化为nullptr，并在第一次访问时真正初始化。

```cpp
class ClassB; // Forward declaration

class ClassA {
    private:
        static ClassB* classBInstance;
    public:
        static ClassB* getClassBInstance();
};

class ClassB {
    private:
        static ClassA* classAInstance;
    public:
        static ClassA* getClassAInstance();
};

// Implementations
ClassB* ClassA::classBInstance = nullptr;
ClassA* ClassB::classAInstance = nullptr;

ClassB* ClassA::getClassBInstance() {
    if (classBInstance == nullptr) {
        classBInstance = new ClassB();
    }
    return classBInstance;
}

ClassA* ClassB::getClassAInstance() {
    if (classAInstance == nullptr) {
        classAInstance = new ClassA();
    }
    return classAInstance;
}
```



# 数据机制

## Data Category

[Value categories - cppreference.com](https://en.cppreference.com/w/cpp/language/value_category)

Each C++ [expression](https://en.cppreference.com/w/cpp/language/expressions "cpp/language/expressions") (an operator with its <u>operands</u>, a <u>literal</u>, a <u>variable name</u>, etc.) is characterized by two independent properties: a **type** and a **value category**. Each expression has some non-reference type, and each expression belongs to exactly one of the three <u>primary value categories</u>: **prvalue**, **xvalue**, and **lvalue**.

- **glvalue** (“generalized” lvalue) (**范左值**): 是一个表达式，其求值可以确定一个对象或函数的 identity。要么是 lvalue 要么是 xvalue。
- **prvalue** (“pure” rvalue) (**纯右值**): 是一个表达式，其求值可以计算运算符的一个操作数（没有result object），或初始化一个对象（有result object）。
- **xvalue** (“eXpiring” value) (**将亡值**): 是一个 <u>glvalue</u>，可以提供出一个对象，该对象的资源可重复使用（通常是因为它接近其生存期的末尾）。
- **lvalue** (**左值**): 是*非 xvalue* 的 <u>glvalue</u>。
- **rvalue** (**右值**): 是一个 <u>prvalue</u> 或 <u>xvalue</u>。要么是prvalue 要么是xvalue。

![400](assets/value_categories.png)

> [!tip]
> glvalue是有名字的东西，prvalue是没名字的东西，xvalue是有名字但其名字即将消失的东西。

## 引用

[Reference declaration - cppreference.com](https://en.cppreference.com/w/cpp/language/reference)

- 左值引用 (lvalue reference)，`T&`，只能绑定左值
- 右值引用 (rvalue reference)，`T&&`，只能绑定右值
- 常量左值引用，`const T&`,既可以绑定左值，又可以绑定右值，但是不能对其进行修改

> [!notice]
> 不管什么引用, 都是lvalue.

在`int a = std::move(b)`中, `std::move(b)`这个函数调用语句是一个**xvalue**, 但左边的`a`(**具名右值引用**)是个**lvalue**.

由于具名右值引用被视为左值，因此我们要把它转回右值引用时就需要再用`move`(再次交出左值)或`forward`(具名赋值的逆操作)。

> [!tip]
> cpp中的引用是通过等号来实现的, 如`int &a = b`和`int &&a = 1`, 而函数的传参也相当于进行了等号操作. 这有时候看起来像是特别奇怪的隐式类型转换.

使用`static_cast`也能产生正常使用的左值引用:
```cpp
int i = 1;
auto &a = static_cast<int &>(i);
a = 2;
std::cout << i << std::endl; // 2
```

### 从左值转化成右值引用

```cpp
int i = 0;
int &&k1 = i; //编译失败
int &&k2 = static_cast<int&&>(i); //编译成功
```

### 引用折叠 (Reference Collapsing)

- X& &、X& &&、X&& &都折叠成X&。
- X&& &&折叠为X&&。

```cpp
typedef int&  lref;
typedef int&& rref;
int n;
 
lref&  r1 = n; // int&
lref&& r2 = n; // int&
rref&  r3 = n; // int&
rref&& r4 = 1; // int&&
```

> [!note]
> - 只要带了引用, 就永远不会被折叠成非引用.
> - 带有左值引用的会一直是左值引用, <u>只有纯粹的右值引用的叠加才是右值引用</u>.

### 万能引用 (universal reference)

由于引用折叠机制, 一个引用带上`&&`之后, 得到的引用类别由引用本身决定, 同时带上引用之后永远不会被推导为非引用, 因此在模板函数中使用`T&&`可以接收任何引用：
```cpp
template<typename T> 
void f(T&&);

int i = 42;
f(i); // 实例化`f(int &)`
f(std::move(i)); // 实例化`f(int &&)`
```

万能引用接收右值参数的时候，在函数体内(形参)就会变成左值（具名右值引用）。使用`std::forward<T>(x)`可以让其回归右值引用，且对左值情况没有任何影响，从而正确地触发拷贝操作/移动操作。

`auto &&`也能做到类似的事情. 于是可以在for循环中写成:
```cpp
for(auto &&elem: vec) {...}
```



## std::move

[【Modern C++】深入理解移动语义](https://mp.weixin.qq.com/s/GYn7g073itjFVg0OupWbVw)

```cpp
//C++11 std::move的实现 
template<typename T> 
typename remove_reference<T>::type&& move(T&& param) { 
	using ReturnType = typename remove_reference<T>::type&&; 
	return static_cast<ReturnType>(param); 
}
```

因此`std::move(arg)`等价于:
```cpp
static_cast<std::remove_reference<decltype(arg)>::type&&>(arg)
```

>std::move is used to indicate that an object t may be "moved from", i.e. allowing the efficient transfer of resources from t to another object. In particular, std::move produces an **xvalue** expression that identifies its argument t. It is exactly equivalent to a static_cast to an rvalue reference type.

转换的过程不存在复制，也不存在数据剥夺，在运行时什么都没做，只是个单纯的数据转换。

若将move结果赋值给一个变量（左值），则会调用其移动赋值函数。

**move语义**：将左值对应的内存的所有权进行转交，原对象在move后不可使用。移动操作应当以move语义为目标。语义仅仅是语义, 没有额外的魔法保证.

因此，移动操作一般要做到：将原对象的数据移至自己的手下，并让原对象失去数据。

**自定义的类不会自动带有真正的、完整的move语义**，编译器自动生成的移动构造函数只是可以调用其各个成员变量的移动构造函数。因此，对于自定义类来说，想实现移动语义必须实现非常具体的移动构造函数。

移动构造函数在删除原对象对数据的所有权时，**要那些指针设为nullptr**。因为即使一个左值a被move到另一个左值b，a本身是不会被消除的，**在a的生命周期结束后会自动调用其析构函数**，从而可能触发野指针错误或者对一个数据进行多次析构（a和b都会调用析构函数）。

## std::forward 完美转发 

[std::forward - cppreference.com](https://en.cppreference.com/w/cpp/utility/forward)

`forward`能把一个函数形参还原成它在此函数被调用时传入的值的data category，从而触发不同的函数。

它能转发[forwarding references](https://en.cppreference.com/w/cpp/language/reference#Forwarding_references). 这些东西在形式上是：
```cpp
[const] T &[&]
即：
const T &
T &
const T &&
T &&
```

> 和move一样在<u>运行时</u>没做任何改动，仅仅是类型转换。

```cpp
#include <iostream>
#include <memory>
#include <utility>
 
struct A {
    A(int&& n) { std::cout << "rvalue overload, n=" << n << "\n"; }
    A(int& n)  { std::cout << "lvalue overload, n=" << n << "\n"; }
};
 
class B {
public:
    template<class T1, class T2, class T3>
    B(T1&& t1, T2&& t2, T3&& t3) :
        a1_{std::forward<T1>(t1)},
        a2_{std::forward<T2>(t2)},
        a3_{std::forward<T3>(t3)}
    {
    }
 
private:
    A a1_, a2_, a3_;
};
 
template<class T, class U>
std::unique_ptr<T> make_unique1(U&& u)
{
    return std::unique_ptr<T>(new T(std::forward<U>(u)));
}
 
template<class T, class... U>
std::unique_ptr<T> make_unique2(U&&... u)
{
    return std::unique_ptr<T>(new T(std::forward<U>(u)...));
}
 
int main()
{   
    auto p1 = make_unique1<A>(2); // 右值
    int i = 1;
    auto p2 = make_unique1<A>(i); // 左值
 
    std::cout << "B\n";
    auto t = make_unique2<B>(2, i, 3);
}
```

输出：

```txt
rvalue overload, n=2
lvalue overload, n=1
B
rvalue overload, n=2
lvalue overload, n=1
rvalue overload, n=3
```

### forward_as_tuple

[std::forward\_as\_tuple - cppreference.com](https://en.cppreference.com/w/cpp/utility/tuple/forward_as_tuple)

将参数组成tuple的形式，并且每个参数都被forward了。可以用于全部完美转发地传参数列表。而且如果参数是临时变量的话，不会将其存储（即不会延长生命周期）；如果不在表达式结束前使用完毕，则会造成悬垂引用。

# 类与对象

## 类成员变量默认初始化


可以在类成员函数处直接指定其默认初始化值, 而不需要全都写在初始化列表里面. 优先级低于初始化列表, 即会被初始化列表指定的值覆盖.

```cpp
class X {
  private:
    int a = 1;
    double b{1.};
};
```

## 类函数性质

![](assets/uTools_1689432203441.png)

## 特殊成员函数

- 默认构造函数 `Obj()`
- 析构函数 `~Obj()`
- 拷贝构造函数 `Obj(const Obj&)`
- 拷贝赋值运算符 `Obj& operator=(const Obj&)`
- 移动构造函数 `Obj(Obj&&)`
- 移动赋值运算符 `Obj& operator=(Obj&&)`

特殊成员函数若未被定义，则会由编译器自动生成。

>注意，xx构造函数都叫构造函数，都不能为virtual！

- `Obj o2 = o1;`调用**拷贝构造函数**
- `Obj o2; o2 = o1;`调用**拷贝赋值运算符**
- `Obj o2 = move(o1);`调用**移动构造函数**
- `Obj o2; o2 = move(o1);`调用**移动赋值运算符**
- `Obj o2 = Obj();`调用**移动构造函数**（因为传的是右值）
- `Obj &&o1; Obj o2 = o1;`调用**拷贝构造函数**（因为具名右值引用是左值）


移动相关：
- 只有一个类没有显示定义**拷贝构造函数、赋值运算符以及析构函数**，且类的**每个非静态成员都可以移动**时，编译器才会生成默认的**移动构造函数或者移动赋值运算符**。这是为了防止生成的移动不是开发人员想要的（因为开发人员选择自己管理复制和释放），或存在有问题的移动。
- 如果类中没有提供移动构造函数和移动赋值运算符，且编译器不会生成默认的，那么我们在代码中通过std::move()调用的移动构造或者移动赋值的行为将**被转换为调用拷贝构造或者赋值运算符**。因此对于不可移动的数据类型也可以像可移动的数据类型一样写代码，会自动变成复制操作。
- 移动构造函数和移动赋值运算符的生成是**不独立**的，若只实现了其中一个，则编译器**不会**自动生成另一个。这是为了防止移动出现问题。

拷贝相关：
- 如果显式声明了**移动构造函数或移动赋值运算符**，则**拷贝构造函数和拷贝赋值运算符**将被 **隐式删除**。
- 拷贝构造函数和拷贝赋值运算符的生成是**独立**的，若只实现了其中一个，则编译器也**会**自动生成另一个。

基础数据类型都没有实现移动。

### 特殊成员函数编写规范

复制、移动操作应该是无副作用的。而且如果有副作用的话，会因为编译器的优化导致出现意想不到的结果。

构造函数不要抛出异常。

复制操作应该尽可能进行复制，特别是new出来的东西要复制一份一模一样的而不是共用。

移动操作尽可能实现move语义，保证原对象被吃干抹净。

### explicit

如果类`X`的某个构造函数只有`Y y`一个参数, 那么`X x = y;`语句会调用此构造函数. 这是一种**隐式类型转换**, 使得`T`类型可以被直接隐式转为`X`类型.

```cpp
Student S1(22); // 显式构造 
Student S2 = 23; // 隐式构造
```

在这种**单参数的构造函数**的<u>前面</u>加上explicit可以防止在等号表达式中调用此构造函数:
```cpp
struct X {
	explicit X(Y y) {...}
}
```

### delete & default


声明特殊函数后在<u>后面</u>加上
- ` = default`: 生成默认的对应函数(**要求**编译器生成)
- ` = delete`: 删除默认的对应函数(**阻止**编译器生成)

```cpp
struct X {
	f1() = default;
	virtual ~f2() = delete;
	f3(const MyType &);
};
X::f3(const MyType &) = default;
```

### 委托构造函数

**委托构造函数**请求**代理构造函数**代为初始化, 即调用本类型的另一个构造函数.

```cpp
class X {
public:
	X() : X(0,0.) {} // X()是委托, X(int a,double b)是代理
	X(int a) : X(a,0.) {}
	X(double b) : X(0,b) {}
	X(int a,double b): a_(a), b_(b) { CommonInit(); }
private:
	void CommonInit()}
	int a_;
	double b_;
};
```

<u>委托构造函数</u>**无法**在<u>初始化列表</u>里面进行成员初始化, 因此想干的额外的事情都要写进函数体.

执行顺序:
1. 代理构造函数的初始化列表
2. 代理构造函数的主体
3. 委托构造函数的主体

如果代理构造函数完成后, 委托构造函数出现异常, 则会调用该类型的析构函数(因为此时认为代理方已经完成了本类型的构造).

可以委托**模板构造函数**来简化多个构造函数的编写.
```cpp
class X {
	template<class T> X(T first, T last): l_(first,last){
	std::list<int> l_;
public:
	X(std::vector<short>&);
	X(std::deque<int>&);
};

X:X(std::vector<short>& v): X(v.begin(), v.end()){}
X:X(std::deque<int>& v): X(v.begin(), v.end()) {}
```

### 继承构造函数

使用using可以直接把基类的所有构造函数**都搬过来**.
```cpp
class Derived: public Base {
public:
	using Base::Base;
};
```

- <u>不会继承</u>基类的**默认构造函数**和**拷贝构造函数**, 因为在相应的时机默认就<u>必然会调用</u>它们, 因此继承是没有必要的.
- 同时, 不会影响派生类的默认构造函数.
- 如果派生类定义了一个构造函数, 其**签名**与基类的某个构造函数一样, 那么就<u>不会继承</u>那个基类构造函数.
- 继承多个**签名**相同的构造函数时会报<u>二义性错误</u>.

## 完整对象示例

```cpp
class BigObj { 
public: 
	// Constructor
	explicit BigObj(size_t length) : length_(length), data_(new int[length]) { } 
	// Destructor. 
	~BigObj() { 
		if (data_ != NULL) { 
			delete[] data_; 
			length_ = 0; 
		} 
	} 
	
	// 拷贝构造函数 
	BigObj(const BigObj& other) : length_(other.length_), data(new int[other.length_]) { 
		//直接复制数据
		std::copy(other.mData, other.mData + mLength, mData); 
	} 
	
	// 赋值运算符 
	BigObj& operator=(const BigObj& other) { 
		if (this != &other;) { 

			//删去自己原有的数据
			delete[] data_; 

			//重建并复制数据
			data_ = new int[length_]; 
			length_ = other.length_; 
			std::copy(other.data_, other.data_ + length_, data_); 
		} 
		return *this; 
	} 
	
	// 移动构造函数 
	BigObj(BigObj&& other) : data_(nullptr), length_(0) { 

		//获取数据所有权
		data_ = other.data_; 
		length_ = other.length_; 

		//剥夺对方的数据所有权
		other.data_ = nullptr; 
		other.length_ = 0; 
	} 
	
	// 移动赋值运算符 
	BigObj& operator=(BigObj&& other) { 
		if (this != &other;) { 

			//删去自己原有的数据
			delete[] data_; 

			//获取数据所有权
			data_ = other.data_; 
			length_ = other.length_; 

			//剥夺对方的数据所有权
			other.data_ = NULL; 
			other.length_ = 0; 
		} 
		return *this; 
	} 
	
private: 
	size_t length_; 
	int* data_; 
};
```

## 聚合类型

聚合类型的定义是, 它可以是一个<u>普通数组</u>, 或者要<u>同时满足</u>:
- 没有用户提供的构造函数(delete和default都没问题)
- 没有私有和受保护的非静态数据成员
- 没有虚函数

在新的扩展中，如果类存在<u>继承</u>关系，额外满足这些条件的也是聚合类型:
- 必须是公开的基类，不能是私有或者受保护的基类
- 必须是非虚继承

```cpp
// 一个非常简单, 平凡, 赤裸裸的类型
class X {
  public:
    int a;
    double b;
};
```

聚合类型**天然支持**[列表初始化](Cpp/Cpp.md#列表初始化), 相当于直接给每个成员变量按顺序赋值, 例如:
```cpp
class X {
  public:
    int a;
    double b;
};

int main() {
    X v = {2, 5.5};
    std::cout << v.a << " " << v.b << std::endl;
    // Output: 2 5.5
}
```

cpp20中, 也能使用**小括号**代替大括号来初始化(仿佛为我们自动生成了一个完整的构造函数). 与初始化列表的区别是它能<u>支持隐式缩窄转换</u>.
```cpp
X v(2, 5.5);
```


## 初始化

- 直接初始化: 使用括号来初始化, 如 `X v(5);`
- 拷贝初始化: 使用等号来初始化, 如 `X v = 5`
这两个概念没有本质区别, 它们都只会调用符合要求的构造函数(包括拷贝构造函数), 只不过一般会使用`explicit`防止拷贝初始化调用非拷贝函数.

### 指定初始化

```cpp
struct Point {
	int x;
	int y;
};
Point p{ .x = 4, .y = 2 };
```

语法要求:
1. 必须是一个**聚合类型**
2. 数据成员必须是非静态数据成员
3. 数据成员最多只能初始化一次
4. 非静态数据成员的初始化**必须按照声明的顺序**进行
5. 针对联合体中的数据成员只能初始化一次，不能同时指定
6. 不能嵌套指定初始化数据成员
7. 一旦使用指定初始化就不能混用其他方法对数据成员初始化了
8. 禁止对数组使用指定初始化


### 列表初始化

[std::initializer\_list - cppreference.com](https://en.cppreference.com/w/cpp/utility/initializer_list)

- `int x{1};` 是直接初始化
- `int x = {1};` 是拷贝初始化
同样, 这两种写法没有本质区别.

```cpp
std::vector v{ 1, 2, 3 };
std::map<std::string, int> m = { {"bob", 3}, {"Alice", 5} };
```

[聚合类型](Cpp/Cpp.md#聚合类型)**天然支持**列表初始化.

一个非聚合类型想要支持列表初始化, 需要拥有一个参数为`std::initializer_list<T>`的构造函数.

`std::initializer_list`是可迭代的, 因此也能使用增强型for循环来遍历.

```cpp
#include <initializer_list>

class Numbers {
private:
    int* values;
    size_t size;

public:
    // 使用 initializer list 的构造函数
    Numbers(std::initializer_list<int> list) : 
        size(list.size()),
        values(new int[list.size()])
    {
        int i = 0;
        for (auto item : list) {
            values[i] = item;
            i++;
        }
    }
};

int main() {
    // 使用 initializer list 创建对象
    Numbers nums = {1, 2, 3, 4, 5};
    Numbers numsList[] = { {1, 2}, {6, 8} };
    return 0;
}
```

无法支持(或者发出警告)**隐式缩窄转换**(即强表达类型转成弱的), 如`int`转成`char`时:
```cpp
int x = 9;
char c1 = 9; // Correct
char c2 = {9}; // Error or Warning
```

## 运算符

### 三向比较运算符`<=>`

两个数的比较可以有三种结果, 使用**spaceship运算符**`<=>`得到三个结果的其中一个.

器返回值只能和0进行比较(否则报错), 从而反映不同的结果.
```cpp
bool b = 7 <=> 11 < 0; // true
```

对于不同的类型, 其进行的比较以及返回值的类型都是不同的:
- strong_ordering (如int)
	- `std::strong_ordering::less`
	- `std::strong_ordering::equal` (可以互相替换)
	- `std::strong_ordering::greater`
- weak_ordering (如忽略大小写的字符串)
	- `std::weak_ordering::less`
	- `std::weak_ordering::equivalent` (相等但不能互相替换)
	- `std::weak_ordering::greater`
- partial_ordering (如float)
	- `std::partial_ordering::less`
	- `std::partial_ordering::equivalent` (相等但不能互相替换)
	- `std::partial_ordering::greater`
	- `std::partial_ordering::unordered` (不可比)

cpp20规定, 如果一个类型声明了三向比较运算符`<=>`的话, 编译器就自动为其生成 `<`, `>`, `<=`, `>=` 这四种运算符函数. 

> [!info]
> 不自动生成`==`运算符函数是因为一般来说判断是否相等的逻辑会更加简单, 例如判断容器是否等同一般都先比较长度, 长度不同才需要进行遍历.

实现了`<`和`==`的类型会自动实现`<=>`.


### 类型转换运算符

在类型`X`中定义`operator Y()`函数即可定义从`X`到`Y`的类型转换(可以是隐式的). 加上`explicit`之后就只能进行显式类型转换(`static_cast`)了.
```cpp
struct X {
	explicit operator Y() {
		...
	}
}
```


### 默认按值比较

默认比较运算符函数可以是一个参数为`const C&`的非静态成员函数，或则是两个参数为`const C&`或`C`的友元函数。加上`= default`之后, 编译器会生成按值判断相等的函数.

```cpp
struct c {
	int i;
	friend bool operator==(C,C) = default;
};
```


# 库, 语法与特性

## 空指针

空指针指向特定范围的存储空间，这个空间永远不会与物理存储器有映射。

对空指针的delete不会报错。

>delete一个指针后，它不会指向空指针，而是指向非法空间，造成野指针错误。因此可以考虑delete一个指针后将其赋值为nullptr。

**nullptr是有类型的，且仅可以被隐式转化为指针类型。**

NULL可能会被定义为0，若有两个同名不同参的函数（重载），一个的参数类型为指针、另一个的参数类型为int，则传入NULL时可能执行参数类型为int的那一个函数。
## 命名空间

`using namespace xxx`表示使用xxx整个命名空间；而`using xxx:fx`表示使用xxx命名空间下的fx。

## 字面量

有两个**输入输出操作器**可以传递给`cout`:
- 传递`std::hexfloat`可以使浮点数以十六进制格式输出
- 传递`std::hexfloat`可以使浮点数用默认十进制输出

```cpp
#include <iostream>

int main() {
    double float_array[]{
		    0x1.7p+2, 0x1.f4p+9, 0x1.df3b64p-4
	    };
    for (auto elem : float_array) {
        std::cout 
        << std::hexfloat << elem << "=" 
        << std::defaultfloat << elem 
        << std::endl;
    }
}

// Output:
// 0x1.7p+2=5.75      
// 0x1.f4p+9=1000     
// 0x1.df3b64p-4=0.117
```

二进制使用`0b`前缀表示, 如`0b11010011`.

使用单引号作为**整数分隔符**(任意进制都能用):
```cpp
constexpr int x = 123'456;
static_assert(x == 0x1e'240);
static_assert(x == 036'11'00);
static_assert(x == 0b11'110'001'001'000'000);
```

使用`R"(...)"`来表示**原生字符串**`...`, 可以不用转义字符:
```cpp
// 缩进也会被考虑在内
char str[] = R"(abc/\&"%
	tt)";
// Value:
// abc/\&"%
//     tt

// 如果字符串里面有`)"`, 
// 则可以在开头结尾的`"`和 `(`或`)` 之间
// 插入任意标识符, 只有带上标识符的`)mark"`才会被识别成终点
char str[] = R"mark1(bbb)")mark1";
// Value:
// bbb)"
```

可以**自定义字面量**, 使得给其他字面量<u>加后缀</u>就能进行<u>数据处理</u>. 默认是在运行时调用处理函数; 想在编译期完成处理的话就要把处理函数标记为`constexpr`.

```cpp
long double operator"" _mm(long double x) {
    return x;
}

long double operator"" _cm(long double x) {
    return x * 10;
}

long double operator"" _m(long double x) {
    return x * 1000;
}

int main() {
    // height = 30.0
    auto height = 3.0_cm;
    // length = 1230.0
    auto length = 1.23_m;
}
```

## 常量相关
### const

被const修饰的东西为常量，必须初始化。类的常量成员使用初始化表来初始化。

>C语言中const被当做变量来编译，可以不初始化。c++中所有出现const常量名字的地方，都被常量的初始值替换了。

```cpp
//a本身不能被更改
const int a = 20;

//不能通过p间接修改其指向的内存，因为此时修饰着*p，表示解引用后的数据
const int *p;
int const *p;

//不能修改p本身，因为此时直接修饰p，为常指针
int* const p;

//p本身不能改，也不能通过p修改其指向的内存
const int *const p;

//函数返回值上的const都是为了防止出现左值、指针等导致外部对内部可修改
const int* func1();

//不能更改调用此方法的对象。此const可以看做是在修饰this
void Obj::func() const {...} 
```

![400](assets/uTools_1689432530764.png)

### mutable

用于修饰成员变量, 来完全突破const的限制, 包括对象本身为const或使用它的函数为const.

```cpp
struct ST {
  int a;
  mutable int b;
};

const ST st={1,2};
st.a=11;//编译错误
st.b=22;//允许
```

### constexpr

表示一个东西**可以**在编译期完成求值. 不强制要求编译器完成.

编译期就能完成计算的表达式可以使用constexpr, 使其在编译期被处理完.

使用`if constexpr (...)`使得当条件可以在编译期被计算时, 可以只编译符合条件的那部分代码块.

在函数<u>前面</u>加上`constexpr`可以定义常量表达式函数, 其值应当可以在编译期算出来. 也可以定义常量表达式构造函数, 使得对象可以在编译期被构造.

```cpp
constexpr int square(int x) {
	return x * x;
}
```

cpp14对常量表达式函数要求的放松:
- 函数体允许声明变量，除了没有初始化, `static`和`thread_local`变量
- 函数允许出现`if`和`switch`语句，不能使用`go`语句
- 函数允许所有的循环语句，包括`for`,`while`,`do-while`
- 函数可以修改生命周期和常量表达式相同的对象
- 函数的返回值可以声明为`void`
- `constexpr`修饰的成员函数不再具有`const`属性

cpp20对常量表达式要求的放松:
- 允许在constexpr中进行平凡的默认初始化
- 允许在constexprk函数中出现Try-catch
- 允许在constexpr中更改联合类型的有效成员
- 允许dynamic_cast和typeid出现在常量表达式中

在修饰变量的时候, `constexpr` 和 `const`是完全等价的.

### consteval

使用consteval声明立即函数, 保证编译期**必须**计算(constexpr只是"可以").

### constinit

使用constinit声明变量, 可以保证它是**通过常量来初始化**的(它本身不需要是常量). 

这可以用来保证static变量的初始化不依赖于其他static变量, 也就保证其于static变量的初始化顺序无关.

## volatile

用于修饰成员变量, 使得每次访问此变量的时候都从内存读取, 而不能进行寄存器寄存的优化. 

- 当变量可能在编译器未知的情况下发生改变的时候使用, 如中断或多线程; 
- 对此内存的读写不能跳过的时候使用, 如 Memory Mapped IO.


## 函数修饰总结

### 函数名前

- `template`: 模板函数.
- `virtual`: 虚函数.
- `inline`: 内联.
- `static`: 对成员函数来说, 则表示静态成员函数, 不从属于具体对象; 对普通函数来说, 表示其作用域为当前文件.
- `extern`: 声明一个外部的函数.
- `explicit`: 构造函数专用, 防止隐式转换.
- `friend`: 友元函数, 使其能访问类内非公成员.
- `constexpr`: 表示编译期可计算.

### 函数名后

使用等号`=`来指定实现:
- `=0`: 虚函数专用, 表示纯虚函数.
- `=default`: 特殊成员函数专用, 表示使用默认实现.
- `=delete`: 特殊成员函数专用, 表示删除其默认实现.

- `const`: 表示函数不会修改对象, 但不包括`mutable`修饰的成员变量. 在自身内只能调用const成员方法.
- `volatile`: 表示使用此函数时对象状态随时会变. 在自身内只能调用volatile成员方法.
- `&`: 表示此成员函数只能被左值对象使用.
- `&&`: 表示此成员函数只能被右值对象使用.
- `override`: 表示此成员函数覆盖父类的对应虚函数, 如果没覆盖到就会报错.
- `final`: 表示此成员函数是最终实现, 子类不能再对此定义或覆盖.
- `noexcept`: 表示函数不会抛出异常.


## static


在函数或闭包内定义**静态局部变量**, 就可以让这些变量(被放在全局存储)仅仅在此上下文被使用, 每次调用函数都能访问此变量, 且永远不会被释放或重置.


## ROV & NROV

两个优化都自动将返回的目标左值作为隐式参数以引用的形式传入到函数中，以减少多余的构造、析构和拷贝。

### RVO 返回值优化（Return Value Optimization）

返回为临时对象时有效. 任何时候都可以完成优化.

```cpp
T f() { 
	return T(); 
} 

int main(){
	T t=f();
}
```

上面的代码若未优化，则`return T()`会进行一次构造，然后拷贝再析构。优化后`f()`等价于：

```cpp
void f(T &t){
	t=T();
	return;
}
```

但是，如果是返回被命名的变量，也即函数内局部变量的话：

```cpp
Data process(int i) {
    Data data;
    data.mem_var = i;
    return data;
}

int main() {
    Data data;
    data = process(5);
}
```

则等价于：

```cpp
void process(Data &data,int i) {
    Data d;
    d.mem_var = i;
    data=d; //data=Data(d);
    return;
}
```

还是有多余的构造、拷贝、析构。此时就需要NRVO来进一步优化.

> [!info]
> 从`c++17`开始，copy elision得到保证，强制使用RVO, 即使拷贝和移动构造函数还有其他作用.

### NRVO 命名返回值优化（Named Return Value Optimization）

若返回的变量是函数内的局部变量时（注意，不能是函数参数）有效。

```cpp
Data process(int i) {
    Data data;
    data.mem_var = i;
    return data;
}

int main() {
    Data data;
    data = process(5);
}
```

上面的代码若没有优化，则process内部也会构造一个对象，然后复制给外部的data变量，随后析构。

优化后，process函数等价于：

```cpp
void process(Data& data,int i) {
    data.mem_var = i;
    return;
}
```

NRVO无法优化的情况：
- 返回类型具有非默认的析构函数。
- 函数有多处return，且返回的变量不同。

编译时添加选项`-fno-elide-constructors`可以关闭NRVO。

## 基于范围的for循环

> 类似于java的增强型for循环.

要求目标是可迭代的, 具体来说:
- 该类型必须有一组和其类型相关的`begin`和`end`函数，它们可以是类型的成员函数也可以是独立函数。
- `begin`和`end`函数需要返回一组类似迭代器的对象，并且这组对象必须支持`operator*`, `operator!=`和`operator++`运算符函数。

cpp20中, 可以在这种for循环前加上初始化语句:
```cpp
T thing foo();
for (auto &x: thing.items()) {}

//C++20
for (T thing = foo(); auto &x: thing.items()) {}
```

## 带初始化语句的 if/switch

在判断语句的前面声明的变量的**生命周期会一直往下持续到整个if-else结束**, 因此`b1`在下面的`elseif`语句块中也能使用.

```cpp
if(bool b1 = foo1(); b1) {
	...
} else if(bool b2 = foo2(); b2) {
	...
}
```

可以利用初始化语句进行加锁保护:
```cpp
if (std::lock_guard<std::mutex> lock(mx); shared_flag)
	shared_flag = false;
}
```

switch的相关语法和if完全一样.

## std::static_cast

用于类型转换（替代了C风格的`(Type)var`）。
## std::copy

```cpp
template<class InputIterator, class OutputIterator>
  OutputIterator copy (InputIterator first, InputIterator last, OutputIterator result)
{
  while (first!=last) {
    *result = *first;
    ++result; ++first;
  }
  return result;
}
```

指定源的起点、终点，和目标的起点，即可将`[first,last)`处的数据进行逐个复制。调用的是拷贝赋值运算符。

目标要有容纳这些被拷贝的数据项的空间。

目标不应在`[first,last)`当中。

## decltype

将参数的类型返回，用于推断类型。

```cpp
// sum的类型就是函数f返回的类型
decltype(f()) sum = x;
```

decltype返回类型说明符（如int、double之类）。

参数可以为表达式。参数中的表达式并没有真被计算，函数也不会真被执行，而只是单纯由编译器分析。

`decltype(e)`(其中`e`的类型为`T`)的**推导规则**有五条：
1. 如果是一个<u>未加括号</u>的**标识符表达式**（结构化绑定除外）或者<u>未加括号</u>的**类成员访问**，则`decltype(e)`推断出的类型是`e`的类型`T`。如果并不存在这样的类型，或者e是一组重载函数，则无法进行推导。
2. 如果`e`是一个函数调用或者仿函数调用，那么`decltype(e)`推断出的类型是其返回值的类型。
3. 如果`e`是一个类型为T的左值，则`decltype(e)`是`T&`。
4. 如果`e`是一个类型为T的将亡值，则`decltype(e)`是`T&&o`
5. 除去以上情况，则`decltype(e)`是`T`。

> [!info]
> 使用括号可以恢复变量本身的各种类型信息(最重要的就是变回左值).

```cpp
const int&& foo();
int i;
struct A {
	double x;
};
const A* a = new A();

decltype(foo()); // const int&&
decltype(i); // int
decltype(a->x); // double
decltype((a->x)); // const double&
```

使用`decltype(auto)`可以借用`decltype`的规则对变量类型进行推导.
```cpp
auto x1=(i); // int
decltype(auto)x2 (i); // int&
```

## 后置返回值类型


[模板函数——后置返回值类型（trailing return type）\_模板函数返回\_HerofH\_的博客-CSDN博客](https://blog.csdn.net/qq_28114615/article/details/100553186)

```cpp
auto foo() -> int {...}
```

```cpp
template <class T>  
auto Get(std::string_view key) const -> const T *;
```

若要使用decltype推导返回值类型, 则需要使用函数参数，于是使用返回值后置使得函数参数先被声明然后才被使用。

## 省略返回值类型

可以不指定返回值, 只写个auto, 编译器会自动根据函数体推导.

```cpp
auto foo() {
	return 0;
}
```


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

## 结构化绑定

把一个tuple给拆开成多个变量, 原本需要使用`std::tie`来完成:
```cpp
int x = 0, y = 0;
std::tie(x, y) = return_multiple_values();
```

现在可以使用:
```cpp
auto [x, y] = return_multiple_values();
```

配合自动推导返回值类型的特性, 可以使其看起来像是能让函数返回多个变量:
```cpp
auto return_multiple_values() {
	return std:make_tuple(11, 7);
}

int main() {
	auto[x, y] = return_multiple_values();
}
```

总共有三种结构化绑定:
1. 绑定**原始数组**, 将从前往后的每一项提出来
2. 绑定**结构体与类**, 将<u>当前可访问</u>的非静态成员变量提出来
3. 绑定**元组**或**类元组**对象

> [!example] 类的结构化绑定不一定要public的例子
> 一个类的友元函数中对该类的对象进行结构化绑定时, 编译器会判断把private的变量也给提出来, 因为此时作用域内是可以访问private的.


## std::string_view

对string的一个引用，不发生拷贝。

## 函数（谓词）
### 仿函数（闭包）

若一个类重载了`operater()`，则其实现是个可调用对象，也叫仿函数、闭包。

### Lambda表达式

[Lambda expressions (since C++11) - cppreference.com](https://en.cppreference.com/w/cpp/language/lambda)

```cpp
[ 捕获 ] ( 形参 ) -> ret { 函数体 };
```

式子会返回一个匿名、右值的闭包。

每个lambda表达式都有唯一的类型(即使签名相同), 这是为了[方便编译器优化](Rust/Rust.md#函数类型唯一性与编译器优化理论).

当编译器可以推导返回值类型的时候（包括无返回值时），可以不写`-> ret`。

底层逻辑其实就是仿函数，即编译器自动生成一个重载了`operater()`的类。

#### 捕获 (capture)

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

**广义捕获**包括**简单捕获**和**初始化捕获**. 上面的捕获是简单捕获. 初始化捕获的例子:
```cpp
int x = 5;
auto foo = [r = x + 1]{ return r; }
```

所有**值捕获**都默认是`const`, 想要其<u>可变</u>的话就要在参数后面加上`mutable`.

#### 泛型lambda表达式

参数中使用auto来定义泛型lambda表达式, 从而兼容多种类型. 

```cpp
auto foo = [](auto a){ return a; }
int three = foo(3);
char const* hello = foo("hello");
```

也可以使用模板语法:
```cpp
auto foo = []<typename T>(std::vector<T> vec) {...}
```


#### 捕获this

如果定义的上下文是在对象里面, 那么**当前对象的this指针**可以捕获进lambda表达式(`[this]`). 显然, this指针只能进行**值捕获**.

但是, 也可以直接拷贝当前对象进去, 即`[*this]`.

虽然<u>目前</u>`[=]`也能直接捕获到`this`, 但是可以写成`[=, this]`来让语义更显式. 这也是为了与`[=, *this]`(捕获所有变量并且复制当前对象)相区分.

#### 可构造和可赋值的无状态lambda表达式

> [!warning]
> GCC 14.2.0 都尚不支持此特性.

cpp20中, 允许无状态(无捕获)的lambda表达式可以被构造和赋值(拷贝), 即为其保留默认构造函数和拷贝构造函数. cpp20前, 闭包类型不是默认可构造的.

例如下面的代码中, 想要用模板类型参数来指定`cmp`, 并且不希望通过传值来传递`cmp`闭包的话, 就要在`std::map`的构造函数内部对`decltype(greater)`这个无状态闭包类型进行构造. 这是cpp20才支持的, 以前还要给`mymap`额外传`greater`参数.
```cpp
auto greater = [](auto x, auto y) { return x > y;};  
std::map<std::string, int, decltype(greater)> mymap;
```

而下面这段代码需要greater拥有拷贝构造函数, 也是cpp20才支持:
```cpp
auto greater = [](auto x, auto y) { return x > y;};  
std::map<std::string, int, decltype(greater)> mymap1, mymap2;  
mymap1 = mymap2;
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

只能绑定值类型, 可以使用[std::ref](Cpp/Cpp.md#std%20ref)实现绑定引用.

## std::ref

`std::ref(t)`可以被<u>隐式类型转换</u>为对`t`的**引用**.

有时候函数参数只能传递值类型, 比如在使用[std::bind](Cpp/Cpp.md#std%20bind)时, 以及当模板类型为`T`而不是`T&`的时候, 可以通过使用`std::ref(t)`来实现传递引用(因为`std::ref`本身是个变量, 可以值传递).

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
priority_queue<类型,容器,比较函数类型>

// 大顶堆（降序）
priority_queue<int> big_heap;
priority_queue<int,vector<int>,less<int>> big_heap2;

// 小顶堆（升序）
priority_queue<int,vector<int>,greater<int>> small_heap;

// 传入自定义函数
priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(&cmp)> q(cmp);

// 传入自定义闭包
auto cmp = [](const ListNode* a, const ListNode* b){
	return a->val > b->val;
};
priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq;

// 使用仿函数
struct cmp {
    bool operator ()(const node &a, const node &b){
        return a.value>b.value; //按照value从小到大排列
    }
};
priority_queue<node, vector<node>, cmp>q;

// 重定义运算符
struct node{
    int value;
    friend bool operator<(const node &a,const node &b){
        return a.value<b.value; //按value从大到小排列
    }
};
priority_queue<node>q;
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

[C++中将Char转换成String的4种方法\_C 语言\_脚本之家](https://www.jb51.net/article/277515.htm)
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

### string

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

### string_view

对string的引用，不进行拷贝。

```cpp
#include <iostream>
#include <string_view>

void printSubstring(std::string_view str)
{
    std::cout << "Substring: " << str << std::endl;
}

int main()
{
    std::string fullString = "Hello, World!";
    std::string_view substring = fullString.substr(7, 5); // 获取子串 "World"

    printSubstring(substring);

    return 0;
}
```

### substr

string的substr是$O(n)$的，会直接拷贝出子串；但string_view的substr是$O(1)$的，不会发生拷贝。

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

## upper_bound & lower_bound


假设有一个有序容器，给出它的两个迭代器，就能使用这两个函数来通过**二分查找**找到想要的值的迭代器。

如果有一个**从小到大**排序好的数组`nums`，则：

```cpp
// 得到第一个大于等于10的元素的迭代器
auto it1 = lower_bound(nums.begin(), nums.end() , 10);
// 得到第一个大于10的元素的迭代器
auto it1 = upper_bound(nums.begin(), nums.end() , 10);
```

可以理解为，这两个函数都找到一个从小到大的有序容器中**等于**目标值的元素的范围`[lower, upper)`，然后`lower_bound`返回`lower`，`upper_bound`返回`upper`。

要在从大到小的数组里面查找的话，要使用`greater<int>()`：
```cpp
auto it = upper_bound(nums.begin(), nums.end() , 10 ,greater<int>());
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


## ranges

[Ranges library (C++20) - cppreference.com](https://en.cppreference.com/w/cpp/ranges)

### range concept

库中定义了`range` concept: [std::ranges::range - cppreference.com](https://en.cppreference.com/w/cpp/ranges/range)
```cpp
template< class T >
concept range = requires(T& t) {
  ranges::begin(t); // equality-preserving for forward iterators
  ranges::end  (t);
};
```

满足了`range` concept的类型就可以使用ranges库的东西了.

例如, 可以简化`sort`等函数的传参, 因为满足该concept后, 必然可以正常获取首尾迭代器.
```cpp
vector<int> vec{3,5,2,8,10};
std::ranges::sort(vec);
std::ranges::sort(vec, std::greater());
```

### Range Adaptors & 管道运算符 `|`

> [!info]
> `std::ranges::views`被起了别名: `std::views`.

range adaptors可以加工满足要求的东西, 如`std::views::take`, `std::views::transform` 和 `std::views::filter`. 下面的代码是通过嵌套调用来完成的: 
```cpp
std::vector<int> vec{20, 1, 12, 4, 20, 3, 10, 1};

auto even = [](const int &a) { return a % 2 == 0; };
auto square = [](const int &a) { return a * a; };

for (int i : 
	 std::views::take(
		 std::views::transform(
			 std::views::filter(vec, even), 
		 square), 
	 2)
	 ) {
	std::cout << i << " ";
}

// Output:
// 400 144
```

显然, 这种嵌套类似于链式调用. cpp提供了管道运算符让前一个函数的输出变成下一个函数的参数, 从而增强可读性:
```cpp
for (int i : 
	 vec | std::views::filter(even) |
	 std::views::transform([](const int &a) {return a * a;}) |
	 std::views::take(2)
	) {
	std::cout << i << " ";
}
```

range adaptors 是懒求值的.

自己实现管道运算符:
```cpp
#include <iostream>
#include <vector>
#include <functional>

std::vector<int>& operator|(std::vector<int>& v, std::function<void(int&)> func)
{
    for (int& i:v)
        func(i);
    return v;
}

int main()
{
    std::vector<int> v{1, 2, 3};
    std::function f{[](const int& i) {std::cout<< i << ' ';}};
    auto f2 = [](int& i) {i *= i;};
    v | f2 | f;
    return 0;
}
```

更通用的写法:
```cpp
template<class T>
std::vector<T>& operator|(std::vector<T>& v, std::invocable<T&> auto const &func)
{
    for (T& i:v)
        func(i);
    return v;
}
```


## piecewise_construct

有的函数参数是用单个tuple接收的。因此，编译器可能不知道应该去匹配tuple为参数的函数，还是单纯有若干个参数（和tuple内部结构一样）的函数。

此时，只要在传参的时候在第一个参数前附上`std::piecewise_construct`，就能显式地匹配单纯有若干个参数的函数。


## inline

TODO

### 将static类成员变量是声明和定义放一起

类中的static变量需要在class的定义中声明, 而class的定义一般写在头文件, 而头文件里面不应当对变量初始化, 于是不得不将其声明与定义分离. 现在使用inline即可在一处直接声明并初始化.

```cpp
class X {
	inline static std::string text{ "hello" };
}
```


## 异常

[std::exception - cppreference.com](https://en.cppreference.com/w/cpp/error/exception)

## 并发编程

conditional variable的第一个参数是lock，第二个参数是返回bool的函数。被notify时，会自动上锁，然后检查函数返回值是否为true，是的话继续执行，不是的话释放锁继续等待。最开始进入wait之前也会检查一次函数返回值。 这是为了让锁来保护函数的参数（条件），避免在wait的前一瞬间另一个线程修改了函数参数并完成调用notify，导致wait开始后一直收不到。在根据一个变量来判断是否应当wait时，上锁也能保证notify方在正确的时机修改变量并进行notify（notify可以写在锁unlock之后）。

unique_lock和lock_guard都在定义时给对应mutex上锁，在生命周期结束后自动释放锁。千万不要对它们使用unlock，否则会很难debug，特别是在不小心unlock了它们包裹的mutex而不是它们本身的时候。

### 线程局部存储 thread_local

在声明变量的时候在前面标上`thread_local`即可让该变量变成线程局部存储变量, 即只需声明一次, 但每个线程都能独立地拥有此变量. 

行为有点像`static`, 只会被初始化一次, 通常在线程结束时销毁. 其地址在运行时决定. 可以但不建议将其指针传递给其他线程.



# 模板

模板代码本身什么都不能做，它需要被使用后知道要生成哪些实例。并且所有实例都自带inline，不会重复定义。

（在预编译之后）声明和实现必须在同一个文件里面。如果把声明放在头文件、把实现放在cpp文件，则另一个cpp文件只会include头文件而会忽视cpp文件，于是只能通过模板函数声明推导出函数声明语句，而不会推导实现语句，使得实例（如`f<int>()`)会找不到实现。
## 函数模板
### 基本使用


```cpp
template <typename T> 
void f(const T &t); 

template <typename T> 
void f(const T &t) {} // 当然，文件内部没有声明依赖关系的时候，声明和实现可以合并
```

## using

using可以替代typedef, 但其最大的作用是创建**别名模板(alias template)**.

```cpp
// 重定义unsigned int
typedef unsigned int uint_t;
using uint_t = unsigned int;
// 指定std::map的模板参数 (别名模板)
typedef std::map<std::string, int> map_int_t;
using map_int_t = std::map<std::string, int>;
```

## 函数模板的默认模板参数

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


## 特化

有时候对模板类型的操作不能通用，需要单独特殊处理，则可以用特化。

```cpp
#include <cstring>

template <typename T>
void add(T &t1, const T &t2) {
  t1 += t2;
}

template <> // 模板特化也要用模板前缀，但由于已经全部特化了，所以参数为空
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

确定了<u>所有模板参数</u>后即为**全特化**。全特化已经是个实例，因此要当成普通函数/类来对待。如果要和模板函数/类一起写在头文件的话，要提防重复定义，如手动加上inline。

## 可变参数模板

使得模板类型参数可以有任意多个, 参数类型可以不一致. 它可获取参数个数(使用`sizeof...`).

```cpp
template<class ...Args> 
void foo(Args ...args) { 
	cout << sizeof...(args) << endl;
} 

template<class ...Args> // 类型模板形参包
class bar {
public:
	bar(Args ...args){ // 函数形参包
		foo(args...); // 形参包展开
	}
};
```

递归计算:
```cpp
// 递归终点, 只剩一个参数的时候
template<class T>
T sum(T arg) {
	return arg;
}

// 每次都提取出第一个参数
template<class T1, class ...Args>
auto sum(T1 arg1, Args ...args){
	return arg1 + sum(args...);
}

int main() {
	std::cout << sum(1, 5.0, 11.7) << std::endl;
}
```

利用**折叠表达式**:
```cpp
template<class...Args>
auto sum(Args ...args) {
	return (args + ...); // 包展开, 每个参数之间用`+`连接
}

int main() {
	std::cout << sum(1 ,5.0, 11.7) << std::endl;
}
```

`(args + ...)`是向右折叠, 先计算最靠右的参数. `(... + args)`就是向左折叠.


## concept

[Constraints and concepts (since C++20) - cppreference.com](https://en.cppreference.com/w/cpp/language/constraints)

[Requires expression (since C++20) - cppreference.com](https://en.cppreference.com/w/cpp/language/requires)

以前的SNIFAE机制, 让编译器只会选择编译正确的那个特化实例; 但有了concept机制之后, 可以把约束全都写到一起并进行组合, 使得在模板特化之前就检查模板类型参数是否满足要求.

一个模板定义可以`require`一些requirements.

concept是被命名的requirements的集合; `require` concept等价于`require`其对应的requirements.

```cpp
// 定义一个concept
template<class T, class U>
concept Derived = std::is_base_of<U, T>::value;
```

```cpp
#include <cstddef>
#include <concepts>
#include <functional>
#include <string>
 
// Declaration of the concept “Hashable”, which is satisfied by any type “T”
// such that for values “a” of type “T”, the expression std::hash<T>{}(a)
// compiles and its result is convertible to std::size_t
template<typename T>
concept Hashable = requires(T a)
{
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};
 
struct meow {};
 
// Constrained C++20 function template:
template<Hashable T>
void f(T) {}

// Alternative ways to apply the same constraint:

// 紧跟模板参数后面
template<typename T>
    requires Hashable<T>
void f1(T) {}

// 函数声明后面
template<typename T>
void f2(T) requires Hashable<T> {}

// 直接写入声明中, 不用写模板参数了.
// 而这个参数类型本身成为concept的第一个参数
void f3(Hashable auto /* parameter-name */) {}
 
int main()
{
    using std::operator""s;
 
    f("abc"s);    // OK, std::string satisfies Hashable
    // f(meow{}); // Error: meow does not satisfy Hashable
}
```




# 问题、技巧、解决方案

## 通用头文件

```cpp
#include<bits/stdc++.h>
```

## 打印类型名

使用`typeid`后, 通过`name`方法获取类型名.

```cpp
int i = 1;
std::cout << typeid(i).name() << std::endl;
```

## 查看算数类型的性质

[std::numeric\_limits - cppreference.com](https://en.cppreference.com/w/cpp/types/numeric_limits)

例如查询`long long`的最大值用`std::numeric_limits<long long>::max()`.

注意要包含`#include <limits>`.

## vector越界检查

使用at访问数据而非`[]`，可以在越界时抛出异常。

## 生成随机数

生成0~1：
```cpp
#include <iostream>
#include <random>

int main() {
  std::mt19937 gen(std::random_device{}());
  std::uniform_real_distribution<double> dis(0, 1);
  for(int i=0;i<100; i++) {
    std::cout<< dis(gen) << std::endl;
  }
}
```

或者用闭包：
```cpp
function<double()> rand = [] {
	static std::mt19937 gen(std::random_device{}());
	static std::uniform_real_distribution<double> dis(0, 1);
	return dis(gen);
};
```

## 获取c数组长度

```cpp
int len=sizeof(arr)/sizeof(arr[0]);
```

## 重载cout输出

写在头文件中
```cpp
friend std::ostream& operator<<(std::ostream& os, const BinarySquareMatrix& matrix) {  
    for (const auto& row : matrix.data) {  
        for (const auto& elem : row) {  
            os << elem << ' ';  
        }  
        os << '\n';  
    }  
    return os;  
}
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

## 遍历枚举

将枚举的最后一项设为专门用于记录枚举数量，即可使用`N_TASKSYS_IMPLS`对枚举进行for循环遍历：
```cpp
enum TaskSystemType {
    SERIAL,
    PARALLEL_SPAWN,
    PARALLEL_THREAD_POOL_SPINNING,
    PARALLEL_THREAD_POOL_SLEEPING,
    N_TASKSYS_IMPLS, // This must be in the last position.
};
```

## 将不能copy的对象放入map

只能使用emplace，但似乎由于emplace的普通使用依然会进行拷贝（TODO: 暂时搞不懂为什么不能普通地写，连move都不行）。

先传入`piecewise_construct`，表示分开构造key和value，因此还需要传入两个tuple。使用`forward_as_tuple(0)`构造key，然后使用`forward_as_tuple(10)`真原地构造不可复制的对象。如果参数（如上面的0和10）不需要完美转发的话，直接`make_tuple`也是没问题的，但有弊无利。无参的话直接`tuple<>()`。

```cpp
class NoCopyClass {  
    std::mutex mu;  
public:  
    NoCopyClass() = default;  
    explicit NoCopyClass(int value) {} // 构造函数  
};  
  
int main() {  
    std::unordered_map<int, NoCopyClass> no_copy_map;  
    // 触发默认构造函数  
    no_copy_map.emplace(std::piecewise_construct,  
                        std::forward_as_tuple(0),  
                        std::tuple<>());  
    // 以完美转发的形式（此处为右值）  
    no_copy_map.emplace(std::piecewise_construct,  
                        std::forward_as_tuple(1),  
                        std::forward_as_tuple(10));  
    // 普通地构造tuple传进去，没有完美转发  
    no_copy_map.emplace(std::piecewise_construct,  
                        std::make_tuple(2),  
                        std::make_tuple(10));  

	// no_copy_map.emplace(3, NoCopyClass()); // Error: In template: no matching constructor for initialization of 'std::pair<const int, NoCopyClass>'

	// no_copy_map.insert({3, NoCopyClass()}); // Error: No matching member function for call to 'insert'

	// no_copy_map.emplace(std::piecewise_construct, std::make_tuple(3), std::forward_as_tuple(NoCopyClass())); // Error: In template: call to implicitly-deleted copy constructor of 'NoCopyClass'

    return 0;  
}
```

## 移除有序容器中的重复元素

>[!note]
 vector先排序就能用

```cpp
hbound.erase(unique(hbound.begin(), hbound.end()), hbound.end());
```
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


# 编译相关

`#pragma optimize("", off)`可以关闭所有编译优化。把off改成on则恢复默认优化。

- `-o xxx` 输出文件的名字 
- `-std=c++11` 切换为cpp11标准
- `-Og` 将编译器的优化等级调整为符合C语言原程序的结构，方便学习




# 杂记

INT_MAX即为int类型的最大值的宏。

















