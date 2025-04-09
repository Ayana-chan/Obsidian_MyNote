# 配置

## 环境变量（Linux）

在`/etc/profile`中加入以下文本以配置环境变量。

```bash
export GOROOT=/usr/local/go

export GOPATH=$HOME/gowork
export GOBIN=$GOPATH/bin
export PATH=$GOPATH:$GOBIN:$GOROOT/bin:$PATH

#go module
export GO111MODULE=on

export GOPROXY=https://goproxy.cn,direct
```

## vscode
### 新建mod文件时报错
[go module的工作区问题 - 那个人后来 - 博客园](https://www.cnblogs.com/ngrhl/p/15878099.html)

# Go Mod

[详解go语言包管理方式(go mod), 分析多目录多文件下的管理，不同工程下包的相互调用\_go 多文件\_强尼.的博客-CSDN博客](https://blog.csdn.net/qq_51721904/article/details/128231308)

[go mod使用 | 全网最详细 - 知乎](https://zhuanlan.zhihu.com/p/482014524?utm_id=0)

函数名首字母要大写才能被导出。

同一个目录下所有package名必须相同。

import的时候传入的是目录名；导入的对象的名字是它的package名。起别名相当于是给package起别名。


# 规范
## 命名规范

| 类型              | 风格                                                    | 样例                             |
| --------------- | ----------------------------------------------------- | ------------------------------ |
| Files           | lower_snake_case                                      | h2_bundle.go                   |
| Packages        | single noun (lower-case, compressed, prefer singular) | base64, arena, strconv         |
| Types           | camel case (prefer noun)                              | Cookie, uint64                 |
| Functions       | camel case (prefer verb)                              | SetCookie                      |
| Variables       | camel case                                            | Path                           |
| Constants       | camel case                                            | maxPad                         |
| Receivers       | single word, usually single lowercase letter          | `func (c *Cookie) String()`    |
| Type parameters | single word, usually single uppercase letter          | `func New[T any](a *Arena) *T` |

## 代码规范

map， func， chan 类型的receiver不要使用指针， 因为它们本身就是指针。

错误内容：除驼峰和缩写以外，禁止出现大写字母；禁止标点符号结尾。

# 基础

## 定义变量

```go
a := 1
var b int
var c = 1
const d = 1
```

对于结构体，`new(T)`和`&T{}`是等价的，都会给对象赋零值（一般人很少用new）。

```go
var a []int                      // nil

a := []int{}                     // not nil

a := *new([]int)                 // nil

a := make([]int,0)               // not nil
```

## 访问权限

Golang的所有东西的访问权限都是通过首字母大小写来决定的，大写=public，小写=private。

private无法被外部包访问、解析。

## 函数&方法

方法：`func`后紧接着用括号指出关联的类型（接收器）。

```go
//函数
func printName(s string) int { 
	fmt.Println(s) 
	return 3
}

//方法
func (p Person) valueShowName(s string) { 
	fmt.Println(p.name) 
	fmt.Println(s) 
}

//方法调用
person := Person{"Bill"}
person.valueShowName("abc")
```

## range

`for`循环中，使用`range`遍历数组，会返回两个变量，分别是下标和值。可以使用`_`来缺省一个变量。

for range出来的值都是拷贝，因此对数组进行遍历拿到的都是副本。想要节约拷贝成本，并且能够达到可变性的话，就只需要遍历index就行了，然后用index访问容器。

## 数据机制

**指针**，指向值，且可以直接使用`a.member`访问成员，不需要解引用。可以是nil

**不透明引用**，对其的拷贝不会发生深拷贝。除字符串外，都可以是nil。
- 字符串 `string`：底层的数据结构为 `stringStruct` ，里面有一个指针指向实际存放数据的字节数组，另外还记录着字符串的长度。不过由于 `string` 是只读类型（所有看起来对 `string` 变量的修改，实际上都是生成了新的实例），在使用上常常把它当做值类型看待。由于做了特殊处理，它甚至可以作为常量。`string` 也是唯一零值不为 `nil` 的引用类型。
- 切片（slice）：底层数据结构为 `slice` 结构体 ，整体结构跟 `stringStruct` 接近，只是多了一个容量（capacity）字段。数据存放在指针指向的底层数组里。
- 映射（map）：底层数据结构为 `hmap` ，数据存放在数据桶（buckets）中，桶对应的数据结构为 `bmap` 。
- 函数（func）：底层数据结构为 `funcval` ，有一个指向真正函数的指针，指向另外的 `_func` 或者 `funcinl` 结构体（`funcinl` 代表被行内优化之后的函数）。
- 接口（interface）：底层数据结构为 `iface` 或 `eface` （专门为空接口优化的结构体），里面持有动态值和值对应的真实类型。
- 通道（chan）：底层数据结构为 `hchan`，分别持有一个数据缓冲区，一个发送者队列和一个接收者队列。

**值类型**不能是nil。自定义的结构体也不能是nil。



## 匿名函数/闭包
可以在代码块内直接用`func`定义一个匿名函数（内部函数inner function）（闭包closure）：

```go
f:=func(s string){
	fmt.Println(s)
}
f(str)

//直接传参运行
func(s string){
	fmt.Println(s)
}(str)
```

如果在前面加上`go`，即可异步执行该匿名函数。

这在想要给一个GoRoutine的前后加些东西时特别有用，比如这段对异步dfs爬虫代码的调用：

```go
for _, u := range urls{
	done.Add(1)//在等待队列中注册自己
	//对u变量进行拷贝防止外部改变
	go func(u string){
		//爬取u
		ConcurrentMutex(u,fetcher,f)
		done.Done()//告诉等待队列该任务已结束
	}(u)
}
```

## defer

`defer fmt.Println("hello")`表示在当前函数执行完成之后马上调用，相当于没有try-catch的finally。

## select

写法类似Switch，但会先执行不阻塞的那个。

```go
select { 
case ch <- 1: 
	return nil 
default: 
	return errors.New("channel blocked, can not write") 
}
```

## 建立二维数组

不能使用变量直接建立二维数组。

```go
// MakeSlice2 创建二维切片
func MakeSlice2(len1, len2 int) [][]int{
	slice2 := make([][]int, len1)
	for i := range slice2 {
		slice2[i] = make([]int, len2)
	}
	return slice2
}
```


## 任意类型

`interface{}`（empty interface）表示可以接受任意类型，可以用在函数参数上。

## 空类型

`struct{}`表示空类型，而`struct{}{}`是对空类型的实例化。它不占用如何空间。

可以定义set为：
```go
type Set[T comparable] map[T]struct{}
```

## 切片
没定义长度的“数组”其实都叫切片，可以使用append和copy。

切片本质是一个指针，指向目标“数组”的一块内存空间，因此作为形参传入也能修改对应内存的值。

使用`[:]`切片是浅拷贝，即切片指向了一定区域的内存而不是截取复制。若想构造一个深拷贝出来的切片的话，需要使用append。超出append的cap值就会把一切东西深拷贝另起炉灶，而cap默认和长度一样。

## 跨级break和continue

使用`Loop:`标记最外层的for, 然后里面`break Loop`就能直接break掉最外层的for.
```go
Loop:
    for n := 0; n < len(src); n += size {
        switch {
        case src[n] < sizeOne:
            if validateOnly {
                break
            }
            size = 1
            update(src[n])

        case src[n] < sizeTwo:
            if n+1 >= len(src) {
                err = errShortInput
                break Loop
            }
            if validateOnly {
                break
            }
            size = 2
            update(src[n] + src[n+1]<<shift)
        }
    }
```

## interface

`interface`用于定义一系列函数签名。是**鸭子类型**机制，即只要有类型吻合了一个接口的所有签名要求，则这个类型可以被看做该接口。

直接把接口名字写在一个类型里面，就表示把接口的签名**组合**到该类型里面，于是这个类型就必然会符合接口的要求。

```go
type AaaSalr interface {
	Funca()
}

type aaaSal struct {
	AaaSalr
}
```

使用`i.(*S)`来把作为interface的i变量转变为类型`S`：
```go
if t, ok := i.(*S); ok {  
    fmt.Println("s implements I", t)  
}
```

获取与检测类型：
```go
switch t := i.(type) {  
case *S:  
    fmt.Println("i store *S", t)  
case *R:  
    fmt.Println("i store *R", t)  
}
```

### 判等

一个interfce相当于有一个`type T`和一个`value V`，只有这两个元素都相等的时候，两个interface才能相等。此外，只有当这两个都为空时，才能等于nil。

```go
var pi *int = nil
var pb *bool = nil

var x interface{} = pi
var y interface{} = pb
var z interface{} = nil

fmt.Println(x == y)   // false
fmt.Println(x == nil) // false
fmt.Println(x == z)   // false
```

## 枚举类型

没有独立的枚举类型，而是通过常量来定义。使用`itoa`来自动给类型按顺序连续赋值。规范上，给每个不同的枚举以不同的类型（单纯是`int`的别名）以做区分。
```go
type SameSite int

const (
    SameSiteDefaultMode SameSite = iota + 1
    SameSiteLaxMode
    SameSiteStrictMode
    SameSiteNoneMode
)

type ClientAuthType int

const (
    NoClientCert ClientAuthType = iota
    RequestClientCert
    RequireAnyClientCert
    VerifyClientCertIfGiven
    RequireAndVerifyClientCert
)
```

## 三个点 `...`


用作展开：
```go
x := []int{1,2,3}
y := []int{4,5,6}

x = append(x, y...) //而不是for循环
x = append(x, 4, 5, 6) //等价于上面的
```

用作可变参数列表：
```go
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

用作简化数组声明：
```go
var _ = [...]language{
        {"C", 1972},
        {"Python", 1991},
        {"Go", 2009},
}

var b = [...]string{0: "foo", 2: "foo"}        // [3]string{"foo", "", "foo"}
```

## 深拷贝

```go
s1 := []int{1,2,3}
s2 := append([]int{}, s1...)

// 效率更高的复制
s1 := []int{1,2,3}
s2 := make([]int, len(s1))
copy(s2, s1)
```

更复杂的类型，可以使用反射：
```go
age := 22
p := &Person{"Bob", &age}
v := reflect.ValueOf(p).Elem()
vp2 := reflect.New(v.Type())
vp2.Elem().Set(v)
```

## error

可以使用`errors.New("abc")`创建错误, 或者用`fmt.Errorf`以包含格式化文本。

`fmt.Errorf("reading srcfiles list: %w", err)`使用`: %w`把错误包含到错误链中。

`error.Unwrap`可以提取error的错误链的上一个error。

`error.Is`可以检测错误链上是否存在目标错误；`==`的话只能判断当前错误是否是目标错误（不推荐）。

`error.As`可以把一个错误提取到目标错误里面去，所提取的原错误链上与目标错误类型相同的错误。

### 自定义error

```go
// /src/os/error.go
// PathError records an error and the operation and file
// path that caused it.
type PathError struct {
    Op   string
    Path string
    Err  error
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}

func (e *PathError) Unwrap() error {
    return e.Err
}
```

## recover

写在defer里面（**只能写在defer**），可以接到函数内的panic。

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

## 临时文件

[Go 语言创建临时文件 ioutil.TempFile() - 简单教程，简单编程](https://www.twle.cn/t/383)

# 并行

## GoRoutine

GoRoutine是Go自己的协程/轻量级线程/用户级线程，由Go来调度。

## WaitGroup
`sync.WaitGroup`是Go实现的一种数据结构，仅仅可以计数，内部实现了互斥逻辑来避免竞争。可以用于记录有几个待完成任务。

它内置了一些方法：
- `Add(n)`: 计数器加n。
- `Done()`: 计数器减1。
- `Wait()`: 阻塞直到计数器为0。

## 保证WaitGroup.Done()必须执行

一开始就用`defer waitGroup.Done()`，使得接下来无论发生什么都会在最后执行Done。

[Defer概念](Golang.md#^a9re2i)

## 对匿名函数使用go语句相关问题
### 变量改变

匿名函数是可以使用外部变量的，但若外部改变了变量的话，匿名函数也会受影响。这种情况只在使用go来运行匿名函数的时候会出现。

经典的例子是for循环里面匿名函数配上go：
```go
for i:=0;i<5;i++{
	go func(){
		fmt.Println(i)
	}
}
```

这段代码不会达到我们的预期。假设第一次go语句执行，然后在开始打印前先进入了第二次for循环，此时第一次go会打印出1而非0。

解决方法是设置一个拷贝：
```go
for i:=0;i<5;i++{
	go func(i int){
		fmt.Println(i)
	}(i)
}
```

此时要多加考虑引用类型和值类型。

### 外部函数结束

内部函数还在异步执行时，外部函数结束了，但此时内部函数使用的外部函数变量不会被销毁。

因为编译器会检测变量是否被某内部函数使用。若是，则把变量分配到堆内存上而不是栈内存上，使内部函数能在外部函数结束后继续运行。

因此不用担心变量被销毁的问题，只需要注意变量是否改变。

>注意，主线程结束的话，GoRoutine也会被关闭！

## 条件变量 `sync.Cond`

条件变量可以看成一种中断源，可以取代轮询、忙等待。

`cond.Wait`: 释放自己获得的锁，并进行等待。当满足结束等待的条件时，会先去尝试获取锁，获取到后继续执行。

`cond.Broadcast`: 让所有使用了`cond.Wait`等待的GoRoutine继续执行。

而`cond.Signal`则仅让一个GoRoutine被唤醒。FIFO顺序。

## 未上锁却依旧正常执行（隐藏race）

由于GoRoutine是协程，少量的GoRoutine可能依旧是单线程的；即使被分配到三四个线程上，也极少出现冲突的情况。这部分的bug的发现是随机的，手动debug极为困难。

### race检测器

Go内置了race检测器，只需要在运行时加个参数，就能在运行时（不是静态检测，且占用内存）检测内存的访问是否存在race逻辑（而不是真的发生race）：

```bash
go run -race hello.go
```

## Channel

[Go Channel的基本使用及底层原理详解\_go channel原理\_xkzeee的博客-CSDN博客](https://blog.csdn.net/y1391625461/article/details/124292119)

>Go语言采用的并发模型是：通信顺序进程（CSP），提倡**通过通信来实现共享内存**而不是通过共享内存来实现通信

往常都是对某块内存上锁来完成线程间通信。而channel提供了一个安全的线程间传递信息的方式，将所要共享的内存数据存储并提供出去。

对于无缓存的channel只有当有GoRoutine往里塞数据并且另一个GoRoutine在取数据时，这两个GoRoutine才跳出阻塞继续运行。因此无法做到在同一个GoRouting内用同一个channel收发数据，会死锁。

有缓存的channel被塞满时添加数据会阻塞，否则不会阻塞。（极少数情况才要用到，比如发送方要一次发完多个数据然后干其他事情）

不要把channel看成queue，而是同步机制。

虽然本质也是有互斥锁逻辑在channel里面，但不要这么理解，而是仅仅将其看成传递消息的通道。

对chanel的range遍历会在空时阻塞等待直到有新值进入。

异步对channel传入初始值是因为写入也会被阻塞直到被读取，不异步的话可能会导致主线程阻塞。

## RPC

可能用到包`"net"`、`"net/http"`、`"net/rpc"`。

服务端：
```go
func (m *Master) server() {
    //此处将m对象给放入rpc注册了，可以在rpc相关方法中调用
    rpc.Register(m)
    rpc.HandleHTTP()
    //l, e := net.Listen("tcp", ":1234")
    sockname := masterSock()
    os.Remove(sockname)
    l, e := net.Listen("unix", sockname)
    if e != nil {
        log.Fatal("listen error:", e)
    }
    go http.Serve(l, nil)
}
```

客户端：

```go
func CallExample() {
    // declare an argument structure.
    args := ExampleArgs{}
  
    // fill in the argument(s).
    args.X = 99

    // declare a reply structure.
    reply := ExampleReply{}
  
    // send the RPC request, wait for the reply.
    call("Master.Example", &args, &reply)
  
    // reply.Y should be 100.
    fmt.Printf("reply.Y %v\n", reply.Y)
}

func call(rpcname string, args interface{}, reply interface{}) bool {
    // c, err := rpc.DialHTTP("tcp", "127.0.0.1"+":1234")
    sockname := masterSock()
    c, err := rpc.DialHTTP("unix", sockname)
    if err != nil {
        log.Fatal("dialing:", err)
    }
    defer c.Close()
    
    err = c.Call(rpcname, args, reply)
    if err == nil {
        return true
    }
    
    fmt.Println(err)
    return false
}
```

# Web 开发

由于没有依赖注入，因此需要在一层里面的一个主文件里面，定义该层所有组件（作为当前模块的全局变量）和初始化调用。例如，下面这样的定义可以使用`sals.AaaSal`访问：
```go
// sals/sals.go
var(
	AaaSal AaaSalr
	BbbSal BbbSalr
)

func Setup(){
	AaaSal = NewAaaSal()
	BbbSal = NewBbbSal()
}
```

在同一个文件里面同时定义接口类型和实现，其中实现是私有的：
```go
type AaaSalr interface {
	Funca()
}

type aaaSal struct {
	AaaSalr
}
```

## handler

在`handler.go`（和`handlers`文件夹同级）里面，给所有接口都再套一个wrap函数以处理错误和生成标准响应格式。

一种wrap函数方案，其中所有响应类型都实现了SetBaseResp，以存储数据以外的信息：
```go
// 所有完整响应类型都要实现（比如kitex自动生成的）
type respWithSetBaseResp interface {  
    SetBaseResp(val *base.BaseResp)  
}  

// 请求包装器
func wrapHandler[T respWithSetBaseResp](resp T, err error) (T, error) {  
    if err != nil && !reflect.ValueOf(resp).IsZero() {  
       resp.SetBaseResp(rpcHandlers.BuildErrBaseResp(err))  
    }  
    return resp, nil  
}

// 把所有嵌套出来的服务函数都塞给AaaServiceImpl以统一管理
type AaaServiceImpl struct{}

// 一个handler
func (s *Aaa) AaaDoSomething(ctx context.Context, request *aaa_type.AaaOneRequest) (resp *aaa_type.AaaOneResponse, err error) {
	return wrapHandler(rpcHandlers.AaaDoSomething(ctx, request))
}
```

## DAL

DAL（Data Access Layer）数据访问层。放在stores文件夹中。

使用`gorm.io/gorm`作为ORM。

对每个数据库，定义一个结构体以包含数据库一个接口`db`并给出创建函数；然后定义一些条件生成函数。定义对应的接口，里面写上各种自定义的复杂数据库操作。在其他文件里面实现这些复杂数据库操作。
```go
// my.go
type MyStorer interface {
	GetNameByAssetId(xxx)
}

type myStore struct {  
    db *gorm.DB  
}  
  
func NewMyStore(db *gorm.DB) MyStorer {  
    return &myStore{db: db}  
}

func inAaaIds(aaaIds []int64) dbs.Option {  
    return func(db *gorm.DB) *gorm.DB {  
       return db.Where("aaa_id in (?)", aaaIds)  
    }  
}

func equalBbbId(bbbId int64) dbs.Option {  
    return func(db *gorm.DB) *gorm.DB {  
       return db.Where("bbb_id = ?", bbbId)  
    }  
}

func moreThanCccId(cccId int64) dbs.Option {  
    return func(db *gorm.DB) *gorm.DB {  
       return db.Where("ccc_id > ?", cccId)  
    }  
}

// some_name.go
func (m *myStore) GetNameByAssetId(xxx) {
	xxx
}
```

# 库

## strings

前后处理：
```go
var s = "abaay森z众xbbab"
o := fmt.Println
o(strings.TrimPrefix(s, "ab")) // aay森z众xbbab
o(strings.TrimSuffix(s, "ab")) // abaay森z众xbb
o(strings.TrimLeft(s, "ab"))   // y森z众xbbab
o(strings.TrimRight(s, "ab"))  // abaay森z众x
o(strings.Trim(s, "ab"))       // y森z众x
o(strings.TrimFunc(s, func(r rune) bool {
        return r < 128 // trim all ascii chars
})) // 森z众
```

分割与合并：
```go
// "1 2 3" -> ["1","2","3"]
func Fields(s string) []string     // 用空白字符分割字符串

// "1|2|3" -> ["1","2","3"]
func Split(s, sep string) []string // 用sep分割字符串，sep会被去掉

// ["1","2","3"] -> "1,2,3"
func Join(a []string, sep string) string // 将一系列字符串连接为一个字符串，之间用sep来分隔

// Note:
// "1||3" -> ["1","","3"]
```

# 技巧

## 不要引用大数组

对大数组的小切片也会导致整个大数组无法被释放。因此，有时候总体上拷贝比引用更高性能。

# 特殊工具

## go generate

在任意位置使用`//go:generate`记录一个命令，之后运行`go generate`的时候就会运行此命令。 用于统一触发代码生成。
```go
//go:generate goyacc -o gopher.go -p parser gopher.y
```

> [!notice]
> `//`和`go`之间不要有空格！


# Arena

一次性申请一大块独立的内存, 然后在里面创建对象进行内存分配, 最后手动释放整个内存块, 以减少gc复杂度.

TODO

# 问题

goroutine会随着主线程的结束被迫关闭。

获取的文件名都是相对路径。

即使直接读取临界区遇到race也不会出bug，也应当去获取锁再操作，因为编译器优化可能导致出现意想不到的bug。编译器可能觉得一个for循环里面对一个变量的不断获取是多余的，但实际上这个变量可能被其他进程改变。









