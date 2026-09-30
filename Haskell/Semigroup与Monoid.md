[← Haskell](Haskell.md)

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

`sconcat` 把 `NonEmpty a` 使用 `<>` 折叠成一个 a；它不是普通的 `[a]`，类型保证至少一个元素。
```haskell
ghci> :m Data.List.NonEmpty
ghci> sconcat $ Just [1, 2, 3] :| [Nothing, Just [4, 5, 6]]
Just [1,2,3,4,5,6]
```

`stimes` 把输入`a`重复`b`次, 使用`<>`折叠. 默认实现用$O(\log n)$次组合；总成本还取决于 `<>` 和求值结果，不能把生成长度 n 列表也算作对数总时间。 库中提供了若干`stimesXxx`函数作为替换实现, 以在特殊情景(约束)下<u>降低复杂度</u>.
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

