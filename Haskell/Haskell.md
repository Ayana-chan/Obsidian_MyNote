
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

使用`:t`命令可以检测出其后跟的表达式的类型。
- `:l :load` 加载(hs文件)
- `:r :reload` 重载
- `:t :type` 获取类型
- `:i :info` 信息，针对函数、类型、类型类等。
- `:k :kind` 得知一个类型的Kind。


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

每个表达式都有type, "a的类型为A"表示为`a::A`.

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

### type variable 类型变量

就是泛型, 由小写字母`a` `b` `c`等表示. 使用类型变量的函数称之为**多态函数**.


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


### `$` 函数调用符

目的是让`$`右侧先进行求值再交给左侧. `f1 $ f2 x`等价于`f1 (f2 x)`. 因此其作用是减少括号的使用.

```haskell
-- 是个中缀函数
($) :: (a -> b) -> a -> b
f $ x = f x
```

具有最低优先级, 因此在表达式中加一个`$`就能将其一刀两断.

`$`是**右结合**的, 达成分块执行的效果 (从右往左运行`$`隔出来的块).

- 左侧不会是单纯变量, 否则就语法错误了(没有东西消耗这堆变量).
- 如果右侧是单纯变量的话就没什么意义了, 相当于给参数列表加括号, 并不会发生什么改变.
- 因此, 使用的时候, 左右两遍都得是函数或者表达式, 让右侧计算(Curry)完毕后再给左侧.

`f (1 + 1)` 等价于 `f $ 1 + 1`.

`$ x`可以**将变量变成函数**: 接受一个函数, 返回将`x`应用于传入的函数后得到的结果.
```haskell
>>> :t ($ 1)
($ 1) :: Num a => (a -> b) -> b
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











