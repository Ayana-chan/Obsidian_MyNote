[Haskell趣学指南\_w3cschool](https://www.w3cschool.cn/hsriti/)
# 安装与使用

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

# ghci

直接运行haskell的工具.

- `:l :load` 加载(hs文件)
- `:r :reload` 重载
- `:t :type` 获取类型
- `:i :info` 信息，针对函数、类型、类型类(能看到有哪些instance)等。
- `:k :kind` 得知一个类型的Kind。

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

# 语法
## 基础

Haskell是强类型和静态类型的。

Haskell是惰性的，如非特殊说明，函数真正需要结果以前不会被求值，加上引用透明，可以把程序看做数据的一系列变形。也就是说惰性语言中的计算只是一组初始数据和变换公式。

在函数式编程语言中，变量一旦指定就不可以更改了，在命令式编程语言中，变量表示状态，如果状态不可变，那么能做的事情将非常有限。而函数式编程语言中，变量的含义更接近数学中的变量，`x=5`表示`x`就是`5`，而不是`x`处于`5`这个状态。

函数式编程语言的一般思路：先取一个初始集合，对其进行变形、执行过滤条件（map and reduce）得到最终结果。

所有的表达式都要求返回一个值. `if`语句也是个表达式, 因此必须有`else`.

**type variable (类型变量)** 就是泛型, 由小写字母`a` `b` `c`等表示. 使用类型变量的函数称之为**多态函数**.


## 注释

```haskell
-- 单行注释
{-
多行注释
-}
```


## 运算符


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

```haskell
data NameOfType = ValueConstructor1 TypesOfParams1 | ... deriving (Typeclass1, Typeclass2, ...)
```

`data`表达式的`=`左侧是一个 **Type Constructor (类型构造子)**, 它是可以有type参数的. **无参数**的 type constructor 称作 **nullary** type constructor, **简称 type**.

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

type constructor 可以看做<u>类型意义</u>上的<u>函数</u>, 但不是haskell的函数 (在lisp里面它直接就是普通函数).

### Value Constructor

`data`表达式的`=`右侧的 **Value Constructor (或Data Constructor, 值构造子)** 使用`|`(或)来分割. 它们明确类型的所有可能取值. **无参数**的value constructor (例如`True`, `[]`, `5`) 称作**nullary** value constructor. 

value constructor是<u>跟着type一起声明</u>的, 可以理解为赋予了这种值结构以一个名字. 如果值构造子只有一个, 那么完全可以让type名和value constructor名相同. 

value constructor 就是个**函数** (不过其名字与参数可以被模式匹配)(而不是类型!). 名字要写成**大驼峰**.

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

### 派生 typeclass

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


### type 类型别名

使用`type`给类型起别名.

```haskell
type String = [Char]
```

可以保留type variable:
```haskell
type AssocList k v = [(k,v)]
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
-- 
elem :: (Foldable t, Eq a) => a -> t a -> Bool
```

type class 可以被 type 给 deriving.

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




## 函数

**小驼峰**命名.

等号左边写参数, 右边写函数体.

不需要先声明就可以使用.

```haskell
functionName arg1 arg2 = arg1 + arg2
```

> [!info]
> `'`也是函数名的合法字符，常常使用单引号来区分一个稍经修改但差别不大的函数。

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

fold系列操作把列表变成值.
- foldl 可以把列表`[a]`从左到右聚合到初值`b`上, 且调用函数时初值在前元素在后.
- foldl 可以把列表`[a]`从右到左(先取尾部)聚合到初值`b`上, 且调用函数时元素在前初值在后.
```haskell
foldl :: Foldable t => (b -> a -> b) -> b -> t a -> b
foldr :: Foldable t => (a -> b -> b) -> b -> t a -> b
```
- `foldl1`和`foldr1`分别将首或尾元素作为初值.

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

```haskell
-- 定义
(&) :: a -> (a -> b) -> b
x & f = f x
```


### `.` Function Composition 函数组合

即**复合函数**$(f \circ g)(x) = f(g(x))$, 要求把`g(x)`的结果作为`f`的自变量进一步求值.

使用`.`来定义复合函数. 使用`f . g`定义$f(g(x))$, 先调用后者.
```haskell
-- 定义
(.) :: (b -> c) -> (a -> b) -> a -> c
f . g = \x -> f (g x)
```

**优先级仅低于函数调用**, 即`f . g a`会先进行`g a`. 
- 这就导致如果`g`是`a -> b`的话, 会导致此句变成`f . b`, 是错误的. 因此, 需要`(f . g) a`, 或者`f . g $ a`.
- 但有时是故意要这么做的, 为了方便对函数进行Curry, 因为这只支持单参函数, 所以必须给每个被串联的函数Curry成单参. 
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

Haskell中，List是最常用的数据结构，十分强大，可以解决许多问题。List是单类型的数据结构，不能将不同类型数据放到同一个List中。

ghci中可以使用`let a = 1`来定义常量，与脚本中`a = 1`相同。

- 对于字符串来说，`"hello"`仅仅只是`['h','e','l','l','o']`的语法糖，也就是说字符串就是List。
- **合并两个List**，`l1 ++ l2`。实现中会遍历左边的List，对于长字符串或者列表不是很友好。
- 使用`:`可以**把元素和列表合并**，`elem1 : list`。如果要使用`++`连接单个元素到List可以用`[elem1] ++ list`。
- 实际上`[1, 2, 3]`就是`1:2:3:[]`的语法糖，也就是空列表依次在前面插入元素。
- 按照**索引**取List元素：`[1, 2, 3] !! 2`，索引从0开始。越界将报错。
- List内部元素可比较时，可以使用`> >= ==`等运算符比较大小，将会按元素依次比较。

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


## Range

- `[1..20]`即表示`[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]`。
- 给出指定第二个元素将使用第一个和第二个元素的间隔作为步长，默认是1`[1,3..20]`表示`[1,3,5,7,9,11,13,15,17,19]`。
- 步长可以为负，到达不了上限则生成列表为空：`[1,0..10]`为`[]`。

- 不指定上限则**无限长度**，步长为0同样也是无限长度。无限长度列表将会在使用到时才求值。
- `cycle list`可以生成循环list元素的无限列表。
- `repeat value`生成一个仅包含该值的无限列表。
- `replicate n value`重复一个值n次。


## pattern matching 模式匹配

赋值就会发生模式匹配, 分解数据到其他变量中.

### let

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

### where

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

# Module 模块


标准库: [Haskell Hierarchical Libraries](https://downloads.haskell.org/ghc/latest/docs/libraries/)

检索库内容(包括第三方库): [Hoogle](https://hoogle.haskell.org/)

[5 Modules](https://www.haskell.org/onlinereport/haskell2010/haskellch5.html)

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














