
# 安装与使用

使用科大源安装GHCup: [GHCup - USTC Mirror Help](https://mirrors.ustc.edu.cn/help/ghcup.html)

[配置 Haskell 开发环境（2023） | Zhichao Guan’s Home Page](https://higher-order.fun/cn/2023/08/27/InstallHaskell.html)

换源下载完GHCup后, 依然会卡在其他安装, 此时可以安全把安装窗口关掉, 换源后手动`ghcup install xxx` 和 `cabal update`等.

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


# 语法
## 基础

Haskell是强类型和静态类型的。

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

函数type用`->`串起来, 最后一个是返回值, 前几个全是输入. 比如输入两个浮点数输出整数的函数的类型为`Float -> Float -> Int`.

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

在表示一个东西的type的时候, 有时候需要使用type variable, 但是又需要对其做出限定(即泛型约束), 那么就需要在`=>`左边写明每个type variable要属于哪些typeclass.
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

没有参数的函数称之为定义或者名字，定义后不可修改:

```haskell
hello = "hello,world!"
```


## List 列表

Haskell中，List是最常用的数据结构，十分强大，可以解决许多问题。List是单类型的数据结构，不能将不同类型数据放到同一个List中。

ghci中可以使用`let a = 1`来定义常量，与脚本中`a = 1`相同。

- 对于字符串来说，`"hello"`仅仅只是`['h','e','l','l','o']`的语法糖，也就是说字符串就是List。
- 合并两个List，`l1 ++ l2`。实现中会遍历左边的List，对于长字符串或者列表不是很友好。
- 使用`:`可以往列表前插入元素，`elem1 : list`。如果要使用`++`连接单个元素到List可以用`[elem1] ++ list`。
- 实际上`[1, 2, 3]`就是`1:2:3:[]`的语法糖，也就是空列表依次在前面插入元素。
- 按照索引取List元素：`[1, 2, 3] !! 2`，索引从0开始。越界将报错。
- List同样可以存放List，不过List的元素类型是它的类型的一部分，需要类型匹配：`[1]:[[2]]`。
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


## 函数相关语法

### pattern matching 模式匹配











