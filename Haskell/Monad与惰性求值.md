[← Haskell](Haskell.md)

## Monad

[Control.Monad](https://downloads.haskell.org/ghc/latest/docs/libraries/base-4.20.0.0-1f57/Control-Monad.html#t:Monad)

Monad typeclass是为了实现: 有一个包装后的值(**monadic value**), 和一个接收普通值返回被包装后的值的函数(**monadic function**, `Monad m => a -> m b`), 把此函数应用到包装内的值, 并且只返回一层包装(也是 monadic value). 强制使用`fmap`会得到`m (m b)`两层包装. **只有Applicative才能实现Monad**. 

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

下面这段 C++ 代码没有正常的递归终止路径（可能栈溢出等，不能保证具体报错）， 因为`factBad(x)`调用`myIfBad`的时候就不得不先把`factBad(x-1)`算出来, 出现了死递归(x会一直减少但不可能进入实际比较).
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
```cpp
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





