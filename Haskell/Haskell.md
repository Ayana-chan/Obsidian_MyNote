[Haskell趣学指南\_w3cschool](https://www.w3cschool.cn/hsriti/)
# 安装与使用

## 安装

使用科大源安装GHCup: [GHCup - USTC Mirror Help](https://mirrors.ustc.edu.cn/help/ghcup.html)

[配置 Haskell 开发环境（2023） | Zhichao Guan’s Home Page](https://higher-order.fun/cn/2023/08/27/InstallHaskell.html)

换源下载完GHCup后, 依然会卡在其他安装, 此时可以安全把安装窗口关掉, 换源后手动`ghcup install xxx` 和 `cabal update`等.

stack是独立的, 就算外部安装了GHC (The Glasgow Haskell Compiler), 在stack里面还得再装一次.

使用`STACK_UP`环境变量设置stack的位置(包和配置文件), 并且只有第一次跑过了stack才会新建文件夹. 要注意重启电脑防止一些终端读不到环境变量而误操作.

stack可能会在代码有warning时出现以下编译报错, 但是原地再次编译就没事了(此时不会报warning), 应该是终端编码问题,使其不兼容warning的情况.
```bash
Error: [S-7282]
       Stack failed to execute the build plan.

       While executing the build plan, Stack encountered the error:

       <stderr>: commitAndReleaseBuffer: invalid argument (cannot encode character '\8226')
```

[Get Started](https://www.haskell.org/get-started/)

## vscode

[Vscode Haskell](https://marketplace.visualstudio.com/items?itemName=haskell.haskell)

`stack.yaml`中, 使用科大源:
```yaml
snapshot:
  url: http://mirrors.ustc.edu.cn/stackage/stackage-snapshots/lts/22/43.yaml
```

## 链接

标准库查询: [Haskell Hierarchical Libraries](https://downloads.haskell.org/ghc/latest/docs/libraries/)

检索库内容(包括第三方库): [Hoogle](https://hoogle.haskell.org/)

# ghci

直接运行haskell的工具.

- `:l :load` 加载(hs文件)
- `:r :reload` 重载
- `:t :type` 获取类型
- `:i :info` 信息，针对函数、类型、类型类(能看到有哪些instance)等。
- `:k :kind` 得知一个类型的Kind。其中`Type`使用`*`表示.

引入模块：
```haskell
:m module1 module2 module3
```

# stack

```bash
stack new my-project
cd my-project
stack setup
stack build
stack exec my-project-exe
```

# 数学

## 群

设$(S, *)$为一个带有二元运算$*$的集合$S$ (称为**原群**), 则:
- 同时满足下面两条的称为**幺元(identity)** 或 **单位元(unit)**:
	- $e$是**左幺元**, 如果$\forall a \in S, e * a = a$.
	- $e$是**右幺元**, 如果$\forall a \in S, a * e = a$.
- 同时满足下面两条的称为**零元(zero element)**:
	- $\theta$是**左零元**, 如果$\forall a \in S, \theta * a = \theta$.
	- $\theta$是**右零元**, 如果$\forall a \in S, a * \theta = \theta$.


若集合$G \neq \varnothing$, 在$G$上的二元运算 $\cdot : G \times G \rightarrow G$ (称为群的乘法, 结果称为积) 构成的代数结构$G(G, \cdot)$, 满足:
1. **封闭性 (Closure)**: 运算结果依然在集合内. $\forall a, b \in G, a \cdot b \in G$.
2. **结合律 (Associativity)**: 运算有结合律. $\forall a,b,c\in G, (a\cdot b)\cdot c=a\cdot(b\cdot c)$.
3. **幺元 (Identity)**: 存在<u>幺元</u>$e$对任意元素的左右乘都为元素本身.$\exists e \in G, \forall a\in G, e\cdot a=a\cdot e=a$.
4. **逆元**：存在$b$称为$a$的<u>逆元</u>,记为$a^{-1}$. $\forall a\in G,\exists b\in G, a\cdot b=b\cdot a=e$. 
则$(G, \cdot)$称为一个**群 (Group)** 或**乘法群**, 简记为$G$.

- 若只满足 Closure 和 Associativity, 则称为**半群 (Semi-group)**.
- 若只满足Closure, Associativity 和 Identity, 则称为**含幺半群 (Monoid)**.

若群$G$满足**交换律**, 即$\forall a,b \in G, a \cdot b = b \cdot a$, 则称为**阿贝尔群 (Abelian Group)** 或**交换群**或**加群**.

若对于两个群$\left(G,\cdot\right)$ 和$\left(H,\times\right)$,有映射$f:G\to H$ 满足以下条件：$\forall a,b \in G, f(a\cdot b)=f(a)\times f(b)$, 则称<u>映射</u>$f$为群$(G,\cdot)$<u>到</u>群$(H,\times)$的**同态**。

- 如果映射$f$为<u>单射</u>，则称$f$为**单同态**。
- 如果映射$f$为<u>双射</u>，则称$f$为**同构**。

如果$e_G$是$G$的幺元，$e_H$是$H$的幺元, 则上述同态$f$满足：
$$f(e_G)=e_H$$



# 语法
## 基础

Haskell是强类型和静态类型的。

Haskell是惰性的，如非特殊说明，函数真正需要结果以前不会被求值，加上引用透明，可以把程序看做数据的一系列变形。也就是说惰性语言中的计算只是一组初始数据和变换公式。例如下面这一段如果长度不一样(`a`)就不会执行内容比较(`b`):
```haskell
lengthCompare :: String -> String -> Ordering  
lengthCompare x y = let a = length x `compare` length y   
                        b = x `compare` y  
                    in  if a == EQ then b else a  
```

在函数式编程语言中，变量一旦指定就不可以更改了，在命令式编程语言中，变量表示状态，如果状态不可变，那么能做的事情将非常有限。而函数式编程语言中，变量的含义更接近数学中的变量，`x=5`表示`x`就是`5`，而不是`x`处于`5`这个状态。

函数式编程语言的一般思路：先取一个初始集合，对其进行变形、执行过滤条件（map and reduce）得到最终结果。

所有的表达式都要求返回一个值. `if`语句也是个表达式, 因此必须有`else`. 如果`else`情况不应该做任何事情, 也不知道该如何返回值的话, 应该使用`when`.

**type variable (类型变量)** 就是泛型, 由小写字母`a` `b` `c`等表示. 使用类型变量的函数称之为**多态(polymorphic)函数**.



## 杂项

**Lifting** 指的是把一个函数转化出一个新函数, 新函数有新配置 (往往更强).

`id`函数(`id :: a -> a`)只是非常单纯地把输入原封不动扔出来(`\x -> x`).

`undefined`被使用时, 会报`Exception: Prelude.undefined`. 可以用来当assert来防止程序使用了不该使用的地方. 但由于Haskell非常Lazy所以可能比较玄学.
```haskell
ghci> head [3,4,5,undefined,2,undefined] 3
```



## 注释

```haskell
-- 单行注释

{-
多行注释
-}
```

在`{- -}`里面使用`>>>`可以进行计算求值, 值会显示在下一行. 注意不要写死循环!
```haskell
{-
>>> fib' 10
55
>>> fib' 100
354224848179261915075
>>> fib' 300
222232244629420445529739893461909967206666939096499764990979600
>>> fib' (50 :: Integer)
12586269025
>>> fib' (100 :: Int)
3736710778780434371
-}

fibonacci :: Integral a => a -> Integer -> Integer -> Integer
fibonacci 0 a b = b
fibonacci n a b = fibonacci (n - 1) (a + b) a
fib' :: Integral a => a -> Integer
fib' n = fibonacci n 1 0
```

## 运算符

`(xxx)`可以**把运算符弄成函数**(或者说形式上变成前缀式), 例如`(+) 2 3`就是`2 + 3`, `(->) a b`就是`a -> b`.


优先级|左结合|不结合|右结合
:-:|:-|:-|:-
9|`!!`||`.`
8|||`^` `^^` `**`
7|`*` `/` `` `div` `` `` `mod` `` `` `rem` `` `` `quot` ``||
6|`+` `-`||
5|||`:` `++`
4||`=` `/=` `<` `<=` `>` `>=` `` `elem` `` `` `notElem` ``|
3|||`&&`
2|||`\|\|`
1|`>` `>>=`||
0|||`$` `$!` `seq`

`<*>`是左结合



## type 类型

**大驼峰**命名.

每个表达式都有type. "a的类型为A"表示为`a :: A`. 函数调用也是表达式, 因此在函数调用末尾加上`:: A`可以限制函数返回类型.

```haskell
'a' :: Char
False :: Bool
"Hello" :: [Char]
(True, 'a') :: (Bool, Char)
4 /= 5 :: Bool
```

`Int`是普通的类似`usize`的整数, 而`Integer`是无界整数.

函数type用`->`串起来, 最后一个是返回值, 前几个全是输入. 比如输入两个浮点数输出整数的函数的类型为`Float -> Float -> Int`. 实际上, 是第一个为输入, 后面的合为一个输出, 详见[函数](Haskell/Haskell.md#函数).

完整的type声明需要结合typeclass.

### Type Constructor

`Int`, `Map` 等都是 **type constructor (类型构造子)**, 有type参数. **无参数**的 type constructor 称作 **nullary** type constructor, **简称 type**.

> [!tip]
> 看样子 type constructor 才是广义上的类型.

type constructor 可以看做<u>类型意义</u>上的<u>函数</u>, 但不是haskell的函数 (在lisp里面它直接就是普通函数). type constructor 也可以被**Curry**, 例如 `Map Int` 依然是一个单参数的type constructor. 在[kind](Haskell/Haskell.md#kind)当中它确实就写作函数type的形式.

`data`表达式的`=`左侧就是一个 type constructor:
```haskell
data NameOfType = ValueConstructor1 TypesOfParams1 | ... deriving (Typeclass1, Typeclass2, ...)
```

带参数的 type constructor 的参数具有声明作用, 使得`=`右边的 value constructor 可以使用这些 type variable.

```haskell
-- `Bool` 是 nullary type constructor
data Bool = True | False
-- `Tree` 是带参数的 type constructor
data Tree a = Tip | Node a (Tree a) (Tree a)
```

```haskell
-- 即 Option<T>
data Maybe a = Nothing | Just a
```

> [!notice]
> 不要在定义type的时候给type variable添加约束. 有约束应当在函数处写.



### Value Constructor

`data`表达式的`=`右侧的 **Value Constructor (或Data Constructor, 值构造子)** 使用`|`(或)来分割. 它们明确类型的所有可能取值. **无参数**的value constructor (例如`True`, `[]`, `5`) 称作**nullary** value constructor. 

value constructor是<u>跟着type一起声明</u>的, 可以理解为赋予了这种值结构以一个名字. 如果值构造子只有一个, 那么完全可以让type名和value constructor名相同. 

value constructor 就是个**函数** (不过其名字与参数可以被模式匹配)(而不是类型!). 名字要写成**大驼峰**. 一样可以被Curry, 也可以被赋值给其他函数.

由无参value constructor组成的类型:
```haskell
data Bool = False | True
```

后面可以跟着`deriving`来表示**派生 typeclass**.
 
```haskell
-- Shape类型的变量可以被输出到控制台
data Shape = Circle Float Float Float | Rectangle Float Float Float Float deriving (Show)
```

上面这个`data`语句相当于定义了两个函数, 它们传入一定的参数就能得到一个Shape type的值; 同时, 它们本身就是对`Shape`这个type的定义 (即使用所有构造方式来定义类型本身). **定义type时所有参数都要填满** (不然函数是定义不出来的).
```haskell
Circle :: Float -> Float -> Float -> Shape   
Rectangle :: Float -> Float -> Float -> Float -> Shape
```

由于一个type的value constructor是可穷举的, 因此可以使用**模式匹配**来区分属于一个type的变量的不同value constructor的情况, 并获取对应的value constructor的参数.

计算`Shape`面积的函数可以使用模式匹配写成:
```haskell
surface :: Shape -> Float    surface (Circle _ _ r) = pi * r ^ 2    surface (Rectangle x1 y1 x2 y2) = (abs $ x2 - x1) * (abs $ y2 - y1)
```

`Shape`的另一种写法:
```haskell
-- 重名, 因为只有一个value constructor
data Point = Point Float Float deriving (Show)    data Shape = Circle Point Float | Rectangle Point Point deriving (Show)
```

**导出value constructor**需要在type后面加`(xxx, xxx)`, 或者使用`(...)`导出所有该type的value constructor. 不导出值构造子就可以禁止模块外直接访问值内部变量, 或自行构造此值.
```haskell
module Shapes( 
	Point(..), 
	Shape(Rectangle, Circle), 
	surface
) where
```

### Algebraic Data Type

在这里"algebra"指和与积.
- 和(sum)表示alternation, 即`A | B`, 只能是其一. 例如`data Pair = I Int | D Double`, 只是一个数字, 使用 tag `I`和`D` 来区分两种情况.
- 积(product)表示combination, 即`A B`, 二者都有. 例如`data Pair = P Int Double`, 把两个type都包进来, 并使用 tag `P` 来进行相关操作.

### Record Syntax

特殊的value constructor定义方式, 赋予 value constructor 的每一个参数以一个名字

```haskell
data Person = Person { firstName :: String   
                     , lastName :: String   
                     , age :: Int   
                     , height :: Float   
                     , phoneNumber :: String   
                     , flavor :: String   
                     } deriving (Show)
```

haskell就为每个名字自动生成一个函数, 以方便从类型中获取对应的值:
```haskell
ghci> :t flavor   
flavor :: Person -> String   
ghci> :t firstName   
firstName :: Person -> String
```

> [!info]
> 这种情况下, Show的打印也会带上名字.

如果一个类型是多个 record syntax 使用`|`连接的, 那么取某一项的时候显然无法静态判断是否能取到, 如果取错了会运行时报错.
```haskell
data Shape
  = Circle {radius :: Float}
  | Rectangle {width :: Float, height :: Float}
  deriving (Show)

main :: IO ()
main = do
  -- Err: No match in record selector radius
  let s = Rectangle 20 30
  let rad = radius s
  print rad
```

### 实现 typeclass

#### deriving

type 可以通过在data表达式末尾使用 `deriving` 来**自动**实现 typeclass.

如果type里所有的value constructor都是nullary的, 显然有前后继, 且有边界, 因此可以直接deriving `Enum` 和 `Bounded`.
```haskell
data Day = Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday    
           deriving (Eq, Ord, Show, Read, Bounded, Enum)
```

也就可以作比较:
```haskell
ghci> Saturday == Sunday   
False   
ghci> Saturday == Saturday   
True   
ghci> Saturday > Friday   
True   
ghci> Monday `compare` Wednesday   
LT
```

#### instance

可以使用`instance`关键字来**手动**实现typeclass, 即自定义实现各个所需函数的实现:
```haskell
import Prelude hiding ((/=), (==)) -- 排除默认符号

data TrafficLight = Red | Yellow | Green
instance Eq' TrafficLight where
	-- 对`Eq'`要求的`==`中缀函数进行模式匹配
    Red == Red = True
    Green == Green = True
    Yellow == Yellow = True
    _ == _ = False
```

支持对带有type variable的type进行instance实现, 从而对<u>所有满足要求</u>的type提供实现.

支持用`=>`对type variable进行**约束**, 在`=>` **左边**要求其实现哪些typeclass.

```haskell
instance (Eq' m) => (Eq' (Maybe m)) where
  Just x == Just y = x == y
  Nothing == Nothing = True
  _ == _ = False
```


### type synonyms 类型别名

使用`type`给类型起别名.

```haskell
type String = [Char]
```

可以保留type variable:
```haskell
type AssocList k v = [(k,v)]
```


### newtype

有时候, 可能要**给一个类型加一个wrapper**, 以让其实现新的功能(尤其是重新定义typeclass). 例如, `ZipList`用于重定义List的apply函数, 因此将其定义为:

```haskell
data ZipList a = ZipList [a]      
-- 使用下面的定义, 使其可以使用`getZipList`函数取得内部List
data ZipList a = ZipList { getZipList :: [a] }
```

而使用`newtype`代替`data`即是在告诉编译器: 这是对单个类型的包装, 不要为包裹和解包付出额外代价(有点类似于rust的transparent). 这也使其只能接受单个单参的value constructor. 

```haskell
newtype ZipList a = ZipList { getZipList :: [a] }
```

这种包装使得值构造子和获取内部变量的函数体现为<u>类型转换</u>的形式:
```haskell
ZipList :: [a] -> ZipList a
getZipList :: ZipList a -> [a]
```

> [!note]
> 把newtype理解为类型变换是没有大问题的.

> [!note]
> 单纯别名的话使用`type`.



#### 用于更换 type variable 顺序

由于实现typeclass的时候, 总是只能留空最后几个 type variable, 例如无法对`(a, b)`的`a`进行Functor操作. 此时就可以使用`newtype`创造一个wrapper, 从而调换type variable顺序.

```haskell
newtype Pair b a = Pair { getPair :: (a,b) }
```

于是就可以通过`Pair`将type variable反转, 在操作完毕之后使用`getPair`转变回来.
```haskell
ghci> getPair $ fmap (*100) (Pair (2,3))  
(200,3)  
ghci> getPair $ fmap reverse (Pair ("london calling", 3))  
("gnillac nodnol",3) 
```


## typeclass 类型类

typeclass定义了类型的行为, 类似于接口或trait.

- `Eq` 可判断相等性的类型
- `Ord` 可比较大小的类型

- `Show` 可**表示为字符串**的类型 `show :: Show a => a -> String`.
- `Read` 可**从字符串转换出值**的类型 `read :: Read a => String -> a`.

- `Enum` 连续的，也就是可枚举的类型。每个值都有后继 (successer) 和前置 (predecesor)，分别可以通过 `succ` 函数和 `pred` 函数得到。**属于`Enum`类型类的类型可以用于`Range`中**。满足`Enum`的类型有：`() Bool Char Ordering Int Integer Float Double`。
- `Bounded` 有**上限和下限**。`minBound :: Bounded a => a`. 例如：`maxBound :: Char`('\1114111'), `maxBound :: Bool`(True).

- `Num` 数字. 使用`fromIntegral`(为`(Integral a, Num b) => a -> b`)将其他数字typeclass转换为`Num`, 使其更通用.
- `Integral` 整数，包括`Int`, `Integer`
- `Floating` 浮点数，包括`Float`, `Double`

在表示一个东西的type的时候, 有时候需要使用type variable, 但是又需要对其做出限定(即泛型约束), 那么就需要在`=>`**左边**写明每个type variable要**属于哪些typeclass** (这部分可以使用括号).
```haskell
-- `==`二元函数对所有满足了Eq typeclass的类型适用
(==) :: Eq a => a -> a -> Bool
elem :: (Foldable t, Eq a) => a -> t a -> Bool
```

type class 可以被 type 给[实现](Haskell/Haskell.md#实现%20typeclass), 实现了typeclass的type称作 **instance**.

### 定义 typeclass

使用`class`关键字来定义typeclass. 如下`Eq`定义, `a`是一个类型变量, 表示任意满足了`Eq` typeclass的类型. `where`后规定`Eq`所要求的若干函数的**类型声明**, 下面的<u>函数定义不是必须</u>的.

```haskell
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y)
```

支持使用`=>`声明typeclass的**派生关系**, 即要求能实现当前typeclass的类型必须满足另一个typeclass的约束 (`=>`<u>右边</u>是要声明的typeclass, 左边是父typeclass):
```haskell
-- `Num` 是 `Eq` 的子typeclass
class (Eq a) => Num a where
    ...
```

typeclass内提供的函数的定义是其提供的**默认实现**. 如`Eq`, 其中`==`默认使用`/=`实现, `/=`默认使用`==`实现. 这要求使用者要提供其中一个函数的新实现, 从而为其自动实现另一个函数 (否则就会出现死递归).

### HKT（High-Kind Type）理论

- **零阶类型**：Value，如`114514`。
- **一阶类型**：如`i32`，`List<i32>`。`List`是类型构造器，可以通过传入`i32`构造出`List<i32>`。OOP的接口、Rust的Trait，都是**对一阶类型的抽象**。
- **高阶类型（HKT）**：类型的类型，是**对类型构造器的抽象**。例如，`class Functor f`这个typeclass，其内部函数定义为`fmap :: (a -> b) -> f a -> f b`，可见它要求f是一个类型构造器。

Rust的trait不是HKT，因为只有具体的类型（一阶类型）可以实现trait。注意，trait的定义方式就注定了它只能让一阶类型去impl；即使让带泛型的类型进行impl，也只不过是自动为多个具体类型实现trait。

Rust定义Monad trait，其bind最多只能定义出`m a -> (a -> m a) -> m a`这样的劣质版本：
```rust
trait Monad<T> {  
    fn pure(v: T) -> Self;  
    fn bind(self, p: impl Fn(T) -> Self) -> Self;  
}  
  
impl<T> Monad<T> for Vec<T> {  
    fn pure(v: T) -> Self {  
        vec!(v)  
    }  
  
    fn bind(self, p: impl Fn(T) -> Self) -> Self {  
        self.into_iter().flat_map(p).collect()  
    }  
}
```
想要定义出`m a -> (a -> m b) -> m b`，则只能写成这样，显然无法过编译，因为Self不能带类型参数：
```rust
trait Monad1 {  
    fn pure(v: T) -> Self<T>;  
    fn bind(v: Self<T>, p: impl Fn(T) -> Self<F>) -> Self<T, F>;  
}
```

> [!note]
> OOP 抽象问的是“你是什么东西？你能做什么？”，而 HKT 问的是“你是哪种容器？你能对你的内容做什么操作？


## kind

**type constructor** 的"类型" 称为 **Kind**, 被表示成函数type的样子, 其中"变量type"是`Type`, 表示任意type. 在ghci中`Type`使用`*`表示.
```haskell
Int     :: Type
Maybe   :: Type -> Type
[]      :: Type -> Type
(->)    :: Type -> Type -> Type
Either  :: Type -> Type -> Type
Map     :: Type -> Type -> Type
Map Int :: Type -> Type
```

当type的定义要求其某个 type variable 为含参type constructor 时, 其kind会将其表示出来(不过本来`Type`就能表示`Type -> Type`). 此时可以将其<u>类比</u>为**将函数作为参数的函数**.

```haskell
-- `:: b a` 表示 `b` 是一个单参type variable
ghci> data Frank a b = Frank {frankField :: b a}
ghci> :k Frank
-- a 写作 k
-- b 写作 k -> *
Frank :: k -> (k -> *) -> *
```

**typeclass** 的 kind 是: 传入一个`Type`, 给出一个`Constraint`.
```haskell
Eq :: Type -> Constraint
```



## 函数

**小驼峰**命名.

等号左边写参数, 右边写函数体.

不需要先声明就可以使用.

```haskell
functionName arg1 arg2 = arg1 + arg2
```

> [!info]
> `'`也是函数名的合法字符，常常使用单引号来区分一个稍经修改但差别不大的函数。

把反引号(`` ` ``)放在函数名首尾来把函数变成中缀, 例如``10 `elem` [1, 2, 10]``.

<u>没有参数的函数</u>称之为**定义**(define)或者名字，定义后不可修改. 例如`otherwise`永远返回True.

### Curry 柯里化

实际上, Haskell的**所有函数都只能接收<u>一个</u>参数**, 因为有[**Curry (柯里化)**](函数式编程/函数式编程.md#Currying%20柯里化)的支持. 例如有一个"双参函数" `add :: a -> b -> c`, 在调用`add x y`时, 首先会把`x`传给`add`, 得到一个类型为`b -> c`的函数; 然后再把`y`传进去, 使之变成值`c`.

> [!note]
> `->`是**右结合**的.

下面两个函数是等价的 (一般会要求写成前者):
```haskell
-- 用柯里化弄出个单参函数来定义自己
max4 :: (Ord a, Num a) => a -> a
max4 = max 4
-- 自己要求一个参数, 但其余的定义只是个变量
max4' :: (Ord a, Num a) => a -> a
max4' x = max 4 x
```

由于右结合, 所以`a -> (a -> a)`与`a -> a -> a`等价. 也正因此可以认为, 函数定义是给等号右边的东西(可以是变量或函数)**在左侧按序增加若干参数**, 将其拼成更长的函数.

定义`max4'' x = max x 4`即Curry了`max`函数的第二个参数, 显得也不是很流畅, 是普通情景下的Curry. 中缀函数也要这样手动柯里化:
```haskell
divBy10 :: Double -> Double
divBy10 = (/10)
divX :: Double -> Double
divX = (10/)
```

> [!info]
> 对于某些一元和二元运算符使用同一个符号的情况，比如`-`用作减号和负号，`(-4)`则表示值-4，而不是接受一个参数将参数减4的函数。属于例外，为了避免冲突的选择，要使用减号含义则可以使用`subtract`，负号含义和`negate`等价。

> [!notice]
> 当想把函数类型作为一个函数的返回值时, 要意识到`->`是右结合的, 因此如果函数内生成函数类型返回值的逻辑过于复杂时, 想想是不是可以把东西都放到参数里面, 通过柯里化的逻辑来实现.

### where

可以使用`where`来定义局部变量, 从而无需使用`do`和`let`.

```haskell
lengthCompare :: String -> String -> Ordering  
lengthCompare x y = (length x `compare` length y) `mappend`  
                    (vowels x `compare` vowels y) `mappend`  
                    (x `compare` y)  
    where vowels = length . filter (`elem` "aeiou")  
```


### 函数作为参数

函数作为参数的时候就需要给`->`链加括号了, 如`(a -> a) -> a`表示 传入`a -> a`函数得到`a`变量 的函数.

```haskell
-- 传入一个双参函数和两个List, 把List元素一一对应应用到函数中
zipWith' :: (t1 -> t2 -> a) -> [t1] -> [t2] -> [a]
-- 存在空列表的情况, 停止递归
zipWith' _ [] _ = []
zipWith' _ _ [] = []
-- 调用`f x y`得到单项结果并递归拼接
zipWith' f (x:xs) (y:ys) = f x y : zipWith' f xs ys
```

### 常用高阶函数

高阶函数：函数可以作为参数、返回值、赋给另一个变量。

- `map :: (a -> b) -> [a] -> [b]` 映射一个列表到另一个列表。
- `filter :: (a -> Bool) -> [a] -> [a]` 筛选符合条件的元素到结果列表。

> [!note]
>  `map` 和 `filter`等价于列表推导式.

- `takeWhile :: (a -> Bool) -> [a] -> [a]` 按顺序取元素直到条件不满足。
- `zipWith :: (a -> b -> c) -> [a] -> [b] -> [c]` 将两个列表的对应元素应用函数后得到新列表。
- `flip :: (a -> b -> c) -> b -> a -> c` 接受一个二元函数，将两个参数翻转并返回新的二元函数。

fold系列操作把列表变成值. 详见[Foldable](Haskell/Haskell.md#Foldable).

scan系列操作保留fold每次的结果, 替换原元素.
```haskell
scanl :: (b -> a -> b) -> b -> [a] -> [b]
scanr :: (a -> b -> b) -> b -> [a] -> [b]
```
- `scanl1`和`scanr1`分别将首或尾元素作为初值.

> [!note]
> 多用`fold`和`scan`来减少类似for循环的递归.


### lambda 匿名函数

使用`\`定义lambda, 语法为`\args -> retval`. 用的时候一般用括号将整个匿名函数括起来.

```haskell
>>> zipWith (\x y -> x + y) [1, 2] [10, 100, 1]
[11,102]
>>> map (\x -> x ** x) [1, 2, 3, 4]
[1.0,4.0,27.0,256.0]
```

> [!notice]
> - `\x -> \y -> \z -> x * y * z` 完全等价于 `\x y z -> x * y * z` (类型为`Num a => a -> a -> a -> a`).
> - `f = \x y -> f y x` 完全等价于`f x y = f y x` (类型为`(t1 -> t2 -> t3) -> t2 -> t1 -> t3`).
> 
> 因此不要随意把lambda表达式作为返回值.


### `$` `&` Function Application Operator 函数应用符

`f1 $ f2 x`等价于`f1 (f2 x)`. 语义上是让左边求得函数`f`, 右边求得参数`x`, 然后把`x`传给`f`. 其作用是减少括号的使用.

```haskell
-- 定义
-- 是个中缀函数
($) :: (a -> b) -> a -> b
f $ x = f x
```

一般来说, 用于表示"调用后面的函数":
```haskell
-- 表达的意思是"先翻转line然后再输出"
putStrLn $ reverse line
```

- 具有 **<u>最低</u>优先级**, 因此在表达式中加一个`$`就能将其一刀两断.
- `$`是**右结合**的, 达成分块执行的效果 (从右往左运行`$`隔出来的块).
- 这使得所有分块都会先自己完成计算(最低优先级导致)(Curry或函数组合), 然后从右往左折叠计算(右结合导致)(单纯Curry).

- `f (1 + 1)` 等价于 `f $ 1 + 1`.

`$ x`可以**将变量变成函数**: 接受一个函数, 返回将`x`应用于传入的函数后得到的结果.
```haskell
>>> :t ($ 1)
($ 1) :: Num a => (a -> b) -> b
```

而`&`是把`$`的输入反过来, 入参`x`写在前面, 目标函数`f`写在后面. 语义上是让左边求得参数x, 右边求和函数`f`, 然后把`x`传给`f`. 目标是**让一个参数被连续应用多个函数**, 即`x & f1 & .. & fn`等价于`fn (.. (f1 x))`.

位于`Data.Function`.

```haskell
-- 定义
(&) :: a -> (a -> b) -> b
x & f = f x
```


### `.` Function Composition 函数组合

即**复合函数**$(f \circ g)(x) = f(g(x))$, 要求把`g(x)`的结果作为`f`的自变量进一步求值.

使用`.`来定义复合函数. 使用`f . g`定义$f(g(x))$, 先调用后者. 要求都为**单参**.
```haskell
-- 定义
(.) :: (b -> c) -> (a -> b) -> a -> c
f . g = \x -> f (g x)
```

**优先级仅低于函数调用**, 即`f . g a`会先进行`g a`. 
- 这就导致如果`g`是`a -> b`的话, 会导致此句变成`f . b`, 是错误的. 因此, 需要`(f . g) a`, 或者`f . g $ a`.
- `f . g $ a` 等价于 `f (g a)`, 可见`.`很多时候用于解决括号的嵌套.
- 因为函数调用更优先, 因此对函数进行Curry很方便. 因为`.`只支持单参函数, 所以必须给每个被串联的函数Curry成单参. 
- `.`可以写出**Pointfree Style**, 就是通过一系列单参函数的串联实现的(即定义过程中没有<u>真正的参数</u>出现在<u>代码</u>中).

```haskell
-- 普通暴力写法
func :: (RealFrac a, Integral b, Floating a) => a -> b
func x = ceiling (negate (tan (cos (max 50 x))))
-- Pointfree Style 写法
-- Curry最后一个`max`双参函数的一个参数, 然后串上一堆单参函数.
-- 这一整串定义没有出现`func'`的真实参数, 只有其使用者要传参.
func' :: Double -> Integer
func' = ceiling . negate . tan . cos . max 50
```

```haskell
-- Curry参与到复合的各个函数, 然后使用`$`给复合函数传值, 得到结果值.
-- 真实参数只出现在`$`后面.
oddSquareSum :: Integer  
oddSquareSum = sum . takeWhile (<10000) . filter odd . map (^2) $ [1..]
```



## List 列表

本质是一个单向链表. 无法从后往前迭代.

ghci中可以使用`let a = 1`来定义常量，与脚本中`a = 1`相同。

- 对于字符串来说，`"hello"`仅仅只是`['h','e','l','l','o']`的语法糖，也就是说字符串就是List。
- **合并两个List**，`l1 ++ l2`。实现中会遍历左边的List，对于长字符串或者列表不是很友好。
- 使用`:`可以**把元素和列表合并**，`elem1 : list`。如果要使用`++`连接单个元素到List可以用`[elem1] ++ list`。
- 实际上`[1, 2, 3]`就是`1:2:3:[]`的语法糖，也就是空列表依次在前面插入元素。
- 按照**索引**取List元素：`[1, 2, 3] !! 2`，索引从0开始。越界将报错。
- List内部元素可比较时，可以使用`> >= ==`等运算符比较大小，将会按元素依次比较。

使用`zipWith`来完成"两个List按元素计算"的操作:
```haskell
-- 定义两个列表
list1 :: [Int]
list1 = [1, 2, 3]

list2 :: [Int]
list2 = [4, 5, 6]

-- 使用zipWith将两个列表的元素相加
sumLists :: [Int]
sumLists = zipWith (+) list1 list2  -- 结果是 [5, 7, 9]

main :: IO ()
main = print sumLists  -- 输出: [5, 7, 9]
```

List常用函数：
- `head`返回首部，即首元素，结果是元素，列表为空将触发异常。
- `tail`返回尾部，去掉首个元素后的部分，结果是列表，列表为空将触发异常。
- `last`返回最后一个元素。
- `init`返回除去最后一个元素的部分。
- 上面几个函数用于空列表，将在运行时触发异常，编译时不会检查到。
- `length`得到列表长度。
- `null`判断列表是否为空，返回`True False`。相比`list == []`来判断会更好。
- `reverse`反转列表。
- `take n list`取前n个元素构成的列表。
- `drop n list`去掉前n个元素，得到剩余元素构成的列表。
- `maximum minimum`得到最大和最小的元素。
- `sum product`返回列表所有元素的和与乘积。
- `elem`判断一个元素是否包含与一个list，`elem 10 [1, 2, 10]`，通常以**中缀形式**调用``10 `elem` [1, 2, 10]``（需要加**反引号**）。

### List Comprehension

`[expression | ranges and constrait]` 生成满足要求的, 加工后的列表.

```haskell
[x * 2 | x <- [1..10]]
-- [2,4,6,8,10,12,14,16,18,20]
```

```haskell
[[x, y] | x <- [1..10], y <- [10..20], x + y == 20]
-- [[1,19],[2,18],[3,17],[4,16],[5,15],[6,14],[7,13],[8,12],[9,11],[10,10]]
```

这是一个语法糖, 最后会被转化成 [List Monad](Haskell/Haskell.md#List%20Monad) 操作.

### Non-deterministic 视角

List可以被看做是 **non-deterministic 的计算**, 即计算结果不知道是什么(范围是里面的所有元素). 那么两个 non-deterministic 的计算互相再计算的话, 就更加具有不确定性了. 

例如, 3长度的两个List相乘得到的是9个元素:
```haskell
ghci> [ x*y | x <- [2,5,10], y <- [8,10,11]] 
-- 使用 applicative style 的话更明显
ghci> (*) <$> [2,5,10] <*> [8,10,11][16,20,22,40,50,55,80,100,110]
```

两个长度为2的List的元组组合生成长度为4的List:
```haskell
listOfTuples :: [(Int,Char)]  
listOfTuples = do  
    n <- [1,2]  
    ch <- ['a','b']  
    return (n,ch) 
-- [(1,'a'),(1,'b'),(2,'a'),(2,'b')] 
```

### NonEmpty

表示非空List的typeclass. 使用`:|`运算符构造, 它等价于一次`:`, 毕竟一次元素添加操作意味着产出的List必然非空.

位于`Data.List.NonEmpty`.

```haskell
type NonEmpty :: * -> *
data NonEmpty a = a :| [a]
```


## Range


- `[1..20]`即表示`[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]`。
- 给出指定第二个元素将使用第一个和第二个元素的间隔作为步长，默认是1`[1,3..20]`表示`[1,3,5,7,9,11,13,15,17,19]`。
- 步长可以为负，到达不了上限则生成列表为空：`[1,0..10]`为`[]`。

- 不指定上限则**无限长度**，步长为0同样也是无限长度。无限长度列表将会在使用到时才求值。
- `cycle list`可以生成循环list元素的无限列表。
- `repeat value`生成一个仅包含该值的无限列表。
- `replicate n value`重复一个值n次。


## let

`let`可以定义局部变量. 

`let [binding] in [expressions]`
- 没`in`时, 就是简单的定义局部变量, 有局部生命周期.
- 有`in`时, 在`binding`中绑定的名字**仅在`expressions`中可见**。

如果要在一行中绑定多个名字, 如果要将多个名字排成一行可以用`;`隔开.

```haskell
main :: IO ()
main = do
	-- 局部变量
	let x = 5
	    y = 10
	-- 局部函数
    print (x + y) -- 15
	let square n = n * n
	print (square 4) -- 16
```

```haskell
calcBmis :: (RealFloat a) => [(a, a)] -> [a]  
calcBmis xs = [bmi | (w, h) <- xs, let bmi = w / h ^ 2, bmi >= 25.0]
```

可以写一长串:
```haskell
roadStep :: (Path, Path) -> Section -> (Path, Path)  
roadStep (pathA, pathB) (Section a b c) =   
    let priceA = sum $ map snd pathA  
        priceB = sum $ map snd pathB  
        forwardPriceToA = priceA + a  
        crossPriceToA = priceB + b + c  
        forwardPriceToB = priceB + b  
        crossPriceToB = priceA + a + c  
        newPathToA = if forwardPriceToA <= crossPriceToA  
                        then (A,a):pathA  
                        else (C,c):(B,b):pathB  
        newPathToB = if forwardPriceToB <= crossPriceToB  
                        then (B,b):pathB  
                        else (C,c):(A,a):pathA  
    in  (newPathToA, newPathToB)  
```

### 有 `in` 的情况

有`in`时, let定义的东西就完全被交给了后面的`expressions`, 返回求得的值, 且定义的临时变量无法被外部访问. 

```haskell
main :: IO ()
main = do
  let r = let square x = x * x in (square 2, square 3)
  print r -- (4, 9)
```

```haskell
>>> let a = 100; b = 20 in a + b
120
```

```haskell
>>> let (a, b) = (100, 20) in a + b
120
```

## pattern matching 模式匹配

赋值就会发生模式匹配, 分解数据到其他变量中.

### case表达式

语法：
```haskell
case expression of pattern1 -> result1
                   pattern2 -> result2
                   pattern3 -> result3
                   ...
```

使用case检测List是否递增：
```haskell
increasing' :: Ord a => [a] -> Bool 
increasing' xs = case xs of 
  (x:y:ys) -> x <= y && increasing' (y:ys) 
  _ -> True
```

> [!tip]
> 函数参数模式匹配只能用于函数定义时，而`case`表达式可以用于任何地方。


### 函数模式匹配

函数的模式匹配是`case`表达式的语法糖.

可以定义一个函数, 把函数名写很多次, 每一次的参数是不同的特殊形式 (如`[a]`或者`(a, b)`, 也可以是常数). 当把一个值传给这个函数时, 就会**按序**匹配**符合要求**的项, 值也被分解到该项的参数中. 类似match.

因为必须要有返回值, 因此必须要让模式匹配**exhaustive**.

```haskell
describeNumber :: Int -> String
describeNumber 0 = "Zero"
describeNumber 1 = "One"
describeNumber 2 = "Two"
describeNumber _ = "Others"
```

定义单分支就相当于一次分解赋值:
```haskell
-- 对任意三元素元组取第二项
second :: (a, b, c) -> b
second (_, y, _) = y
```

因为`:`可以连接元素和List, 因此也可以在模式匹配中用来分离List两端的元素.

```haskell
-- 递归求list长度
lengthList :: [a] -> Int
lengthList [] = 0  -- 空列表
lengthList (_:xs) = 1 + lengthList xs -- 取出一个元素就+1
```

使用模式匹配检查数组是否递增:
```haskell
increasing' :: Ord a => [a] -> Bool 
increasing' (x:y:ys) = x <= y && increasing (y:ys)
increasing' _ = True
```

### Guard 守卫

使用guard来进行**判断分支**. <u>只能用于模式匹配</u>的一项当中. 使用等号.

可以使用永真函数`otherwise`来让其exhaustive. 如果当前的guard尚未exhaust, 那么**剩下的情况都会交给下一个模式匹配**.

如果只有单个分支, 就可以看成是模式匹配的一项的额外检查, 暴力的一个例子:
```haskell
grade :: Int -> String
grade score = case score of
    s | s >= 90 -> "优秀"      -- 使用守卫检查分数
    s | s >= 80 -> "良好"
    s | s >= 70 -> "中等"
    s | s >= 60 -> "及格"
    _           -> "不及格"   -- 默认情况
```

检查数组非空:
```haskell
checkLen :: [[Integer]] -> Bool
checkLen p
  | not (null p) = True
  | otherwise = False
```

使用guard判断数组递增:
```haskell
increasing' :: (Ord a) => [a] -> Bool
increasing' xs
    | null xs = True
    | head xs <= head (tail xs) = increasing' (tail xs)
    | otherwise = False
```

```haskell
-- 定义一个代数数据类型
data Person = Person String Int  -- Person 包含姓名和年龄

-- 定义一个函数，根据年龄返回不同的描述
describePerson :: Person -> String
describePerson (Person name age)
    | age < 0 = "年龄不能为负数"
    | age < 18 = name ++ " 是未成年人"
    | age < 65 = name ++ " 是成年人"
    | otherwise = name ++ " 是老年人"
```

```haskell
-- 定义一个代数数据类型
data Shape = Circle Float | Rectangle Float Float

-- 定义一个函数，根据形状计算面积
area :: Shape -> String
area shape = case shape of
    Circle r 
        | r < 0     -> "半径不能为负数"
        | otherwise -> "圆的面积是: " ++ show (pi * r * r)
    Rectangle width height
        | width < 0 || height < 0 -> "宽度和高度不能为负数"
        | otherwise               -> "矩形的面积是: " ++ show (width * height)
```

### 利用where

在函数进行模式匹配前, 可以**定义或预处理数据**, 使用`where`实现.

```haskell
bmiTell :: (RealFloat a) => a -> a -> String  
bmiTell weight height  
    | bmi <= skinny = "You're underweight, you emo, you!"  
    | bmi <= normal = "You're supposedly normal. Pffft, I bet you're ugly!"  
    | bmi <= fat    = "You're fat! Lose some weight, fatty!"  
    | otherwise     = "You're a whale, congratulations!"  
    where bmi = weight / height ^ 2  
          skinny = 18.5  
          normal = 25.0  
          fat = 30.0
```

## 递归

无状态的纯函数编程无法使用循环, 因此必须使用递归.

```haskell
-- 求List最大项
maximum' :: (Ord a) => [a] -> a
maximum' [] = error "maximum a empty list"
maximum' [x] = x
maximum' (x:xs) = max x (maximum' xs)
```

```haskell
-- 把一个数值重复数次组成List
replicate' :: (Ord t, Num t) => t -> a -> [a]
replicate' n x
    | n <= 0 = []
    | otherwise = x:replicate' (n-1) x
```

```haskell
-- 取List的前数个元素组成新List
take' :: (Ord a1, Num a1) => a1 -> [a2] -> [a2]
take' n _
    | n <= 0 = []
take' _ [] = []
take' n (x:xs) = x : take' (n-1) xs
```

```haskell
-- 判断元素是否在List中
elem' :: Eq t => t -> [t] -> Bool
elem' a [] = False 
elem' a (x:xs)
    | x == a = True 
    | otherwise = a `elem'` xs
```

递归时要注意最好有**尾递归**优化. 原本是循环迭代的算法写成递归形式似乎都能是尾递归, 毕竟不可能"回归"上次循环. 例如斐波那契数列的迭代版本($O(n)$复杂度)就能写成:
```haskell
fibonacci :: Integral a => a -> Integer -> Integer -> Integer
fibonacci 0 a b = b
fibonacci n a b = fibonacci (n - 1) (a + b) a
fib' :: Integral a => a -> Integer
fib' n = fibonacci n 1 0
```


## I/O

### IO Action

> [!info]
> IO Action 是一个**Monad**, 更是一个MonadFail和MonadPlus.

```haskell
main :: IO ()
main = do
    putStrLn "hello, input your name:"
    name <- getLine 
    putStrLn ("Hey ! " ++ name ++ " Yo ! what's up !")
```

一个IO action是一个会造成副作用的动作, 常常是读取**输入<u>或者</u>输出**到屏幕, 同时会返回值. `IO`这个type constructor是单参数的, 而其type参数即为IO action的**返回值type**. IO action像盒子一样包装着返回值, 只能通过绑定来读取返回值.

> [!info]
> `main`函数要求返回`IO ()`, 因此要么在最后一行写输出, 要么使用`return ()`来满足其要求.

在标准输出打印字符串没有具体的值返回, 用`()`(即unit)代表.
```haskell
Prelude> :t putStrLn
putStrLn :: String -> IO ()
Prelude> :t putStrLn "hello" 
putStrLn "hello" :: IO ()
```

读取输入的`getLine`函数, 其type参数就是String:
```haskell
ghci> :t getLine 
getLine :: IO String
```

使用`<-`把IO action的结果**绑定**到一个名字上, 例如`name <- getLine`. 不使用返回值, 因为这样就能防止出现"调用两次同一个函数却得到了不同的返回值"的<u>不纯</u>情况. 一个IO action必须使用`<-`后, <u>才能影响普通程序部分</u>(普通部分没有IO类型, 因此直接使用的话无法过类型检查).

> [!tip]
> `putStrLn`没有返回值因此也不会对其绑定.

一个IO action会在被**传给main**并且执行程序的时候**触发**. TODO: 待测试


### return


使用`return`**函数**把一个变量使用IO action包裹.

```haskell
-- 这么做没什么意义, 效果等价于 let a = "hello"
a <- return "hello"
```

可以在一些返回值为IO action的函数里面生成返回值.

例: 封装一个读取函数, 它读两行然后拼成一行输出:
```haskell
myAction :: IO String  
myAction = do  
    a <- getLine  
    b <- getLine  
    return $ a ++ b

-- 或者使用applicative style
myAction = (++) <$> getLine <*> getLine
```

例: 一行一行地读输入, 一读到就按词翻转后输出:
```haskell
main = do
    line <- getLine
    if null line
	    -- 读到空行, 退出
	    -- main 要求返回`IO ()`, 因此这里使用`return ()`
        then return ()
        else do
            putStrLn $ reverseWords line
            main -- 递归

-- 按词翻转
reverseWords :: String -> String
reverseWords = unwords . map reverse . words
```

### sequence

`sequence`函数可以用于把多个IO action串成一个, 从而简化多步骤的IO操作的编写(包括输入输出).

```haskell
main = do
    rs <- sequence [getLine, getLine, getLine]
    print rs

-- 等价于分开写
main = do
    a <- getLine
    b <- getLine
    c <- getLine
    print [a,b,c]
```


# 特殊 typeclass

## Semigroup

[Data.Semigroup](https://downloads.haskell.org/ghc/latest/docs/libraries/base-4.20.0.0-1f57/Data-Semigroup.html)

Semigroup(**半群**)要求存在 <u>参数和返回值类型都是它(Closure封闭的)</u> 的一个二元函数拥有**结合律(associativity)**. 

提供`<>`的实现即可.

```haskell
class Semigroup a where
        (<>) :: a -> a -> a
        sconcat :: NonEmpty a -> a
        stimes :: Integral b => b -> a -> a
```

`<>`就是所要求的封闭的**结合律**的二元函数.
```haskell
ghci> Just [1, 2, 3] <> Just [4, 5, 6]
Just [1,2,3,4,5,6]
```

`sconcat` 把非空的List(`[a]`) 使用`<>`折叠成单一变量`a`.
```haskell
ghci> :m Data.List.NonEmpty
ghci> sconcat $ Just [1, 2, 3] :| [Nothing, Just [4, 5, 6]]
Just [1,2,3,4,5,6]
```

`stimes` 把输入`a`重复`b`次, 使用`<>`折叠. 复杂度是$O(\log n)$. 库中提供了若干`stimesXxx`函数作为替换实现, 以在特殊情景(约束)下<u>降低复杂度</u>.
```haskell
ghci> stimes 4 [1]
[1,1,1,1]
```

> [!tip]
> 当一个type有多种满足Semigroup的二元函数时, 需要使用`newtype`分别实现. Monoid也一样.

### Semigroup Law

Associativity:
```haskell
(x <> y) <> z = x <> (y <> z)
```

## Monoid

[Data.Monoid](https://downloads.haskell.org/ghc/latest/docs/libraries/base-4.20.0.0-1f57/Data-Monoid.html)

Monoid(**幺半群**) 基于Semigroup, 只比它多了Identity的要求. 它要求一个type:
- *Semigroup的要求*: 存在 <u>参数和返回值类型都是它(Closure封闭的)</u> 的一个二元函数拥有**结合律(associativity)**;
- *额外要求*: 该type存在某个**值**(这个值称为<u>此函数的</u>**identity(幺元)**), 当其为此二元函数的其中一个参数的时候, <u>计算结果等于另一个参数</u>.

```haskell
type Monoid :: * -> Constraint
class Semigroup a => Monoid a where
  mempty :: a
  
  -- Use `<>` instead of `mappend`.
  mappend :: a -> a -> a
  mappend = (<>)
  
  mconcat :: [a] -> a 
  mconcat = foldr mappend mempty
```

`mempty`不接受任何参数, 因此是个<u>常数</u>, 它表示此Monoid的**identity**.

`mappend`等价于`<>`, **不应当使用**, 它未来会被删除.

`mconcat`是把该Monoid type的一个List合成单个结果(依然属于该type). 有<u>默认实现</u>, 是使用`mappend`从`mempty`开始给**fold**成结果.

### Monoid Law

Right Identity:
```haskell
x <> mempty = x
```

Left Identity:
```haskell
mempty <> x = x
```

Associativity (Semigroup's law)
```haskell
(x <> y) <> z = x <> (y <> z)
```

Concatenation:
```haskell
mconcat = foldr (<>) empty
```


### 例子

> 使用`<>`替换了所有示例的`mappend`, 但为了方便, 定义仍然使用`mappend`.

List是Monoid, 因为`[]`与任意List进行`++`都得到那个List本身; `++`不需要考虑顺序.
```haskell
instance Monoid [a] where  
    mempty = []  
    mappend = (++)  
```

---

`Data.Monoid`有**Product**和**Sum**两个type, 分别实现**乘法**(`*` 和 `1`)和**加法**(`+` 和 `0`)的Monoid. Product的定义和Monoid实现:
```haskell
newtype Product a =  Product { getProduct :: a }  
    deriving (Eq, Ord, Read, Show, Bounded)  

instance Num a => Monoid (Product a) where  
    mempty = Product 1  
    Product x `mappend` Product y = Product (x * y)  
```

其使用:
```haskell
ghci> getProduct $ Product 3 <> Product 9  
27  
ghci> getProduct $ Product 3 <> mempty  
3  
ghci> getProduct $ Product 3 <> Product 4 <> Product 2  
24 
-- 使用 `map Product` 将其变成`[Product]`后,
-- 进行mconcat, 再把结果(单一的Product)转回来.
ghci> getProduct . mconcat . map Product $ [3,4,2]  
24  
```

---

Bool也有两种Monoid. 使用**Any**表示`False`不影响`||`, 使用**All**表示`True`不影响`&&`.
```haskell
newtype Any = Any { getAny :: Bool }  
    deriving (Eq, Ord, Read, Show, Bounded)  

instance Monoid Any where  
    mempty = Any False  
    Any x `mappend` Any y = Any (x || y)  
```

```haskell
newtype All = All { getAll :: Bool }  
        deriving (Eq, Ord, Read, Show, Bounded)  

instance Monoid All where  
        mempty = All True  
        All x `mappend` All y = All (x && y)  
```

Any type的使用:
```haskell
ghci> getAny $ Any True <> Any False  
True  
ghci> getAny $ mempty <> Any True  
True  
ghci> getAny . mconcat . map Any $ [False, False, False, True]
True  
ghci> getAny $ mempty <> mempty  
False  
```

---

`Ordering` type可以是Monoid, 其实现为:
```haskell
instance Monoid Ordering where  
    mempty = EQ  
    LT `mappend` _ = LT  
    EQ `mappend` y = y  
    GT `mappend` _ = GT  
```

这表示如果当前比较**不为**`EQ`的话, 就立刻返回结果, 否则返回另一个参数. 可以用于构建**有优先级顺序的比较**, 如字典序等.

例如先比较字符串长度再比较其内容:
```haskell
import Data.Monoid

strCompare :: String -> String -> Ordering  
strCompare x y = (length x `compare` length y) <>  
                    (x `compare` y)  

-- 等价于下面这个更复杂的写法
strCompare' :: String -> String -> Ordering  
strCompare' x y = let a = length x `compare` length y   
                        b = x `compare` y  
                    in  if a == EQ then b else a  
```

---

当`a`为Monoid时, `Maybe a` 也可以定义成 Monoid, 其使用`Nothing`作为identity, **使用`a`的二元函数作为其二元函数**.

```haskell
instance Monoid a => Monoid (Maybe a) where  
    mempty = Nothing  
    Nothing `mappend` m = m  
    m `mappend` Nothing = m  
    Just m1 `mappend` Just m2 = Just (m1 `mappend` m2)  
```

这使得被`Maybe`包裹的两个Monoid的`mappend`**不需要手动解包**. 例如:
```haskell
ghci> Nothing <> Just "andy"  
Just "andy"  
ghci> Just LT <> Nothing  
Just LT  
ghci> Just (Sum 3) <> Just (Sum 4)  
Just (Sum {getSum = 7}) 
ghci> getFirst . mconcat . map First $ [Nothing, Just 9, Just 10]  
Just 9  
```

`First` type 则让`Maybe`换了一种Monoid实现, 让它**永远返回第一个`Just x`**.
```haskell
newtype First a = First { getFirst :: Maybe a }  
    deriving (Eq, Ord, Read, Show) 

instance Monoid (First a) where  
    mempty = First Nothing  
    First (Just x) `mappend` _ = First (Just x)  
    First Nothing `mappend` x = x  
```

`Last`也是类似的定义但**永远返回最后一个`Just x`**.

## Functor

[Control.Monad (Functor)](https://downloads.haskell.org/ghc/latest/docs/libraries/base-4.20.0.0-1f57/Control-Monad.html)

Functor(**函子**)是一个typeclass, 只有kind为`* -> *`的(即单参数的)type constructor才能实现它. 它要求可以对内部包含的变量进行操作(称作 **map over**), 且操作返回类型是任意类型.

只需要实现`fmap`.

```haskell
ghci> :i Functor
type Functor :: (* -> *) -> Constraint
class Functor f where
  -- 对内部变量进行操作
  fmap :: (a -> b) -> f a -> f b
  -- 把变量塞入内部
  (<$) :: a -> f b -> f a
```

> [!tip]
> 这种map over操作是彻底的, 例如List `[a]`, 在加工过后一个`a`都不能留, 才能变成`[b]`.

由于只能接收单参数的type constructor, 因此需要把多参type constructor给Curry了才能实现, 例如`Either`只能实现`instance Functor (Either a) where`.

`fmap`可以被理解为对Functor的内部元素进行映射变换, 但也可以理解为是一种**lifting操作**. 显然, 它接受<u>一个</u>function, 然后返回这样的一个function: 接受一个Functor, 返回加工后的Functor. 也就是说, `fmap`是一个把`a -> b`变成`f a -> f b`的**lifting操作**.

```haskell
-- fmap把 `a -> [a]` lift到了 `f a -> f [a]`
ghci> :t replicate 3
replicate 3 :: a -> [a]
ghci> :t fmap (replicate 3)
fmap (replicate 3) :: Functor f => f a -> f [a]
```

> [!tip]
> 我们可以把 functor 看作**输出具有 context 的值**。例如说 Just 3 就是输出 3，但他又带有一个可能没有值的 context。\[1,2,3\] 输出三个值，1,2 跟 3，同时也带有可能有多个值或没有值的 context。(+3) 则会带有一个依赖于参数的 context。
> 
> 如果你把 functor 想做是输出值这件事，那你可以把 map over 一个 functor 这件事想成**在 functor 输出的后面再多加一层转换**。当我们做 fmap (+3) \[1,2,3\] 的时候，我们是把 (+3) 接到 \[1,2,3\] 后面，所以当我们查看任何一个 list 的输出的时候，(+3) 也会被套用在上面。另一个例子是对函数做 map over。当我们做 fmap (+3) (*3)，我们是把 (+3) 这个转换套用在 (*3) 后面。这样想的话会很自然就会把 fmap 跟函数合成关联起来（fmap (+3) (*3) 等价于 (+3) . (*3)，也等价于 \x -> ((x*3)+3)），毕竟我们是接受一个函数 (*3) 然后套用 (+3) 转换。最后的结果仍然是一个函数，只是当我们喂给他一个数字的时候，他会先乘上三然后做转换加上三。这基本上就是函数合成在做的事。

使用`<$>`来当做`fmap`的中缀形式:
```haskell
(<$>) :: (Functor f) => (a -> b) -> f a -> f b  
f <$> x = fmap f x  
```


### Functor Law

Functor应当满足两个Functor Law. 这是设计上的要求, 不会被编译器检查.

Identity: 
```haskell
fmap id = id
``` 

意为Functor应当满足: 对一个Functor使用`id`函数进行`fmap`, 等价于对Functor本身调用`id`函数. 简单来说, 使用`id`的`fmap`应该什么都不做.

> [!note]
> 这防止了额外固定操作. 例如, `Maybe`应当在`Nothing`的时候什么都不做, 但如果在进行`fmap`的时候, 把`Nothing`全换成某个默认值, 那也<u>不会违背</u>Functor typeclass, 但是<u>违背</u>了 Functor Law 1. 显然这种替换完全不符合使用者的本意. `fmap`时固定改变其他变量的值也违背了本意. 

Composition: 
```haskell
fmap (f . g) = fmap f . fmap g
```

或者说对任意Functor `F`, 有`fmap (f . g) F = fmap f (fmap g F)`. 这就要求对Functor的多次`fmap`等价于所有操作函数的复合函数的单次`fmap`. 

> [!note]
> 这防止了内部操作的行为不完全局限于传入的操作函数. 例如, 如果`fmap g`会让元素`x`变成`g x + 1`, 那么再次`fmap f`的时候就会得到`(f $ g x + 1) + 1`; 但是`fmap (f . g)` 得到的是`(f g x) + 1`, 显然不等价. 应当老老实实应用函数, 不允许有额外的操作.

### 例子

**IO action** 可以对内部的值进行操作, 最后还是能通过`return`得到新的IO action. 它以Functor的形式实现这个功能:
```haskell
instance Functor IO where
    fmap f action = do
        result <- action
        return (f result)

-- 直接读到翻转后的行
line <- fmap reverse getLine

-- 更复杂的加工, 利用function combination
line <- fmap (intersperse '-' . reverse . map toUpper) getLine 
```

---

**function combination** 把一个`a -> b`的函数变成一个`a -> c`的函数, 只需要给出一个`b -> c`的函数. 显然这也是个Functor, 它定义成: 把原函数`g :: r -> a`应用参数`x :: r`的结果变成目标函数`f :: a -> b`的参数, 从而最终得到`r -> b`.
```haskell
instance Functor ((->) r) where  
    fmap f g = (\x -> f (g x))  
    -- 或者直接写成
    -- fmap = (.)
```

> [!tip]
> `(->) r`可以看做`F a`, 因此fmap是对`a`(返回值)做加工.

可以认为`g x`这一表达式是为了取出`g`的返回值, 然后把它扔给`f`加工.

目标函数为双参函数的例子. 此时就可以看出, `g x`只是被用来Curry `f` 的第一个参数. 从Functor的类型角度分析, (下面的type variable实际上是一样的, 只是为了区分) 其中`(+) :: a -> b -> c`被看做是`a -> (b -> c)`的"单参函数", 用其对`*100 :: d -> a`的结果进行加工, 即加工`a`, 最终成为`d -> (b -> c)`, 是一个就算执行了乘法后还需要等待一个参数以执行加法的双参函数, 即`x * 100 + y`.
```haskell
ghci> let simple = (+) <$> (*100)
ghci> :t simple
simple :: Num a => a -> a -> a
ghci> simple 4 5
405
```

---

Functor的内部可以是一个**函数**, 例如函数List:
```haskell
[\x -> x + 1, \x -> x * 2] :: Num a => [a -> a]
```

如果fmap的操作函数是一个多参函数的话, 就可以让Functor内部的变量变成函数(变量用来Curry了). 例如从`[a]`使用`a -> a -> a`使其变成`[a -> a]`:
```haskell
fmap (*) [1,2,3,4] :: Num a => [a -> a]
```

对Functor内函数(`a -> b`)传`xxx` (`xxx :: a`)作为**参数**时, 可以采用`fmap (\f f xxx)` 来完成, 如:
```haskell
ghci> let a = fmap (*) [1,2,3,4]  
ghci> :t a  
a :: [Integer -> Integer] 
-- 对任一内部函数, 把`9`传进去
ghci> fmap (\f -> f 9) a  
[9,18,27,36]  
```

## Applicative

[Control.Applicative](https://hackage.haskell.org/package/base-4.20.0.1/docs/Control-Applicative.html)

> [!info]
> 其完整名称应该叫 Applicative Functor, 形容词 + 名词.

对于一个内部为函数的Functor `F (a -> b)`, 函数的传参可以使用`fmap (\f f xxx)`来完成; 但是, 如果没有`xxx: a`, 而只有另一个Functor `yyy :: G a`呢? 换句话说, 如何对`Functor (a -> b)`以`Functor a`进行map over? 最暴力的解法是手动模式匹配, 但显然不美观.

Applicative typeclass实现了这个功能. **只有实现了Functor的type才能实现Applicative**. 同样只能接收单参数的type constructor.

只需同时实现`pure`和`<*>`.

> [!tip]
> Applicative表示有一个拥有context的type, 可以对其进行操作并保留context. 因为全程都在<u>同一个Applicative</u>的包裹下, 因此context是一致的.

```haskell
ghci> :i Applicative
type Applicative :: (* -> *) -> Constraint
class Functor f => Applicative f where
  pure :: a -> f a
  (<*>) :: f (a -> b) -> f a -> f b
  GHC.Base.liftA2 :: (a -> b -> c) -> f a -> f b -> f c
  (*>) :: f a -> f b -> f b
  (<*) :: f a -> f b -> f a
```

`pure` 可以**把一个变量塞进Applicative Functor里面**; 或者说: 把一个普通值放到一个默认 context 下, 且它是<u>能包含这个值的最小的context</u>. 

`<*>` (叫做**apply**函数) (**左结合**)把`f a`应用到了`f (a -> b)`上, 产生了`f b`, 也就是上面所说的功能. 但也可以看做是把`f (a -> b)`给变成`f a -> f b`, 即把Applicative内部的函数**lift**成Applicative之间的函数. 

> [!notice]
> 同一个Applicative才能apply. 也因此, 一个type的apply行为会非常明确, 可以通过看其实现源码来确定.

`liftA2`提供了更方便的`f a -> f b -> f c`函数的获取方式. 换句话说, Applicative**都能使用**形如`f a -> f b -> f c`的东西. 详见[liftA2](Haskell/Haskell.md#liftA2).


### Applicative Style

`pure f <*> x` **等价于** `fmap f x`, 得到的都是被Functor包裹的被传了参数`x`的函数`f`. 

当`f`包含多个参数时, 其连续的apply相当于把各个Functor的内容按序传给(Curry)了`f`. 

```haskell
ghci> pure (+) <*> pure 3 <*> pure 5
8
```

显然对于参数足够多的`f`, 对Functor包裹的参数`x`, `y`等, 有下面三个等价写法. 
```haskell
-- 先让`pure f`原地造出 Applicative Functor
pure f <*> x <*> y <*> ...
-- 先使用`fmap f x`造出 Applicative Functor
fmap f x <*> y <*> ...
-- 利用`<$>`. Applicative style.
-- 这些都相当于`f x y ...`的Functor参数版本.
fmap f <$> x <*> y <*> ...
```

其中最后一个利用了`<$>`写出了**applicative style**, 即开头的`f`只是个普通函数, 然后`<$>`起手, 接着连续`<*>`, 其效果就像是把所有Functor里面的东西拿出来传参给`f`.

> [!tip]
> Applicative Style 依然要求所有参数使用同一个Applicative Functor.


### Applicative Law

Identity: 
```haskell
pure id <*> v = v
```

Composition:
```haskell
pure (.) <*> u <*> v <*> w = u <*> (v <*> w)
```

Homomorphism:
```haskell
pure f <*> pure x = pure (f x)
```

Interchange:
```haskell
u <*> pure y = pure ($ y) <*> u
```

### 例子

`Maybe`的Applicative实现:
```haskell
instance Applicative Maybe where  
    pure = Just  
    Nothing <*> _ = Nothing  
    (Just f) <*> something = fmap f something
```

可见, 其`<*>`实现是在内部使用<u>模式匹配</u>把`a -> b`提取出来之后, `fmap`到`f a`里面. 

只要apply的**任一方**为`Nothing`, 则结果为`Nothing`. 因为如果是主动方为`Nothing`, 则无法提取需要用于`fmap`的函数; 如果被动方为`Nothing`, 则`fmap`结果为`Nothing`.

```haskell
ghci> let mayCal = (+) <$> Just 1
-- mayCal :: Num a => Maybe (a -> a)
-- 相当于值为 Just (1+)
ghci> mayCal <*> Just 3
Just 4
```

---

List的apply会生成所有可能的组合, 这可以使用[Non-deterministic的视角](Haskell/Haskell.md#Non-deterministic%20视角)来理解.

```haskell
instance Applicative [] where  
    pure x = [x]  
    fs <*> xs = [f x | f <- fs, x <- xs] 
```

```haskell
-- 等价于 [x y | x <- [(+3),(*2)], y <- [7,8]]
ghci> [(+3),(*2)] <*> [7,8]
[10,11,14,16]
```

```haskell
-- 先生成了`[10+, 20+, 10*, 20*]`, 然后分别应用到3和4上
ghci> [(+),(*)] <*> [10,20] <*> [3,4] 
[13,14,23,24,30,40,60,80]
```

它的 applicative style:
```haskell
ghci> (++) <$> ["ha","heh","hmm"] <*> ["?","!","."]  
["ha?","ha!","ha.","heh?","heh!","heh.","hmm?","hmm!","hmm."] 
-- 等价于: [ x y | x <- [2*,5*,10*], y <- [8,10,11]]
ghci> (*) <$> [2,5,10] <*> [8,10,11] 
[16,20,22,40,50,55,80,100,110]
-- 等价于: [ x y | x <- [1:,2:,3:], y <- [[4,5,6],[7,8,9]]]
ghci> (:) <$> [1, 2, 3] <*> [[4,5,6],[7,8,9]]
[[1,4,5,6],[1,7,8,9],[2,4,5,6],[2,7,8,9],[3,4,5,6],[3,7,8,9]]
```

**ZipList** type 是List的一个包装(包一层单参 type constructor), 位于`Control.Applicative`, 目的是以另一种形式实现Applicative, 让apply函数变成**一一对应匹配(和`zipWith`一样)** 而不是完全组合. 使用`getZipList`从中取出所含的List.

```haskell
instance Applicative ZipList where  
	pure x = ZipList (repeat x)  
	ZipList fs <*> ZipList xs = ZipList (zipWith (\f x -> f x) fs xs)  
```

```haskell
ghci> :m Control.Applicative
ghci> let zl1 = ZipList [1,2,3]
ghci> let zl2 = ZipList [50,100,150]
ghci> getZipList $ (+) <$> zl1 <*> zl2
[51,102,153]
```

---

IO action也是个Applicative, 可以对内部返回值进行直接操作. 例如对两次读取直接进行拼接从而产生新的IO action:
```haskell
myAction :: IO String  
myAction = (++) <$> getLine <*> getLine  
```


### 函数(`->`)的Applicative

`(->) r`要求传入`f :: r -> a -> b` (即`(->) r (a -> b)`), 以生成这样一个函数: 对于传入的参数`x :: r`, 先计算原函数`y = g x :: a`(`g :: r -> a`), 再计算`f x y :: b`, 从而最终得到`r -> b`. 

```haskell
instance Applicative ((->) r) where  
	-- 永远返回x的最小context
    pure x = (\_ -> x)  
    -- g对参数的计算结果被传到f上
    f <*> g = \x -> f x (g x)  
```

从Applicative的角度在类型上分析, `f` 是 `a -> (b -> c)`, 是包裹了<u>函数</u> `b -> c`的Functor. `g` 是 `a -> b`, 是包裹了<u>变量</u>`b`的普通Functor. Applicative 把 `f` 给apply到 `g` 上, 就得到了 `a -> c`. 

**总结来说**, `f x`得到Functor包含的函数, `g x`则得到Functor包含的变量; 要将该变量传给函数完成转化, 则为`f x (g x)`.

<u>连续</u>的apply就像是在`f x`后面一直加函数对应的**匿名函数表达式**(正如普通apply在其后添加变量一样). 但<u>函数类型上</u>, 函数参数一直被Curry, 会<u>每次apply缩减一个</u>, 直到<u>剩余一个</u>各匿名函数表达式通用的参数.

f的<u>首个参数</u>可以通过`fmap`一个函数来实现. `fmap f g`得到`f'`为`f (g x)`, 那么接下来的apply`f' <*> h` 就得到 `f' x (h x)`, 也就是`f (g x) (h x)`. 

因此, **applicative style** 就体现为这样的功能: 把后面的所有函数逻辑都绑到首个多参函数的各个参数上.

例如, 对双参函数`(+)`, 把单参函数 `(+3)` 和 `*100` 对应的匿名函数作为`(+)`的两个参数, 形成了`(x + 3) + (x * 100)`.
```haskell
-- (+) <$> (+3) :: Num a => a -> a -> a
ghci> let cal = (+) <$> (+3) <*> (*100)
ghci> :t cal
cal :: Num b => b -> b
ghci> cal 5
508
```

利用`(,,)`把参数元组化, 可以看出在applicative style下被足够Curry的此函数变成了`a -> (a, a, a)`, 其中元组的每一项都是<u>按序绑定</u>的对应函数将会计算出来的.
```haskell
ghci> let funcList = (,,) <$> (+3) <*> (*2) <*> (/2)
-- funcList :: Fractional a => a -> (a, a, a)
ghci> funcList 5
(8.0,10.0,2.5)
```

### liftA2

其内部实现就是applicative style, 内涵一个函数和两个变量. 其参数为**一个双参函数和两个 Applicative Functor**. 其目的是将其封装为一个lifting操作, 即给出普通双参函数, 将其lift成 Applicative Functor的双参函数. 或者说, `liftA2`使一个双参函数可以直接<u>无视Functor的包装</u>, 直接对内部进行操作.

```haskell
liftA2 :: (Applicative f) => (a -> b -> c) -> f a -> f b -> f c 
liftA2 f a b = f <$> a <*> b
```

`(<*>) = liftA2 id`.

例如, 有`3 : [4,5] = [3,4,5]`, 那么想要把`Just 3`和`Just [4,5]`合并, 就需要使用`liftA2`把`:`给lift一下:
```haskell
ghci> liftA2 (:) (Just 3) (Just [4,5])
Just [3,4,5]
```

### sequenceA

把元素为Applicative的List `[f a]` 给变成 普通元素的List的Applicative `f [a]`.

```haskell
sequenceA :: (Applicative f) => [f a] -> f [a]  
sequenceA [] = pure []  
sequenceA (x:xs) = (:) <$> x <*> sequenceA xs  
```

可见, 实现方式是通过递归, 使用`x:xs`把元素一个一个取出来, 使用`(:) <$> x`将其变成了 包含一个把`x`合并到新接收到的`[a]`上的函数的Applicative. 而其本身也返回的是`Applicative [a]`, 因此可以通过递归调用来将子串的返回值进行apply来完成目标功能.

也可以使用fold实现该功能. 使用`liftA2 (:)`把`:` lift成了能处理Applicative包裹下的列表操作的函数, 于是就能简单地使用fold, 最后也能得到Applicative包裹下的结果列表.
```haskell
sequenceA = foldr (liftA2 (:)) (pure [])
```

> [!tip]
> - 首先注意Functor的`fmap`的行为, 确定`(:) <$> x`会得到什么样的`f a`. 例如`Maybe`的话如果`x`为`Nothing`那就会得到`Nothing`; 
> - 再看`f a <*> f [a]`会发生什么, 也就是看`f`的apply函数的实现. 如`Maybe`是有`Nothing`就返回`Nothing`, List是进行完全组合.

如果`sequenceA`的参数是函数列表`[a -> b]`的话, 就会生成`a -> [b]`, 即接收一个参数, 将其应用到List中的每个函数上, 形成结果List.
```haskell
ghci> sequenceA [(+3),(+2),(+1)] 3 
[6,5,4]
```

`Maybe`中只要有`Nothing`就只会得到`Nothing`, 因为`xxx <*> Nothing`永远返回`Nothing`:
```haskell
ghci> sequenceA [Just 3, Just 2, Just 1]  
Just [3,2,1]  
ghci> sequenceA [Just 3, Nothing, Just 1]  
Nothing  
```

List之间的lifting后的 `:` 也依然会出现完全组合:
```haskell
ghci> sequenceA [[1,2,3],[4,5,6]]  
[[1,4],[1,5],[1,6],[2,4],[2,5],[2,6],[3,4],[3,5],[3,6]]  
ghci> sequenceA [[1,2,3],[4,5,6],[3,4,4],[]]  
[]
ghci> sequenceA [[1,2,3],[4,5,6], [7,8,9]]
[[1,4,7],[1,4,8],[1,4,9],[1,5,7],[1,5,8],[1,5,9],[1,6,7],[1,6,8],[1,6,9],[2,4,7],[2,4,8],[2,4,9],[2,5,7],[2,5,8],[2,5,9],[2,6,7],[2,6,8],[2,6,9],[3,4,7],[3,4,8],[3,4,9],[3,5,7],[3,5,8],[3,5,9],[3,6,7],[3,6,8],[3,6,9]]
```

可以用于多个`Bool`判断:
```haskell
-- 等价于: sequenceA [(>4),(<10),odd] 7
ghci> and $ sequenceA [(>4),(<10),odd] 7 
True
```

## Alternative

Alternative 是 **Applicative的Monoid**. 相比于Monoid, 实现Alternative的type必须包含一个type variable, 且由Applicative保证其可操作性.

需要提供`empty`和`<|>`的实现.

```haskell
type Alternative :: (* -> *) -> Constraint
class Applicative f => Alternative f where
  empty :: f a
  (<|>) :: f a -> f a -> f a
  
  -- One or more.
  some :: f a -> f [a]
  some v = (:) <$> v <*> many v

  -- Zero or more.
  many :: f a -> f [a]
  many v = some v <|> pure []
```

`empty`是`<|>`的 **identity**.

`<|>`是一个有**结合律**的且以`empty`为identity的**二元函数**.

> [!info]
> [MonadPlus](Haskell/Haskell.md#MonadPlus)是Alternative和Monad的简单拼接, 能扩展出很多功能.

`some`会在`v`为 一个<u>不执行apply目标函数</u>的值 的时候中断, 例如Nothing. `some`最少返回空.

`many`调用`some`, 就算`some`返回空, 也起码有`pure []`, 因此最少返回单一结果.

```haskell
ghci> some Nothing
Nothing
ghci> many Nothing
Just []
ghci> some []
[]
ghci> many []
[[]]
```

> [!warning]
> 依然没搞懂some和many的用法, 说是可以用于parser.

### Alternative Law

```haskell
-- Identity
empty <|> a     == a
a     <|> empty == a
-- Associativity
u <|> (v <|> w) = (u <|> v) <|> w
```

### 例子

List 是 Alternative. 它会把所有List**拼接**起来. 因此空List不会产生任何影响.
```haskell
instance Alternative [] where
    empty = []
    (<|>) = (++)
```

```haskell
ghci> [1,2] <|> [3,4]
[1,2,3,4]
ghci> [1,2] <|> empty
[1,2]
```

---

ZipList 是 Alternative. 相比于List, ZipList会把后面的元素用前面的同下标的元素给**覆盖**掉.
```haskell
instance Alternative ZipList where
   empty = ZipList []
   ZipList xs <|> ZipList ys = ZipList (xs ++ drop (length xs) ys)
```

```haskell
ghci> getZipList $ ZipList [1,2] <|> ZipList [3,4,5,6]
[1,2,5,6]
ghci> getZipList $ ZipList [1,2,3,4] <|> ZipList [3,4,5,6]
[1,2,3,4]
```

---

Maybe 是 Alternative. 它永远选择**第一个非Nothing的东西**(或在全为Nothing的时候返回Nothing). 因此`Nothing`不会产生任何影响.
```haskell
instance Alternative Maybe where
    empty = Nothing
    Nothing <|> r = r
    l       <|> _ = l
```

```haskell
ghci> Just 2 <|> Just 3
Just 2
ghci> Nothing <|> Just 2
Just 2
```

### asum

`asum`函数用于折叠Alternative. 过程类似于foldMap利用Monoid进行折叠, 但不需要转化.
```haskell
asum :: (Foldable t, Alternative f) => t (f a) -> f a
asum = foldr (<|>) empty
```

```haskell
ghci> asum [Nothing, Just 5, Just 3]
Just 5
ghci> asum [[2],[3],[4,5]]
[2,3,4,5]
```

## Monad

[Control.Monad](https://downloads.haskell.org/ghc/latest/docs/libraries/base-4.20.0.0-1f57/Control-Monad.html#t:Monad)

Monad typeclass是为了实现: 有一个包装后的值(**monadic value**), 和一个接收普通值返回被包装后的值的函数(**monadic function**, `Monad m => a -> m b`), 把此函数应用到包装内的值, 并且只返回一层包装(也是 monadic value). 强制使用`fmap`会得到`f f a`两层包装. **只有Applicative才能实现Monad**. 

只需要实现`>>=`.

```haskell
type Monad :: (* -> *) -> Constraint
class Applicative m => Monad m where
  (>>=) :: m a -> (a -> m b) -> m b
  
  (>>) :: m a -> m b -> m b
  m >> k = m >>= \_ -> k -- m >>= const k
  
  return :: a -> m a
  return = pure
```

`>>=`称为**bind**函数, 或者`flatMap`. 是**左结合**的. 它提供了monadic function(`a -> m b`)的**连续应用**的可能性. 

> [!info]
> lambda表达式的优先级极低, 所以下面的式子被解释为:
> - `f >>= \x -> b >>= g`
> - `f >>= (\x -> (b >>= g))`.
>
> 因此带闭包且不加括号的`>>=`式子实际上是**嵌套调用**, 每一层嵌套相当于使用lambda的参数来"定义"一个"变量"(`>>=`左边的Monad里面的值); 这"变量"<u>可以被内部的lambda捕获到</u>; 先执行嵌套外层的代码, 再往内层走. 所以do-notation的基本结构就是嵌套的`>>=`式, 使其可以"绑定"变量, 并且像命令式语言一样从前往后执行.
>
> 注意与普通的`f >>= g >>= h`区分. 该普通式没有出现广域的变量, 仅仅是对一个Monad<u>从左往右</u>**连续应用monadic函数**.
> 
> 如果内层操作<u>依赖外层变量</u>的话, 就必须使用嵌套形式, 而此时一般就会写成do-notation.
> 
> 设有式子`f >>= \x -> b >>= P`, 其中`P`是一个表达式; 如果`P`与`x`**无关**, 那么它就符合<u>结合律</u>的形式, 可以安全地写成`f >>= (\x -> b) >>= P`. 如果`b`实际上是函数调用`k x`, 而`P`可以看做函数`h`(参数无`x`)的话, 那么就写成`f >>= k >>= h`.

`>>`利用bind拼接Monad, 会让bind的目标函数永远返回第二个Monad, 注意要考虑bind的时候目标函数是否会被执行, 也因此可以用于判断Monad内容是不是"正常"(即查看是否失败). 类似于命令式语言的分号`;`, 只会执行不会取值, 这在do-notation里面有更多体现. 使用`MonadFail`可以更好地定义失败的情况.

> [!note]
> 普通变量的函数只需要给返回值加上return包装就能用于Monad. 可以使用复合来包装: `return . f`.

> [!tip]
> Monad 可以通过 monadic function **凭空生成**.

Monad和Applicative有如下关系:
- `return = pure`, 在这里是把普通变量用Monad包装.
- `m1 <*> m2 = m1 >>= (\x1 -> m2 >>= (\x2 -> return (x1 x2)))`.

### Monad Law

Left Identity (Left Unit Law): 把变量装入Monad再应用`a -> m b`**等价于直接应用**. 即`return`是其**特殊的左幺元**. 因此Monad计算的开头可以不使用`return`生成初始Monad, 而是直接应用monadic function. 
```haskell
return a >>= k = k a
```

Right Identity (Right Unit Law): Monad bind到return上得到它本身. 即Monad是**自相似**的几何结构. 即`return`是其**特殊的右幺元**
```haskell
m >>= return = m
```

Associativity: `>>=`计算有**特殊的结合律**, 需要写成lambda, 但可以使用`<=<`让其更简洁. 这保证了bind不会出现额外作用.
```haskell
(m >>= k) >>= h = m >>= (\x -> k x >>= h)
```

> [!tip]
> 类似于Monoid的三个Law, 这也是为什么说Monad是**自函子(endofunctor)范畴上的一个幺半群(monoid)**. 不过这里的幺半群指的是范畴论里的而不是群论里的.

> [!info]
> 可以推得:
> ```haskell
> fmap f xs = xs >>= return . f
> ```

通过Associativity, 可以认为两个 monadic function 可以特殊地**复合**, 保证连续bind这两个函数(等号右边)<u>等价于</u>bind复合后的函数(等号左边). 使用`<=<`复合两个 monadic function (在`>>=`计算意义上):
```haskell
(<=<) :: (Monad m) => (b -> m c) -> (a -> m b) -> (a -> m c)
f <=< g = (\x -> g x >>= f)
```

由于Monad可以看做是monadic function的结果, 因此可以把 Monad Law 写成:
```haskell
-- Left Identity
return <=< f = f
-- Right Identity
f <=< return = f
-- Associativity
(f <=< g) <=< h == f <=< (g <=< h)
```

> [!note]
> 可见 monadic function `Monad m => a -> m b` 是在`<=<`运算上的Monoid.

这种一连串的`<=<`复合就像是`.`复合的Monad版本, 它会得到一个单参函数, 输入一个普通变量, 然后进行一连串的Monad计算后输出结果Monad.
```haskell
ghci> f x = Just (x + 3)
ghci> g x = Just (x * 2)
ghci> h x = Just (x + 5)
-- f(g(h(7)))
ghci> ((f <=< g) <=< h) 7
Just 27
-- 结合律, 因此都不需要加括号
ghci> (f <=< (g <=< h)) 7
Just 27
```


### do-notation

> [!info]
> `do`是一个强大的语法糖.

Monad想要bind的 monadic function 的内部很可能也是使用bind来实现的, 或者说组合多个Monad就会出现这种情况, 此时会出现<u>临时的lambda</u>. 例如把两个Maybe给bind到一个二元函数(`show x ++ y`)中:
```haskell
foo :: Maybe String  
-- Just 3 >>= (\x -> Just "!" >>= (\y -> Just (show x ++ y)))
foo = Just 3 >>= \x -> Just "!" >>= \y -> Just (show x ++ y)
```

使用do可以更简洁地等价表示成:
```haskell
foo :: Maybe String  
foo = do  
    x <- Just 3  
    y <- Just "!"  
    Just (show x ++ y)  
```

do可以通过递归的形式解糖, 每次把第一行扔出去, 直到简单返回最后一行. 在最后的解糖结果表达式中, 第一行是最外面一层, 最后一行是最里面的入参.

- `<-`把Monad里的东西绑定, 绑定出的是**lambda表达式的参数**, 可以理解为Monad里面的被包含type的一个单位. 因此`Monad a`绑定后的type就是`a`, 无论此Monad内部如何存放`a`. 
	- 即`do { x <- m1; m2 x }` 等价于 `m1 >>= (\x -> do { m2 x } )`.
- 在`do`里面出现了不绑定的行(即普通表达式)时, 相当于使用`>>`, 只是**执行**, 其结果不被考虑. 
	- 即`do { m1; m2 }` 等价于 `m1 >> do { m2 }`.
- `do`里面可以有**let表达式**, 相当于在外部书写, 只是为了写代码更流畅所以允许写里面.
	- 即`do { let s1; m1 s1 }` 等价于 `let s1 in do { m1 s1 }`.
	- 或`do { let s1; m1 s1 }` 等价于 `do { m1 s1 } where s1`.
- `do`要求最后一行决定返回类型, 因此**最后一行不进行绑定**. 
	- 即`do { m1 }` 等价于 `m1`.

### Maybe Monad

`Maybe`的Monad实现就是把内部值用模式匹配取出来传给函数即可. 这使得一些<u>可能失败的函数</u>(即可能返回`Nothing`的函数)对一个变量进行**连续**的`>>=`的时候, 如果有**任一**时刻出现失败, 那么结果必然是失败.
```haskell
instance Monad Maybe where  
	return x = Just x
    Nothing >>= f = Nothing  
    Just x >>= f  = f x  
```

```haskell
ghci> return "WHAT" :: Maybe String  
Just "WHAT"  
ghci> Just 9 >>= \x -> return (x*10)  
Just 90  
ghci> Nothing >>= \x -> return (x*10)  
Nothing 
```

`>>`表现为, 连续的`Maybe`中只要出现Nothing就返回Nothing, 否则返回最后一项.
```haskell
ghci> Nothing >> Just 3  
Nothing  
ghci> Just 3 >> Just 4  
Just 4  
ghci> Just 3 >> Nothing  
Nothing  
```

### List Monad

```haskell
instance Monad [] where  
    return x = [x]  
    xs >>= f = concat (map f xs)  
```

```haskell
ghci> [3,4,5] >>= \x -> [x,-x]  
[3,-3,4,-4,5,-5]  
ghci> [] >>= \x -> ["bad","mad","rad"] 
[]
```

两个List Monad放到一个二元函数内的时候(使用do-notation), 更可以体现其non-deterministic:
```haskell
ghci> [1,2] >>= \n -> ['a','b'] >>= \ch -> return (n,ch)  
[(1,'a'),(1,'b'),(2,'a'),(2,'b')] 
-- 等价于
listOfTuples :: [(Int,Char)]  
listOfTuples = do  
    n <- [1,2]  
    ch <- ['a','b']  
    return (n,ch) 
-- 等价于
[ (n,ch) | n <- [1,2], ch <- ['a','b'] ]
```

可见, **list comprehension是个语法糖**, 其中的`<-`就是do-notation的绑定操作, `|`左边就是将被return包装的返回值, 因此整个语句最后会变成`>>=`语句.

## MonadFail

表示可能失败的Monad. 

```haskell
type MonadFail :: (* -> *) -> Constraint
class Monad m => MonadFail m where
  fail :: String -> m a
```

其**law**是保证`fail s`是`>>=`的 **left zero**. 这保证<u>失败后会取消后面的所有操作</u>.
```haskell
fail s >>= f = fail s
```

如果这还是个MonadPlus, 那么应当把`fail`实现成永远返回`mzero`, 毕竟这恰好就是 left zero.
```haskell
fail _ = mzero
```

## MonadPlus

MonadPlus是 **Alternative 和 Monad 的简单拼接**, 即其内新增的东西都等价于Alternative里的东西.

不需要提供任何实现, 因此在Monad是Alternative的时候可以直接声明它是MonadPlus.

```haskell
type MonadPlus :: (* -> *) -> Constraint
class (Alternative m, Monad m) => MonadPlus m where
  mzero :: m a
  mzero = empty
  
  mplus :: m a -> m a -> m a
  mplus = (<|>)
```

`mzero`可以代表**失败**情况, 因此功能上强于MonadFail.

`mplus`被解释为Choice, 是个有identity的可结合的二元函数.

# Lazy

下面这段cpp代码会死循环(最后出现segment fault), 因为`factBad(x)`调用`myIfBad`的时候就不得不先把`factBad(x-1)`算出来, 出现了死递归(x会一直减少但不可能进入实际比较).
```cpp
auto myIfBad(bool flag, int a, int b) -> int {
    if (flag) {
        return a;
    }
    return b;
}

auto factBad(int x) -> int {
    return myIfBad(x == 0, 1, x * factBad(x - 1));
}

int main() {
    std::cout << factBad(10) << std::endl;
}
```

但使用无参闭包进行lazy化后就能执行, 它满足"不使用就无影响":
```haskell
auto myIf(bool flag, std::function<int()> a, std::function<int()> b) -> int {
    if (flag) {
        return a();
    }
    return b();
}

auto fact(int x) -> int {
    return myIf(x == 0, []() { return 1; }, [x]() { return x * fact(x - 1); });
}
```

java, cpp等语言都是在参数上eager, 但是条件表达式不eager.

一个为了延迟计算(delay evaluation)的**无参函数**称为**thunk**. 用作动词: "thunk the expression", 表示将表达式包装延迟计算. 并不是朴素的无参函数, 它可能会有记忆化等特性.





# 库, 工具与技巧

## 构造元组

`(,,)` 等价于 `\x y z -> (x,y,z)`, 即把各的参数分别写成元组的每一项. `,`可以写任意数量个.

```haskell
ghci> (,,,) 1 2 3 4
(1,2,3,4)
```




## Conditional execution of monadic expressions

[Control.Monad: Conditional execution of monadic expressions](https://downloads.haskell.org/ghc/latest/docs/libraries/base-4.20.0.0-1f57/Control-Monad.html#g:5)
### guard

`guard`函数会进行**条件判断**:
- 为真时返回`pure ()`(最简地标记"成功")
- 否则返回`empty`(表示"失败").
因此也要求type实现了**Alternative**以指明**identity**作为失败标志.
```haskell
guard :: (Alternative f) => Bool -> f () 
guard True = pure () 
guard False = empty
```

```haskell
ghci> guard (5 > 2) :: Maybe ()
Just ()
ghci> guard (1 > 2) :: Maybe ()
Nothing
ghci> guard (5 > 2) :: [()]
[()]
ghci> guard (1 > 2) :: [()]
[]
```

MonadPlus往往会在Empty时采取另一种行为, 而`>>`就是用来检测Monad内容是否"正常"的. 因此`guard`配合`>>`可以用于通过条件判断来决定是否执行`>>`右侧的东西. 

在`do`语句里面, `guard`体现为: 如果判断失败, 则**直接返回失败值**, 不再继续执行.

```haskell
ghci> [1..50] >>= (\x -> guard ('7' `elem` show x) >> return x) 
[7,17,27,37,47]
ghci> do { x <- [1..50]; guard ('7' `elem` show x); return x }
[7,17,27,37,47]
ghci> [ x | x <- [1..50], '7' `elem` show x ]
[7,17,27,37,47]
```

```haskell
safeDiv :: Int -> Int -> Maybe Int
safeDiv x y | y /= 0    = Just (x `div` y)
            | otherwise = Nothing
-- 等价于
safeDiv :: Int -> Int -> Maybe Int
safeDiv x y = do
  guard (y /= 0) -- 判断失败则直接返回Nothing
  return (x `div` y)
```

### when

进行条件判断, 如果判定失败, 则**Applicative**被<u>灭杀</u>成`pure`, 失去其之前构建的其他信息.

```haskell
when :: Applicative f => Bool -> f () -> f ()
when p s = if p then s else pure ()
```

例如`putStrLn String`会得到`IO ()`, 直接丢给main的话就能输出字符串内容了, 但是如果`when`判定失败就相当于返回了`return ()`, 什么都不会输出.
```haskell
main :: IO ()
main = do
    let x = 10
    when (x > 5) $ putStrLn "x is greater than 5"
    when (x < 5) $ putStrLn "x is less than 5"
```

```haskell
main :: IO ()
main = do
    c <- getChar
    when (c /= ' ') $ do
        putChar c
        main
```

### unless

仅仅是把`when`的对错反过来.


## sequence

sequence把 多个Monad组合 合成 一个包含组合的Monad, 例如把`[m a]`变成`m [a]`.

```haskell
sequence :: (Traversable t, Monad m) => t (m a) -> m (t a)
```

```haskell
sequence :: Monad m => [m a] -> m [a]
sequence :: [IO a] -> IO [a]
```


## Foldable

[Data.Foldable](https://hackage.haskell.org/package/base-4.20.0.1/docs/Data-Foldable.html)

- foldl 可以把列表`[a]`从左到右聚合到初值`b`上, 且调用函数时**累计值在左**, 元素在右.
- foldr 可以把列表`[a]`从右到左(先取尾部)聚合到初值`b`上, 且调用函数时**累计值在右**, 元素在左.
```haskell
-- foldl f z [x1, x2, ..., xn] == 
-- (...((z `f` x1) `f` x2) `f`...) `f` xn
foldl :: Foldable t => (b -> a -> b) -> b -> t a -> b
-- foldr f z [x1, x2, ..., xn] == 
-- x1 `f` (x2 `f` ... (xn `f` z)...)
foldr :: Foldable t => (a -> b -> b) -> b -> t a -> b
```
- `foldl1`和`foldr1`分别将首或尾元素作为初值.
- `foldl'` 和 `foldr'` 需要使用 `import Data.Foldable` 导入, 它们是对应fold函数的变型, 不使用lazy而是严格按照运算符顺序计算.

> [!info]
> 定义 `foldMap` **或** `foldr` 即可完成`Foldable`实现.

更直观的理解:
- `foldr (-) 1 [2,3,4]`: 把初值`1`放到**最后**, 写出`2 -' 3 -' 4 -' 1`, 然后函数**右结合**成`2 - (3 - (4 - 1))`. 得到答案`2`.
- `foldl (-) 1 [2,3,4]`: 把初值`1`放到**开头**, 写出`1 -' 2 -' 3 -' 4`, 然后函数**左结合**成`((1 - 2) - 3) - 4`. 得到答案`-8`.
```haskell
-- 使用`++`理解
-- 因为有结合律所以比较规整
ghci> foldr (++) "1" ["2","3","4"]
"2341" -- 1在最后
ghci> foldl (++) "1" ["2","3","4"]
"1234" -- 1在开头
```

```haskell
-- 二者功能相同但函数参数反了
ghci> foldl (\acc c -> acc ++ [c]) "foo" ['a', 'b', 'c', 'd']
"fooabcd"
ghci> foldr (\c acc -> acc ++ [c]) "foo" ['a', 'b', 'c', 'd']
"foodcba"
```

> [!warning]
> When it comes to **lists**, you always want to use either `foldl'` or `foldr` instead of `foldl`.

在List上的两个`fold`函数实现:
```haskell
foldr :: (a -> b -> b) -> b -> [a] -> b
foldr f z (x:xs) = f x (foldr f z xs)
foldr f z [] = z

foldl :: (b -> a -> b) -> b -> [a] -> b
foldl f z (x:xs) = foldl f (f z x) xs
foldl f z [] = z
```

令`fi = f xi`, 有:

- `foldr f z xs` 第一步先计算`f x1`. 其等价于: `f1 . f2 . ... . fn $ z`.
- `foldr' f z xs` 第一步先计算`f xn z`. 其等价于: `z & fn & ... & f2 & f1`.
```text
   f
  / \
x1   f
    / \
  x2   ...
         \
          f
         / \
       xn   z
```

- `foldl f z xs` 第一步先计算`f xn`. 其等价于: `fn . ... . f2 . f1 $ z`.
- `foldl' f z xs` 第一步先计算`f x1 z`. 其等价于: `z & f1 & f2 & ... & fn`.
```text
       f
      / \
    ...  xn
   /
  f
 / \
z   x1
```

- 可见`foldl`和`foldr'`都会先找到最末端开始算, 因此禁止在无限数据结构中使用, 且在没有逆迭代器的数据结构下需要$O(n)$的thunk.
- `foldl'`的计算方式和其他普通程序语言的`fold`一样.

对无限List, lazy且从左边开始的`foldr`在有限步骤内能确定结果(要求函数是short-circuiting的, 且对其第二个参数lazy)的情况下是可以穷尽计算的:
```haskell
ghci> foldr (&&) True (repeat False)
False
```

要注意**strict**的非对称性, 如`++`的话不可交换, 只对其第一个参数strict, 那么`foldr`的时候传把元素放第一个参数性能会好得多. 换成`+`就没这问题. 见[Expectation of efficient left-to-right iteration](https://hackage.haskell.org/package/base-4.20.0.1/docs/Data-Foldable.html#g:8).
```haskell
-- 第一行更快
myconcat xs = foldr (\a b -> a ++ b) [] xs
revconcat xs = foldr (\a b -> b ++ a) [] xs
-- 第二行不如正向拼接后翻转
revconcat = foldr (++) [] . reverse
```

### foldMap

Foldable可以通过仅提供`foldMap`来完成实现, 而`foldMap`**使用Monoid来fold数据**. 它要求提供一个函数把容器内的数据转化成Monoid, 然后使用对应的`mappend`将它们fold到`mempty`上.

```haskell
-- a -> m 是源type到Monoid实现type的转化函数, 如`Sum`, `Product`等
-- t a 是包含了源数据a的type变量, 如`[a]`
-- 得到 a type 数据全部变成 m Monoid type 后的fold结果, 其type为m
foldMap :: Monoid m => (a -> m) -> t a -> m
```

例如, 使用`foldMap Sum`获取求和函数, 使用`foldMap (replicate 3)`获取重复3次的函数:
```haskell
ghci> foldMap Sum [1, 3, 5]
Sum {getSum = 9}
ghci> foldMap (replicate 3) [1, 2, 3]
[1,1,1,2,2,2,3,3,3]
```

例如, 对树的fold应当是遍历整棵树, 对每个节点使用给定的函数进行计算聚合. 
```haskell
data Tree a = Empty | Node a (Tree a) (Tree a) deriving (Show, Read, Eq)      

instance Foldable Tree where  
	-- 空树
    foldMap f Empty = mempty  
    -- 前序遍历
    foldMap f (Node x l r) = foldMap f l `mappend`  
                                f x           `mappend`  
                                foldMap f r  
```


# Module 模块

[haskell org Modules](https://www.haskell.org/onlinereport/haskell2010/haskellch5.html)

## 导入模块

库使用**大驼峰**命名, 使用`.`来表示路径.

许多东西都位于`Prelude`模块, 自动引入.

```haskell
import module1 module2 module3
```

使用`(xxx)`来引入部分：
```haskell
import module1 (func1, func2)
```

使用`hiding`来exclude部分：
```haskell
import module1 hiding (func)
```

> [!info]
> `Prelude`模块虽然已经自动引入，但仍可以手动只引入其中部分符号或者屏蔽其中部分符号：`import Prelude hiding (max)`.

使用`qualified`, 使得使用时必须使用名称`modulename.xxx`, 防止名字冲突:
```haskell
import qualified modulename
```

使用`as`起别名:
```haskell
import qualified Data.Map as M hiding (map)
```


## 编写模块

开头定义模块名, 使用`(xxx, xxx)`来进行**export**(没写在这里的都是**私有**项), 后跟`where`. 必须写在开头, import写在后面. 可以把import进来的东西进行export.

```haskell
-- 文件名: MyModule.hs
module MyModule (
	greet, 
	add, 
	Shape(...) -- 导出所有 value constructor
) where

greet :: String -> String
greet name = "Hello, " ++ name ++ "!"

add :: Int -> Int -> Int
add x y = x + y

data Shape = Circle Point Float | Rectangle Point Point deriving (Show)
```


# 调试

## trace

`Debug.Trace`模块提供`trace`函数, 使其可以在不破坏程序逻辑的情况下输出指定文本. 这是不纯的操作, 因此仅在调试时使用.

`trace`接收一个字符串和一个普通值, 会输出字符串, 然后返回普通值本身.
```haskell
trace :: String -> a -> a
```

例如, 要给`subStringToIndex s n = take n s`添加调试信息的话, 就先使用`trace`输出字符串, 然后把`take n s`直接写在后面作为参数, 这样`take n s`依然会是结果.
```haskell
subStringToIndex :: String -> Int -> String
subStringToIndex s n = trace ("para s is " ++ show s)  take n s 
```

在**Monad操作**中(尤其是do-notation里面), 可以使用`traceM`来在输出文本时返回空Monad, 这个Monad往往会通过`>>`丢掉.
```haskell
traceM :: Monad m => String -> m ()
```

例如, 要打印包装在`State` Monad里面的字符串, 就需要使用`traceM`. 它会返回`State ()`并且丢弃, 满足检查.
```haskell
monadicSubStringToIndex ::  Int -> State String String
monadicSubStringToIndex n = do
    s1 <- get
    traceM $ "original string is " ++ show s1
    return $ take n s1

main :: IO ()
main = putStrLn $ show $ runState (monadicSubStringToIndex 2) "abcde"
```

















