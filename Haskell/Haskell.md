# Haskell

- [代数背景与类型](%E4%BB%A3%E6%95%B0%E8%83%8C%E6%99%AF%E4%B8%8E%E7%B1%BB%E5%9E%8B.md)
- [函数与基础语法](%E5%87%BD%E6%95%B0%E4%B8%8E%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95.md)
- [Semigroup与Monoid](Semigroup%E4%B8%8EMonoid.md)
- [Functor与Applicative](Functor%E4%B8%8EApplicative.md)
- [Monad与惰性求值](Monad%E4%B8%8E%E6%83%B0%E6%80%A7%E6%B1%82%E5%80%BC.md)
- [工具模块与调试](%E5%B7%A5%E5%85%B7%E6%A8%A1%E5%9D%97%E4%B8%8E%E8%B0%83%E8%AF%95.md)

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


转载参考单独保留：[Haskell - by tch0](Haskell%20-%20by%20tch0.md)。
