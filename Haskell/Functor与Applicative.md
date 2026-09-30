[← Haskell](Haskell.md)

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

对Functor内函数(`a -> b`)传`xxx` (`xxx :: a`)作为**参数**时, 可以采用`fmap (\f -> f xxx)` 来完成, 如:
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

对于一个内部为函数的Functor `F (a -> b)`, 函数的传参可以使用`fmap (\f -> f xxx)`来完成; 但是, 如果没有`xxx: a`, 而只有另一个Functor `yyy :: F a`呢? 换句话说, 如何对`Functor (a -> b)`以`Functor a`进行map over? 最暴力的解法是手动模式匹配, 但显然不美观.

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

`liftA2`提供了更方便的`f a -> f b -> f c`函数的获取方式. 换句话说, Applicative**都能使用**形如`f a -> f b -> f c`的东西. 详见[liftA2](#liftA2).


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

List的apply会生成所有可能的组合, 这可以使用[Non-deterministic的视角](%E5%87%BD%E6%95%B0%E4%B8%8E%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95.md#Non-deterministic%20%E8%A7%86%E8%A7%92)来理解.

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
> [MonadPlus](Monad%E4%B8%8E%E6%83%B0%E6%80%A7%E6%B1%82%E5%80%BC.md#MonadPlus)是Alternative和Monad的简单拼接, 能扩展出很多功能.

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

