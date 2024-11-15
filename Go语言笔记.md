# Go语言笔记

## Go语言特点

- 简洁语法，类C语言，仅有25个关键字
- 内置垃圾回收
- 没有头文件
- 显式依赖(通过package关键字)
- 没有循环依赖
- 常量只是数字
- 首字母大小写决定可见性
- 任何类型都可以拥有方法
- 没有类，没有类型继承，没有父类子类的概念
- 没有构造函数，析构函数
- 没有模板，泛型
- 没有异常
- 没有算术转换
- 接口是隐式的，无需声明实现
- 没有++n和--n，只有n++和n--，且是语句不是表达式
- 赋值不是表达式
- 没有指针算术
- 内存总是初始化为零值
- 没有类型注解语法(如static等)
- 内置字符串，切片，map类型
- 内置数组边界检查
- 内置并发支持(编译器级别的并发优化，而非操作系统级别)，对多核CPU友好
- 标准库方便好用

## 基础语法

### 变量

变量命名规则: 小驼峰命名原则(开头字母小写，后续单词开头大写)

``` go
// 常量在创建时须进行声明，在大部分强类型语言中，“=”，即赋值运算符常与变量的声明相结合，做到声明并初始化
// 如：
var a int = 10
// 以上代码可以简化为
var a := 10
// Go虽然是强类型语言，但其支持自动类型推断
// 其中，“:=”，是，“=”赋值运算符的一种简化形式，即声明变量且赋其初值
// 但与“=”赋值运算符不同的是，“:=”对于统一变量仅可使用一次，因为该运算符包含对于该变量的声明
// 若对同一变量名重复使用，编译器会报panic

// 变量块常使用var ()的形式进行声明
var (
    
)
```

### 常量

注：Go推崇使用无类型常量进行常量定义

```go
// 常量块使用const ()的形式进行声明
// 无类型常量，无需显示指定常量类型，编译器可以自动推断
const (
    a = 5
    pi = 3.1415926
    str = "abc"
)

// 常量的隐式赋值
const (
    a, b = 11, 22
    c, d
    e, f
)
// 以上写法等价于以下写法
const (
    a, b = 11, 22
    c, d = 11, 22
    e, f = 11, 22
)
```

注：枚举也是一种常量，常通过iota来实现枚举

```go
const (
    sunday = iota
    monday = iota
    tuesday = iota
    wednesday = iota
    thursday = iota
    friday = iota
    saturday = iota
)
// 等价于
const (
    sunday = 0
    monday = 1
    tuesday = 2
    wednesday = 3
    thursday = 4
    friday = 5
    saturday = 6
)
// 更简洁的写法
const (
    sunday = iota
    monday
    tuesday
    wednesday
    thursday
    friday
    saturday
)
// 若想枚举从1开始，可以使用_占位符，则可以这样写
const (
    _ = iota
    sunday
    monday
    tuesday
    wednesday
    thursday
    friday
    saturday
)
```
内置原生类型的默认值
- 所有整型：0
- 浮点类型：0.0
- 布尔类型：false
- 字符串类型：“”
- 指针、interface、切片、channel、map、function：nil

> 零值可用的限制
``` go
// 零值可用的切片不能通过下标的形式操作
var s []int
s[0] = 12   // 报错
s = append(s, 12)   // 正确

// 像map这样的类型没有提供对零值可用的支持
var m map[string]int
m[“go”] = 1 // 报错

m1 := make(map[string]int)
m1[“go”] = 1 // 正确

// 零值可用要注意避免值复制
var mu sync.Mutex
mu1 := mu // 错误

// 函数传递是应通过引用传递
foo(mu) // 错误
foo(&mu) // 正确
```

### 切片
切片是Go语言在数组之上提供的一个重要的抽象数据结构，在绝大多数需要使用数组的场合中，切片实现了完美替代。

C语言的数组定义，如：
``` C
int a[10];
```

Go语言的数组定义，如：
``` Go
var a [10]int
```

C语言中，数组的定义是引用定义，即，数组名称是指向数组第一个元素的指针；
而在Go语言当中，数组的定义是值定义，即，一个数组变量表示的是整个数组。
所以，在Go语言中，当我们把整数组传递到函数中时，若数组中已经存有大量的元素，则会出现不小的性能损耗。在C语言中，常使用数组指针来定义函数参数；但在Go语言中，更“地道”的做法是使用切片。

**切片之于数组就像是文件描述符之于文件。**
此时数组将作为切片的底层存储空间，而切片则为使用者打开了操作数组的窗口。
**切片的本质是一个封装了一个数组的结构体。**
其中有三个字段，如：
``` Go
type slice struct {
    array unsafe.Pointer
    len int
    cap int
}
```
- array：指向下层数组的某元素指针，该元素也是切片的起始元素
- len：切片的长度，即切片当中当前元素的个数
- cap：切片的最大容量，cap >= len。

切片创建时，使用make关键字进行创建
``` s := make([]byte, 5) ```
还可以通过语法 ``` s[low: high] ``` 进行创建，如

``` Go
s := make([]byte, 5)
s1 := s[2, 4]
```

也常使用零切片定义空切片

```go
var s []string
```

切片支持动态扩容
具体的扩展算法在$GOROOT/src/runtime/slice.go中的growslice函数中。

当扩容导致切片的cap触碰到数组的上界，切片就会与原数组接触绑定，若原数组未被引用，则会被垃圾回收。

尽管动态扩容的特性非常方便操作，但是依旧建议在初始化时就给定cap参数，以此来进一步提高切片的性能。

### map

map又被称为字典或哈希表，由一组无序的键值对组成。

map中对value的类型没有严格限制，但是对key的类型有严格要求：key的类型应该严格定义了作为“==”和“!=”两个操作符的行为，因此，函数、map、切片均不能作为map的key类型。

map类型不支持零值可用，未显式赋初值的map类型变量的零值为nil。

map可以使用make关键字创建或使用复合字面值创建

与切片一样，map也是引用类型，将map类型变量传入函数不会有太大的性能损耗。

``` GO
m := make(map[K]V)
// 插入元素
m[k1] = v1
m[k2] = v2
m[k3] = v3

// 使用Go的内置函数
// 获取元素个数
fmt.Println(len(m))
// 删除元素
delete(m, “k2”)
fmt.Println(len(m))
// 注：即使要删除的key不在map中，delete函数也不会导致panic

// 查找
// 这里使用“comma ok”惯用法来进行查找
_, ok := m[“key”]
if !ok {
    // “key”不在map中
}

// 取值
// 依旧使用“comma ok”惯用法
v, ok := m[“key”]
if !ok {
    // “key”不在map中
}
fmt.Println(v)
// 注：我们需要通过ok是否为true来判断key是否存在于map中

// 遍历
for k, v := range m {
    fmt.Println(“[%d, %d]”, k, v)
}
```



### string

> 特性

1. string类型的数据是不可变的，想要改变其中的字符只能重新申请空间
2. 零值可用，即空串为 ``` "" ```
3. 获取长度时的时间复杂度为O(1)，由于特性1，所以每个字符串实际上时定长的
4. 支持通过+/+=操作符进行字符串连接
5. 支持通过，如：==、!=、>=、<=、>和<进行比较

```go
s1 := "12345"
s2 := "23456"
fmt.Println(s1 < s2)	// true
fmt.Println(s1 <= s2)	// true

s1 = "12345"
s2 = "123"
fmt.Println(s1 > s2)	// true
fmt.Println(s1 >= s2)	// true
```

同样是因为特性1，因此如果两个字符串长度不相同，则它们一定不相同。如果它们长度相同，则需要比较它们其中的数据指针是否指向同一块空间，如果不同，则还需要进一步比较其实际空间中的数据内容

6. 对非ASCII字符提供原生支持
7. 支持多行字符串

```go
s1 := `
abcd
efgh
`
fmt.Println(s1)

// abcd
// efgh
```



> 字符串构建方式及特点

- fmt.Sprintf

该方式常用于格式字符串的构建，如：

```go
host := "127.0.0.1"
port := 54000
url := fmt.Sprintf("%s:%d", host, port)
```

- strings.Join

该方式类似于常用于连接字符串，在 ``` filepath ``` 中也有类似的函数，在构建文件目录路径的时候尤为好用，如

```go
dir, _ := os.ReadDir(rootPath)
for _, file := range dir {
    absolutePath := filepath.Join(rootPath, file.Name())
}
```

- strings.Builder

相较于其他构建方法，拥有最高的构建效率

- bytes.Buffer

``` Buffer ``` 为 ``` bytes ``` 库中提供的字节数组缓冲区，用以拼接字节数组，同时，其也提供了将其中的字节数组转换成字符串的功能

```go
var buf bytes.Buffer
buf.Write(response.Body)
jsStr := buf.String()
```

从性能层面分析以上方法， ``` strings.Builder ``` > ``` strings.Join ``` > ``` bytes.Buffer ``` >> ``` fmt.Sprintf ```，但以上方法各有用处，不能以性能论其优劣。

### 包



#### 包结构



#### init函数



#### main函数



### 函数

基本特性：

- 函数可以像变量一样被赋值，传递给变量，作为参数传递，作为返回值返回，在函数内部创建
- 函数可以像变量那样被显式转换

#### 函数的显式转换

> 例子1

``` go
// 这里将greeting函数转换成了Handler接口的实现方法
// server.go
type Handler interface {
	ServerHTTP(ResponseWriter, *Request)
}

type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServerHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}

func ListenAndServer(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServer()
}


import main

func greeting(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Welcome, Gopher!\n")
}

func main() {
	http.ListenAndServer(":8080", http.HandlerFunc(greeting))
}
```

> 例子2

``` go
// 这里将MyAdd转换成BinaryAdder的实现
type BinaryAdder interface {
    Add(int, int) int
}

type MyAdderFunc func(int, int) int

func (f MyAdderFunc) Add(x, y int) int {
    return f(x, y)
}

func MyAdd(x, y int) int {
    return x + y
}

func main() {
    var i BinaryAdder = MyAdderFunc(MyAdd)
    fmt.Println(i.Add(5, 6))
}
```



#### 函数式编程

>  柯里化函数

``` go
package main

import "fmt"

func times(x, y int) int {
    return x * y
}

func partialTimes(x int) func(int) int {
    // 闭包， 闭包是函数内部定义的匿名函数，并且允许匿名函数访问定义它的外部函数作用域
    return func(y int) int {
        return times(x, y)
    }
}

func main() {
    timesTwo := partialTimes(2)
    timesThree := partialTimes(3)
    timesFour := partialTimes(4)
    // 实质上调用的是times(2, y)
    fmt.Println(timesTwo(5))
    fmt.Println(timesThree(5))
    fmt.Println(timesFour(5))
}

// 运行结果
// 10
// 15
// 20
```

> 函子

- 函子本身是一个容器类型，如切片，map甚至channel
- 该容器类型需要实现一个方法，该方法接受一个函数作为参数，并在容器的每个元素上应用那个函数，得到一个新函子，原函子容器内的元素值不受影响

``` go
type IntSliceFunctor interface {
    Fmap(fn func(int) int) IntSliceFunctor
}

type intSliceFunctorImpl struct {
    ints []int
}

func (isf intSliceFunctorImpl) Fmap(fn func(int) int) IntSliceFunctor {
    newInts := make([]int, len(isf.ints))
    for i, elt := range isf.ints {
        retInt := fn(elt)
        newInts[i] = retInt
    }
    return intSliceFunctorImpl{ints: newInts}
}

func NewIntSliceFunctor(slice []int) IntSliceFunctor {
    return intSliceFunctorImpl{ints: slice}
}

func main() {
    intSlice := []int{1, 2, 3, 4}
    fmt.Println("init a functor from int slice: %#v\n", intSlice)
    f := NewIntSliceFunctor(intSlice)
    fmt.Println("original functor: %+v\n", f)
    
    mapperFuinc1 := func(i int) int {
        return i + 10
    }
    
    mapped1 := f.Fmap(mapperFunc1)
    fmt.Println("mapped functor1: %+v\n", mapped1)
    
    mapperFunc2 := func(i int) int {
        return i * 3
    }
    mapped2 := mapped2.Fmap(mapperFunc2)
    fmt.Println("mapped functor2: %+v\n", mapped2)
    fmt.Println("original functor: %+v\n", f)
    fmt.Println("composite functor: %+v\n", f.Fmap(mapperFunc1).Fmap(mapperFunc2))
}

// init a functor from int slice: []int{1, 2, 3, 4}
// original functor: {ints:[1 2 3 4]}
// mapped functor1: {ints:[11 12 13 14]}
// mapped functor2: {ints:[33 36 39 42]}
// original functor: {ints:[1 2 3 4]}
// composite functor: {ints:[33 36 39 42]}
```

以上代码的说明：

- 定义了一个intSliceFunctorImpl类型，用来作为函子的载体。
- 我们把函子要实现的方法命名为Fmap，intSliceFunctorImpl类型实现了该方法。该方法也是IntSliceFunctor接口的唯一方法。可以看到在这个代码中，真正的函子其实是IntSliceFunctor，这符合Go的惯用法。
- 我们定义了创建IntSliceFunctor的函数NewIntSliceFunctor，通过该函数以及一个初始化切片，我们可以实例化一个函子。
- 我们在main中定义了两个转换函数，并将这两个函数应用到上述函子实例。得到的新函子的内部容器元素值是原容器的元素值经由转换函数转换后得到的。
- 最后，我们还可以对最初的函子实例连续（组合）应用转换函数，这让我们想到了数学课程中的函数组合。
- 无论如何应用转换函数，原函子中容器内的元素值不受影响。

#### defer关键字

- 在Go中，**只有在函数和方法内部才能使用defer**
- defer关键字后面**只能接函数或方法**，这些函数被称为deferred函数。defer将它们注册到其所在的goroutine用于存放deferred函数的栈数据结构中，这些deferred函数将在执行defer的函数退出前被按后进先出的顺序调度。

defer关键字有一下作用：

> 拦截panic

```go
// 拦截panic
func bar() {
    fmt.Println("raise a panic")
    panic(-1)
}

func foo() {
    defer func() {
        if e := recover(); e != nil {
            fmt.Println("recovered from a panic")
        }
    }()
    bar()
}

func main() {
    foo()
    fmt.Println("main exit normally")
}
// 运行结果:
// raise a panic
// recovered from a panic
// main exit normally

// 释放资源
var mu sync.Mutex

func f() {
    mu.Lock()
    defer mu.Unlock
    bizOperation()
}

func g() {
    mu.Lock()
    bizOperation()
    mu.Unlock
}

// 拦截panic的意外，无法拦截的panic
// #include <stdio.h>
// void crash() {
// 		int *q = NULL;
//		(*q) = 15000;
//		printf("%d\n", *q)
// }
package main
import "C"

import (
    "fmt"
)

func bar() {
    C.crash()
}

func foo() {
    defer func() {
        if e := recover(); e != nil {
            fmt.Println("recovered from a panic")
        }
    }()
    bar()
}

func main() {
    foo()
    fmt.Println("main exit normally")
}
// 运行结果:
// SIGILL: illegal instruction
// ...
```



> 修改函数的具名返回值

```go
func foo(a, b int) (x, y int) {
    defer func() {
        x = x * 5
        y = y * 10
    }()
    
    x = a + 5
    y = b + 6
    return
}

func main() {
    x, y := foo(1, 2)
    fmt.Println("x = ", x, "y = ", y)
}
// 运行结果:
// x = 30 y = 80
```



> 输出调试信息

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
// 运行结果:
// entering:b
// in: b
// entering:a
// in: a
// leaving:a
// leaving:b
```



> 还原变量旧值

```go
func init() {
    oldFsinit := fsint
    defer func() {
        fsinit = oldFsinit
    }()
    fsinit = func() { }
    Mkdir(...)
    Mkdir(...)
    mkdev(...)
    mkdev(...)
    mkdev(...)
    mkdev(...)
    chdirEnv()
}
```



不是所有的函数都允许被defer注册为deferred的，如：append、cap、len、make、new等内置函数不能作为deferred函数。

#### receiver类型

与函数相比，方法在声明形式上仅仅多了一个参数，Go称之为receiver参数。**receiver参数是方法与类型之间的纽带。**

方法的一般声明形式如下：

func	(receiver T/*T)	MethodName(参数列表)	返回值列表 { 
			// 方法体
} 

方法的特点有：

- 方法名的首字母是否大写将决定该方法是不是导出方法（该特点对于函数也同样适用）
- 方法定义要与类型定义放在同一个包内。
- 每个方法只能有一个receiver参数，不支持多receiver参数列表或变长receiver参数。一个方法只能绑定一个基类型。
- receiver参数的基类型本身不能是指针类型或接口类型，但可以是类型的指针。
- receiver参数的基类型可以是类型的指针、类型本身；这里的类型既可以指的是结构体类型，也可以指的是函数类型，甚至可以是基础类型。

由此我们可以得出：

- 不能为原生类型（如int、float64、map等）添加方法，只能为自定义类型定义方法。
- 不能横跨Go包为其他包内的自定义类型定义方法。

> 方法的本质

方法的定义：

```go
type T struct {
    a int
}

func (t T) Get() int {
    return t.a
}

func (t *T) Set(a int) int {
	t.a = a
    return t.a
}
```

以下的两个函数是上面方法的等价转换函数，也成为方法的原型

```go
func Get(t T) int {
    return t.a
}

func Set(t *T, a int) int {
    t.a = a
    return t.a
}
```

方法的使用：

```go
var t T
t.Get()
t.Set(1)

// 我们亦可以这样使用
T.Get(t)
(*T).Set(&t, 1)
```

直接以类型名T调用方法的表达方式被称为方法表达式。类型 T只能调用T的方法集合中的方法，*T只能调用 *T 的方法集合中的方法。方法表达式的调用方法与之前方法与函数的等价转换如出一辙，因此，Go方法的本质就是：

**一个以方法所绑定类型实例为第一个参数的普通方法**

> receiver类型

func	（t	T）	M1()	<=>	M1(t	T)

func	（t	*T）	M2()	<=>	M2(t	*T)

注：**不论receiver类型是T类型还是 *T ，使用T类型的实例都可以将其调用；同理， *T 类型的实例也可以调用receiver类型为T类型的方法**

- 当receiver参数的类型为T时，Go函数的参数采用值传递，即函数内部无法改变T类型实例中成员的值。
- 当receiver参数的类型为 *T 时，Go函数的参数采用引用传递，即传递进函数的是T类型实例的地址，此时对该实例进行的任何修改都会反映到原T类型实例上。

大部分时候都推荐使用 *T 作为receiver的参数类型，只在T类型实例的size不大且内部函数处理只读时使用值传递。

一个值得思考的例子

``` go
type field struct {
    name string
}

func (p *field) print() {
    fmt.Println(p.name)
}

func main() {
    data1 := []*field{{"one"}, {"two"}, {"three"}}
    for _, v := range data1 {
        go v.print()
    }
    
    data2 := []field{{"one"}, {"two"}, {"three"}}
    for _, v := range data2 {
        go v.print()
    }
    
    time.Sleep(3 * time.Second)
}
// data2的输出是什么？为什么？
```

通过方法表达式转换后如下（ ```t.Set(1)``` <=> ```(*T).Set(&t, 1)``` ）：

``` go
type field struct {
    name string
}

func (p *field) print() {
    fmt.Println(p.name)
}

func main() {
    data1 := []*field{{"one"}, {"two"}, {"three"}}
    for _, v := range data1 {
        go (*field).print(v)
    }
    
    data2 := []field{{"one"}, {"two"}, {"three"}}
    for _, v := range data2 {
        go (*field).print(&v)
    }
    
    time.Sleep(3 * time.Second)
}
```

#### 方法决定接口实现

```go
type Interface interface {
    M1()
    M2()
}

type T struct {}

func (t T) M1() {}
func (t *T) M2() {}

func main() {
    var t T
    var pt *T
    var i Interface
    
    i = t
    i = pt
}
```

> 方法集合

receiver类型的选择除了要考虑是否需要对类型实例进行修改、类型实例值复制导致的性能损耗之外，还有一个重要的考量因素，那就是**类型是否要实现某个接口**

自定义类型与接口之间是松耦合的：**如果某个自定义类型T的方法集合是某个接口类型的方法集合的超集，那么就说类型T实现了该接口，并且类型T的实例变量可以被赋值给该接口类型的变量**，即我们说的**方法集合决定接口实现**

```go
// 工具函数，用以输出一个自定义类型或接口类型的方法集合
func DumpMethodSet(i interface{}) {
    v := reflect.TypeOf(i)
    elemTyp := v.Elem()
    
    n := elemTyp.NumMethod()
    if n == 0 {
        fmt.Println("%s'smethod set is empty!\n", elemTyp)
        return
    }
    
    fmt.Println("%s's method set:\n", elemTyp)
    for j := 0; j < n; j++ {
        fmt.Println("-", elemTyp.Method(j).Name)
    }
    fmt.Println("\n")
}
```

通过该函数检测上述例子中的t，pt

```go
func main() {
    var t T
    var pt *T
    DumpMethodSet(&t)
    DumpMethodSet(&pt)
    DumpMethodSet((*Interface)(nil))
}

// 输出
`
main.T's method set:
- M1

main.T's method set:
- M1
- M2

main.Interface's method set:
- M1
- M2
`
```

> defined类型的方法集合

```go
type MyInterface I
type MyStruct T
```

已有类型如上面的I和T被称为underlying类型，而新类型被称为defined类型

对于接口，defined类型将继承underlying类型的方法集合

对于结构体，defined类型不继承underlying类型的方法集合，defined类型中的方法集合默认为空

**方法集合决定接口实现**，即便underlying类型实现了某些接口，基于其创建的defined类型也没有“继承”这一隐式关联。

> 类型别名的方法集合

类型别名的语法

```go
type MyInterface = I
type MyStruct = T
```

类型别名的方法集合和原类型相同，不论对于接口亦或是结构体。

#### 变参函数

> 语法糖

```go
// 定义
func fun(args ...T) {
    ...
}

// 使用
func main() {
    // 方式1
    a, b, c := 1, 2, 3
    println(fun(a, b, c))
    // 方式2
    nums := []T{4, 5, 6}
    println(fun(nums...))
}
```

**在这两种使用方式中，不可以混用，每次使用时只能选择一种使用方式。**

> 通过变参函数实现功能选项模式

```go
type FinishedHouse struct {
    style					int		// 0: Chinese; 1: American; 2: Eurpean
    centralAirConditioning	bool
    floorMaterial			string  // "ground-tile"或"wood"
    wallMaterial			string  // "latex"或"paper"或"diatom-mud"
}

type Option func(*FinishedHouse)

func NewFinishedHouse(options ...Option) *FinishedHouse {
    h := &FinishedHouse{
        // default options
        style:					0,
        centralAirConditioning:	true,
        floorMaterial:			"wood"
        wallMaterial:			"paper"
    }
    
    for _, option := range options {
        option(h)
    }
    
    return h
}

func WithStyle(style int) Option {
    return func(h *FinishedHouse) {
        h.style = style
    }
}

func WithFloorMaterial(meterial string) Option {
    return func(h *FinishedHouse) {
        h.floorMaterial = material
    }
}

func WithWallmaterial(meterial string) Option {
    return func(h *FinishedHouse) {
        h.wallMaterial = material
    }
}

func WithCentralAirConditioning(centralAirConditioning bool) Option {
    return func(h *FinishedHouse) {
        h.centralAirConditioning = centralAirConditioning
    }
}

func main() {
    // 默认选项
    fmt.Printf("%+v\n", NewFinishedHouse())
    // 自定义选项
    fmt.Printf("%+v\n", NewFinishedHouse(WithStyle(1), 
        WithFloorMaterial("groud-tile"),
        WithCentralAirConditioning(false)))
}
```



### 接口

#### 动态特性

接口类型的变量在程序运行时可以被赋值为不同类型的动态类型变量，从而支持运行时**多态**。

反之，将任意类型的变量赋值给一个接口类型的变量时，将会出发**装箱**操作；装箱，是编程语言领域的一个基础概念，一般是指把值类型转换成引用类型。

```go
a := 10
var i interface{}
i = a
```

在经过装箱之后，原类型所使用的空间，与接口变量中该类型变量所占用的空间已无任何联系，因为当将任意类型的变量赋值给一个接口类型的变量时，Go编译器将触发装箱操作，操作时，会将原变量所指向的空间中的值拷贝到一块新的内存空间中，并将新空间的首地址赋值给接口中的指针。装箱操作后，即使改变了原变量的值，接口变量的值也不会被改变。

```go
func main() {
    var n int = 61
    var ei interface{} = n
    n = 62
    fmt.Println(ei)
}
```



#### 静态特性

**接口的实质是一组方法的签名集合**，在编译阶段会进行类型检查：当一个接口类型变量被赋值时，编译器会检查右值的类型是否实现了该接口方法集合中的所有方法。

> 例子1

```go
// 这里将greeting函数转换成了Handler接口的实现方法
// server.go
type Handler interface {
	ServerHTTP(ResponseWriter, *Request)
}

type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServerHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}

func ListenAndServer(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServer()
}


import main

func greeting(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Welcome, Gopher!\n")
}

func main() {
	http.ListenAndServer(":8080", http.HandlerFunc(greeting))
}
```

> 例子2

```go
// scrape@Sophos.go
package scrape

type SophosScrape struct {
}

type ISophosScrape interface {
	UseCollyScrapeSophos(url string) (bool, string)
}

var sophosLock sync.Mutex

func (sophosScrape SophosScrape) UseCollyScrapeSophos(url string) (bool, string) {
    // 函数的具体实现
}

// scrape@SonicWall.go
package scrape

type SonicWallScrape struct {
}

type ISonicWallScrape interface {
	UseCollyScrapeSonicWall(url string) (bool, string)
}

var sonicWallLock sync.Mutex

func (sonicWallScrape SonicWallScrape) UseCollyScrapeSonicWall(url string) (bool, string) {
    // 函数的具体实现
}

// main.go
package main

type ScrapeFuncs interface {
	scrape.ISonicWallScrape
	scrape.ISophosScrape
}

type Scraper struct {
	scrape.SonicWallScrape
	scrape.SophosScrape
}

func main() {
	flag.Parse()
	var wg sync.WaitGroup
	var funcs ScrapeFuncs
	var scraper Scraper
	funcs = scraper
	targets := input.Dispatch()

	SonicWallScrapeTitle := "SonicWall"
	wg.Add(1)
	go func() {
		sonicWallRes, sonicWallLogs := taskBuilder.RunScrape(SonicWallScrapeTitle, funcs.UseCollyScrapeSonicWall, targets)
		output.Output(SonicWallScrapeTitle, sonicWallRes, sonicWallLogs)
		wg.Done()
	}()

	SophosScrapeTitle := "Sophos"
	wg.Add(1)
	go func() {
		sophosRes, sophosLogs := taskBuilder.RunScrape(SophosScrapeTitle, funcs.UseCollyScrapeSophos, targets)
		output.Output(SophosScrapeTitle, sophosRes, sophosLogs)
		wg.Done()
	}()

	wg.Wait()
}

// taskRunner.go
import taskBuilder
func RunScrape(taskTitle string, s func(url string) (bool, string), targets []string) ([]string, []string) {
    // 任务分配
}
```



#### 对Go中的OO特性的一点思考(OO，object-oriented，面向对象)

```go
type P struct {
    A int
    b string
}

func (P) M1() {
}

func (P) M2() {
}

type Q struct {
    c [5]int
    D float64
}

func (Q) M3() {
}

func (Q) M4() {
}

type T struct {
    P
    Q
    E int
}

func main() {
    var t T
    t.M1()
    t.M2()
    t.M3()
    t.M4()
    println(t.A, t.D, t.E)
}
```

我们看到类型T通过嵌入P、Q两个类型，“继承”了P、Q的导出方法(M1~M4)和导出字段(A、D)。

不过实际Go中的这种“继承”机制并非经典OO中的继承，其外围类型(T)与嵌入的类型(P、Q)之间没有任何“亲缘”关系。P、Q的导出字段和导出方法只是被提升为T的字段和方法罢了，**其本质是一种组合**，是组合中的代理（delegate）模式的一种实现。T只是一个代理（delegate），对外它提供了它可以代理的所有方法，如例子中的M1~M4方法。当外界发起对T的M1方法的调用后，T将该调用委派给它内部的P实例来实际执行M1方法。

以经典OO理论话术去理解就是**T与P、Q的关系不是is-a，而是has-a的关系**



### 并发

#### Go提供的并发原语

- goroutine：对用GPM模型中的P，封装了数据的处理逻辑，是Go运行时调度的基本执行单元。
- channel：对应GPM模型中的**输入/输出原语**，用于goroutine之间的**通信和同步**。
- select：用于应对多路输入/输出，可以让goroutine同时协调处理多个channel操作。

#### 并发模式

##### 创建模式

```go
// go关键字+函数/方法创建goroutine
go fmt.Println("I am a goroutine")

c := srv.newConn(rw)
go c.server(connCtx)

// 使用闭包，channel来创建goroutine
type T struct {...}

func spawn(f func()) chan T {
    c := make(chan T)
    go func() {
        // 使用channel变量c(通过闭包方式)与调用spawn的goroutine通信
        ...
        f()
        ...
    } ()
    
    return c
}

func main() {
    c := spawn(func() {})
    // 使用channel变量c与新创建的goroutine通信
}
```

第二种方式中通过spawn函数建立起了**spawn内部创建的goroutine和调用spawn的goroutine之间的联系**，两个goroutine通过spawn函数返回的channel进行通信。这是更常用的创建方式。

##### 退出模式

goroutine的使用代价很低，Go官方推荐多使用goroutine。goroutine函数执行结束即意味着goroutine推出，但一些常驻的后台服务程序可能会对goroutine有着优雅推出的要求。

###### 分离模式

顾名思义创建者在创建了它的goroutine之后就不需要关心它的推出，这类goroutine函数返回即退出。该模式有两种用途：

- 一次性任务

```go
// $GOROOT/src/net/dial.go
func (d *Dialer) DialContext(ctx context.Context, network, address string) (Conn, error) {
    ...
    if oldCancel := d.Cancel; oldCancel != nil {
        subCtx, cancel := context.WithCancel(ctx)
        defer cancel()
        go func() {
            select {
                case <-oldCancel:
	                cancel()
                case <-subCtx.Done():
            }
        }()
        ctx = subCtx
    }
    ...
}
```

以上代码的含义为，在DialContext方法中创建了一个goroutine，用来监听两个channel是否有数据，一旦有数据，处理后退出。

- 常驻后台的任务



###### join模式

goroutine的创建者需要等待新goroutine结束

- 等待一个goroutine退出

```go
func worker(args ...interface{}) {
    if len(args) == 0 {
        return
    }    
    interval, ok := args[0].(int)
    if !ok {
        return
    }
    time.Sleep(time.Second * (time.Duration(interval)))
}

func spawn(f func(args ...interface{}), args ...interface{}) chan struct{} {
    c := make(chan struct{})
    go func() {
        f(args...)
        c <- struct{}{}
    }()
    return c
}

func main() {
    done := spawn(worker, 5)
    println("spawn a worker goroutine")
    <-done
    println("worker done")
}
```



- 获取goroutine的退出状态

```go
var OK = errors.New("ok")

func worker(args ...interface{}) error {
    if len(args) == 0 {
        return errors.New("invalid args")
    }
    interval, ok := args[0].(int)
    if !ok {
        return errors.New("invalid interval args")
    }
    time.Sleep(time.Second * (time.Duration(interval)))
    return OK
}

func spawn(f func(args ...interface{}) error, args ...interface{}) chan error {
    c := make(chan error)
    go func() {
        c <- f(args...)
    }()
    return c
}

func main() {
    done := spawn(worker, 5)
    println("spawn worker1")
    err := <-done
    println("worker1 done:", err)
    done := spawn(worker)
    println("spawn worker2")
    err := <-done
    println("worker2 done:", err)
}
```



- 等待多个goroutine退出

```go
func worker(args ...interface{}) {
    if len(args) == 0 {
        return
    }    
    interval, ok := args[0].(int)
    if !ok {
        return
    }
    time.Sleep(time.Second * (time.Duration(interval)))
}

func spawn(n int, f func(args ...interface{}), args ...interface{}) chan struct{} {
    c := make(chan struct{})
    var wg sync.WaitGroup
    
    for i := 0; i < n; i++ {
        wg.Add(1)
        go func(i int) {
            name := fmt.Sprintf("worker-%d:", i)
            f(args...)
            println(name, "done")
            wg.Done()
        }(i)
    }
    
    go func() {
        // 该方法将会等待所有wg.Done()调用完毕后才会返回
        wg.Wait()
        c <- struct{}{}
    }()
    
    return c
}

func main() {
    done := spawn(5, worker, 3)
    println("spawn a group of workers")
    <-done
    println("group workers done")
}
```



- 支持超时机制的等待

```go
func worker(args ...interface{}) {
    if len(args) == 0 {
        return
    }    
    interval, ok := args[0].(int)
    if !ok {
        return
    }
    time.Sleep(time.Second * (time.Duration(interval)))
}

func spawn(n int, f func(args ...interface{}), args ...interface{}) chan struct{} {
    c := make(chan struct{})
    var wg sync.WaitGroup
    
    for i := 0; i < n; i++ {
        wg.Add(1)
        go func(i int) {
            name := fmt.Sprintf("worker-%d:", i)
            f(args...)
            println(name, "done")
            wg.Done()
        }(i)
    }
    
    go func() {
        // 该方法将会等待所有wg.Done()调用完毕后才会返回
        wg.Wait()
        c <- struct{}{}
    }()
    
    return c
}

func main() {
    done := spawn(5, worker, 3)
    println("spawn a group of workers")
    
    timer := time.NewTimer(time.Second * 5)
    defer timer.Stop()
    select {
        case <-timer.C:
        	println("wait group workers exit timeout")
        case <-done:
        	println("group workers done")
    }
}
```



###### notify-and-wait模式

- 通知并等待要给goroutine退出

```go
func worker(j int) {
    time.Sleep(time.Second * (time.Duration(j)))
}

func spawn(f func(int)) chan string {
    quit := make(chan string)
    go func() {
        var job chan int // 模拟job channel
        for {
            select {
            case j := <-job:
                f(j)
            case <- quit:
                quit <- "ok"
            }
        }
    }()
    return quit
}

func main() {
    quit := spawn(worker)
    println("spawn a worker goroutine")
    
    time.Sleep(5 * time.Second)
    
    println("notify the worker to exit...")
    quit <- "exit"
    
    timer := time.NewTimer(time.Second * 10)
    defer timer.Stop()
    select {
    case status := <-quit:
        println("worker done:", status)
    case <-timer.C:
    	println("wait worker exit timeout")
    }
}
```



- 通知并等待多个goroutine退出

channel有一个特性，当使用close函数关闭channel时，所有阻塞到该channel上的goroutine都会得到通知。

```go
func worker(j int) {
    time.Sleep(time.Second * (time.Duration(j)))
}

func spawn(n int, f func(int)) chan string {
    quit := make(chan struct{})
    job := make(chan int)
    var wg sync.WaitGroup
    
    for i := 0; i < n; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done() // 保证wg.Done在goroutine退出前被执行
            name := fmt.Sprintf("worker-%d:", i)
            for {
                j, ok := <-job
                if !ok {
                    println(name, "done")
                    return
                }
                // 执行这个job
                worker(j)
            }
        }(i)
    }
    
    go func() {
        <-quit
        close(job) // 广播给所有新goroutine
        wg.Wait()
        quit <- struct{}{}
    }()
    return quit
}

func main() {
    quit := spawn(5, worker)
    println("spawn a group of workers")
    
    time.Sleep(5 * time.Second)
    // 通知 worker goroutine 组退出
    println("notify the worker group to exit...")
    quit <- struct{}{}
    
    timer := time.NewTimer(time.Second * 5)
    defer timer.Stop()
    select {
    case <-quit:
        println("group workers done")
    case <-timer.C:
    	println("wait group workers exit timeout")
    }
}
```

##### 退出模式的应用

这里定义一个统一的超时退出的接口

``` go
type GracefullShutDowner interface {
    Shutdown(waitTimeout time.Duration) error
}

type ShutdownerFunc func(time.Duration) error

func (f ShutdownerFunc) Shutdown(waitTimeout time.Duration) error {
    return f(waitTimeout)
}
```

- 并发退出

```go
func ConcurrentShutdown(waitTimeout time.Duration, shutdowners ...GracefullShutDowner) error {
    c := make(chan struct{})
    
    go func() {
        var wg sync.WaitGroup
        for _, g := range shutdowners {
            wg.Add(1)
            go func(shutdowner GracefullShutDowner) {
                defer wg.Done()
                shutdowner.Shutdown(waitTimeout)
            }(g)
        }
        wg.Wait()
        c <- struct{}{}
    }
    
    timer :=  time.NewTimer(waitTimeout)
    defer timer.Stop()
    
    select {
    case <-c:
        return nil
    case <-timer.C:
        return errors.New("wait timeout")
    }
}
```

测试用例

```go
func ShutdownMaker(processTm int) func(time.Duration) error {
    return func(time.Duration) error {
        time.Sleep(time.Second * time.Duration(processTm))
        return nill
    }
}

func TestConcurrentShutdown(t *testing.T) {
    f1 := shutdownMaker(2)
    f2 := shutdownMaker(6)
    
    err := ConcurrentShutdown(10 * time.Second, ShutdownMaker(f1), ShutdownMaker(f2))
    if err != nil {
        t.Errorf("want nil, actual: %s", err)
        return
    }
    
    err := ConcurrentShutdown(4 * time.Second, ShutdownMaker(f1), ShutdownMaker(f2))
    if err == nil {
        t.Errorf("want timeout, actual: nil")
        return
    }
}
```

- 串行退出

```go
func SquentialShutdown(waitTimeout time.Duration, shutdowners ...GracefullyShutdowner) error {
    start := time.Now()
    var left time.Duration
    timer := time.NewTimer(waitTimeout)
    
    for _, g := range shutdowners {
        elapsed := time.Since(start)
        left = waitTimeout - elapsed
        
        c := make(chan struct{})
        go func(shutdowner GacefullyShutdowner) {
            shutdowner.Shutdown(left)
            c <- struct{}{}
        }(g)
        
        timer.Reset(left)
        select {
        case <-c:
            // 继续执行
        case <-timer.C:
            return errors.New("wait timeout")
        }
    }
    return nil
}
```



##### 管道模式

```go
func newNumGenerator(start, count int) <-chan int {
    c := make(chan int)
    go func() {
        for i := start; i < start+count; i++ {
            c <- i
        }
        close(c)
    }()
    return c
}

func filterOdd(in int) (int, bool) {
    if in%2 != 0 {
        return 0, false
    }
    return in, true
}

func square(in int) (int, bool) {
    return in * in, true
}

func spawn(f func(int) (int, bool), in <-chan int) <-chan int {
    out := make(chan int)
    
    go func() {
        for v := range in {
            r, ok := f(v)
            if ok {
                out <- r
            }
        }
        close(out)
    }()
    
    return out
}

func main() {
    in := newNumGenerator(1, 20)
    out := spawn(square, spawn(filerOdd, in))
    
    for v := range out {
        println(v)
    }
}
```

以上流水线用以生产“偶数的平方”，具有4个环节

第一个环节是生成最初的数据序列。这个环节由 ``` newNumGenerator ``` 创建goroutine负责生成，并发送到输出channel中，在序列全部发送完毕后，该goroutine关闭channel。

第二个环节是从序列中过滤奇数。由 ``` spawn ``` 函数创建的goroutine从第一个环节的输出中读取数据，并交由 ``` filterOdd ``` 函数进行处理。如果是奇数，则抛弃；如果是偶数，则将其发送到该goroutine的输出channel中，在全部数据发送完毕后，该goroutine关闭channel并退出

第三个环节是对序列进行平方处理。由 ``` spawn ``` 函数创建的goroutine从第二个环节输出的channel中读取数据，并交由 ``` square ``` 函数进行处理。处理后的数据被发到该goroutine的输出channel中，在全部数据发送完毕后，该goroutine关闭channel并退出

第四个环节是将序列中的数据输出到控制台。main goroutine从第三个环节的输出channel中读取数据，并将数据通过println输出到控制台上。在全部数据输出完毕后，main goroutine退出。

管道模式具有良好的可扩展性，如：

```go
in := newNumGenerator(1, 20)
out := spawn(square, spawn(filerOdd, spawn(filterNumOver100, in)))
```

对于管道模式，有两种扩展模式

###### 扇出模式

在某个处理环节，多个功能相同的goroutine从同一个channel中读取数据并处理，直到该channel关闭，这种情况被称为“扇出”。使用扇出模式可以在一组goroutine中均衡分配工作量，从而更均衡地利用CPU。

###### 扇入模式

在某个处理环节，处理程序面对不止一个输入channel。我们把所有输入channel的数据汇聚到一个统一的输入channel，然后处理程序再从这个channel中读取数据并处理，直到该channel因所有输入channel关闭而关闭。这种情况被称为“扇入”。

```go
func newNumGenerator(start, count int) <-chan int {
    c := make(chan int)
    go func() {
        for i := start; i < start+count; i++ {
            c <- i
        }
        close(c)
    }()
    return c
}

func filterOdd(in int) (int, bool) {
    if in%2 != 0 {
        return 0, false
    }
    return in, true
}

func square(in int) (int, bool) {
    return in * in, true
}

func spawnGroup(name string, num int, f func(int) (int, bool), in <-chan int) <-chan int {
    groupOut := make(chan int)
    var outSlice []chan int
    
    // 扇出模式
    for i := 0; i < num; i++ {
        out := make(chan int)
        go func(i int) {
            name := fmt.Sprintf("%s-%d:", name, i)
            fmt.Printf("%s begin to work...\n", name)
            
            for v := range in {
                r, ok := f(v) {
                    if ok {
                        out <- r
                    }
                }
            }
            close(out)
            fmt.Printf("%s work done\n", name)
        }(i)
        outSlice = append(outSlice, out)
    }
    
    // 扇入模式
    go func() {
        var wg sync.WaitGroup
        
        for _, out := range outSlice {
            wg.Add(1)
            go func(out <- chan int) {
                for V := range out {
                    groupOut <- v
                }
                wg.Done()
            }(out)
        }
        wg.Wait()
        close(groupOut)
    }()
    
    return groupOut
}

func main() {
    in := newNumGenerator(1, 20)
    out := spawnGroup("square", 2, square, spawnGroup("filterOdd", 3, filterOdd, in))
    
    // 为了输出更直观的结果,这里等待上面的goroutine都就绪
    time.Sleep(3 * time.Second)
    
    for v := range out {
        println(v)
    }
}
```



##### 超时与取消模式

我们常会使用Go编写向服务发起请求并获取应答结果的客户端应用。这里用来演示超时与取消模式再合适不过了。一个需求场景：编写一个从气象数据服务中心获取气象信息的客户端；该客户端每次会并发向三个气象数据服务中心发起数据查询请求，并以最快返回的那个响应信息作为此次请求的应答返回值。

下面是该需求的第一版实现：

```go
type result struct {
    value string
}

func first(servers ...*httptest.Server) (result, error) {
    c := make(chan result, len(servers))
    queryFunc := func(server *httptest.Server) {
        url := server.URL
        resp, err := http.Get(url)
        if err != nil {
            log.Printf("http get error: %s\n", err)
        }
        defer resp.Body.Close()
        body, _ := ioutil.ReadAll(resp.Body)
        c <- result {
            value: string(body),
        }
    }
    for _, serv := range servers {
        go queryFunc(serv)
    }
    return <-c, nil
}

func fakeWeatherServer(name string) *httptest.Server {
    return httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("%s receive a http request\n", name)
        time.Sleep(1 * time.Second)
        w.Write([]byte(name + ":ok"))
    }))
}

func main() {
    result, err := first(fakeWeatherServer("open-weather-1"), 
                         fakeWeatherServer("open-weather-2"),
                         fakeWeatherServer("open-weather-3"))
    if err != nil {
        log.Println("invoke first error:", err)
        return
    }
    
    log.Println(result)
}
```

以上代码使用 ``` httptest ``` 包的 ``` NewServer ``` 函数创建了三个模拟气象数据中心的服务器，然后将这三个气象数据服务中心的实例传入 ``` first ``` 函数。后者创建了三个goroutine，每个goroutine向一个气象数据中心发起查询请求。三个发起查询的goroutine都会将应答结果写入同一个channel中， ``` first ``` 获取第一个结果数据后就返回了

运行结果：

```shell
2023/03/15 17:02:23 open-weather-2 receive a http request
2023/03/15 17:02:23 open-weather-1 receive a http request
2023/03/15 17:02:23 open-weather-3 receive a http request
2023/03/15 17:02:24 {open-weather-2:ok}
```

上述例子运行在一种较为理想的情况下，而现实中的网络环境错综复杂，远程服务的状态也不甚明朗，很可能出现服务端长时间没有响应的情况。这时为了保证良好的用户体验，我们需要对可对客户端进行精细化控制，比如：只等待500ms，超过500ms仍然没有收到任何一个气象数据中心的响应， ``` first ``` 函数就会返回失败，以保证等待时间在人类的忍耐力承受范围之内。

我们对原 ``` first ``` 函数做出调整：

```go
func first(servers ...*httptest.Server) (result, error) {
	c := make(chan result, len(servers))
	queryFunc := func(server *httptest.Server) {
		url := server.URL
		resp, err := http.Get(url)
		if err != nil {
			log.Printf("http get error: %s\n", err)
		}
		defer resp.Body.Close()
		body, _ := ioutil.ReadAll(resp.Body)
		c <- result{
			value: string(body),
		}
	}
	for _, serv := range servers {
		go queryFunc(serv)
	}
	
	select {
	case r := <-c:
		return r, nil
	case <-time.After(500 * time.Millisecond):
		return result{}, errors.New("timeout")
	}
}
```

我们增加了一个定时器，并通过select原语监视该定时器事件和响应channel上的事件。如果响应channel上长时间没有数据返回，则当定时器事件触发时， ``` first ``` 函数返回超时错误：

``` shell
2023/03/15 17:12:53 open-weather-3 receive a http request
2023/03/15 17:12:53 open-weather-2 receive a http request
2023/03/15 17:12:53 open-weather-1 receive a http request
2023/03/15 17:12:53 invoke first error: timeout
```

加上**超时模式**的版本依然存在一个明显的问题，那就是即便 ``` first ``` 函数因超时而返回，三个已经创建的goroutine可能依然处在向气象数据中心请求或等待应答的状态，没有返回，也没有被回收，资源仍在占用，即使它们的存在已经没有任何意义。一种合理的解决思路是让这三个goroutine支持**取消操作**。在这种情况下，我们一般使用Go的 ``` context ``` 包来实现取消模式。 ``` context ``` 包是谷歌内部关于Go的一个最佳实践，Go在1.7版本将 ``` context ``` 包引入标准库中。下面是利用 ``` context ``` 包实现取消模式的代码：

```go
func first(servers ...*httptest.Server) (result, error) {
	c := make(chan result, len(servers))
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	queryFunc := func(i int, server *httptest.Server) {
		url := server.URL
		req, err := http.NewRequest("GET", url, nil)
		if err != nil {
			log.Printf("query goroutine-%d: http NewRequest error: %s\n", i, err)
		}
		req = req.WithContext(ctx)
		
		log.Printf("query goroutine-%d: send request...\n", i)
		resp, err := http.DefaultClient.Do(req)
		if err != nil {
			log.Printf("query goroutine-%d: get return error: %s\n", i, err)
		}
		log.Printf("query goroutine-%d: get response\n", i)
		defer resp.Body.Close()
		body, _ := ioutil.ReadAll(resp.Body)
		c <- result{
			value: string(body),
		}
		return
	}
	
	for i, serv := range servers {
		go queryFunc(i, serv)
	}

	select {
	case r := <-c:
		return r, nil
	case <-time.After(500 * time.Millisecond):
		return result{}, errors.New("timeout")
	}
}

func fakeWeatherServer(name string, interval int) *httptest.Server {
	return httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		log.Printf("%s receive a http request\n", name)
		time.Sleep(time.Duration(interval) * time.Millisecond)
		w.Write([]byte(name + ":ok"))
	}))
}

func main() {
	result, err := first(fakeWeatherServer("open-weather-1", 200),
		fakeWeatherServer("open-weather-2", 1000),
		fakeWeatherServer("open-weather-3", 600))
	if err != nil {
		log.Println("invoke first error:", err)
		return
	}

	fmt.Println(result)
	time.Sleep(10 * time.Second)
}
```

在这版实现中，我们利用 ``` context.WithCancel ``` 创建了一个可以被取消的 ``` context.Context ``` 变量，在每个发起查询请求的goroutine中，我们利用该变量更新了request中的 ``` ctx ``` 变量，使其支持被取消。这样在 ``` first ``` 函数中，无论成功得到某个查询goroutine的返回结果，还是超时失败返回，通过 ``` defer cancel() ``` 设定 ``` cancel ``` 函数在 ``` first ``` 函数返回前执行，那些尚未返回的在途查询的goroutine都将收到 ``` cancel ``` 时间并退出( ``` http ``` 包支持利用 ``` context.Context ``` 的超时和 ``` cancel ``` 机制)。以下是运行结果：

```shell
2023/03/15 17:37:29 query goroutine-2: send request...
2023/03/15 17:37:29 query goroutine-0: send request...
2023/03/15 17:37:29 query goroutine-1: send request...
2023/03/15 17:37:29 open-weather-2 receive a http request
2023/03/15 17:37:29 open-weather-3 receive a http request
2023/03/15 17:37:29 open-weather-1 receive a http request
2023/03/15 17:37:29 query goroutine-0: get response
{open-weather-1:ok}
2023/03/15 17:37:29 query goroutine-2: get return error: Get "http://127.0.0.1:52859": context canceled
2023/03/15 17:37:29 query goroutine-2: get response
2023/03/15 17:37:29 query goroutine-1: get return error: Get "http://127.0.0.1:52858": context canceled
2023/03/15 17:37:29 query goroutine-1: get response
```

可以看到，```  first ``` 函数在得到 ``` open-weather-1 ``` 这个气象数据服务中心的响应后，执行了 ``` cancel ``` 函数，其余两个 ``` http.DefaultClient.Do ``` 调用便**取消了请求**，返回了 ``` context canceled ``` 的错误，于是这两个goroutine得以退出。



### 网络编程



在Go语言的网络编程中，`net.Conn`接口用于表示网络连接，我们可以对其进行读写操作。Go标准库提供了多种方法和工具来处理`net.Conn`的读写操作。以下是一些常见的方式和使用示例：

#### 读操作

1. **直接使用`Read`方法**

   - 从连接中读取字节数据到给定的字节切片中。

   ```go
   func readUsingRead(conn net.Conn) ([]byte, error) {
       buf := make([]byte, 1024)
       n, err := conn.Read(buf)
       if err != nil {
           return nil, err
       }
       return buf[:n], nil
   }
   ```

2. **使用`bufio.Reader`**

   - 可以实现按行读取或者缓冲读取，提高读取效率。

   ```go
   func readUsingBufio(conn net.Conn) (string, error) {
       reader := bufio.NewReader(conn)
       line, err := reader.ReadString('\n')
       if err != nil {
           return "", err
       }
       return line, nil
   }
   ```

3. **使用`io.ReadFull`**

   - 确保读取到指定的字节数量，否则返回错误。

   ```go
   func readUsingReadFull(conn net.Conn) ([]byte, error) {
       buf := make([]byte, 256) // assume we know the exact number of bytes
       _, err := io.ReadFull(conn, buf)
       if err != nil {
           return nil, err
       }
       return buf, nil
   }
   ```

#### 写操作

1. **直接使用`Write`方法**

   - 将字节切片写入到连接中。

   ```go
   func writeUsingWrite(conn net.Conn, data []byte) error {
       _, err := conn.Write(data)
       return err
   }
   ```

2. **使用`bufio.Writer`**

   - 可以实现缓冲写入，减少对底层连接的写调用次数。

   ```go
   func writeUsingBufio(conn net.Conn, data string) error {
       writer := bufio.NewWriter(conn)
       _, err := writer.WriteString(data + "\n")
       if err != nil {
           return err
       }
       return writer.Flush() // 确保所有缓冲区数据被写出
   }
   ```

3. **使用`io.WriteString`**

   - 直接将字符串写入到连接中（实际上是一种便利函数）。

   ```go
   func writeUsingIoWriteString(conn net.Conn, data string) error {
       _, err := io.WriteString(conn, data)
       return err
   }
   ```

#### 读写注意事项

- **缓冲使用**：`bufio`中的缓冲读取和写入在性能上可能更优，尤其是在读取或写入频繁的小数据量时，但是需要注意`Flush`方法对`bufio.Writer`的使用。
- **错误处理**：每种方法在执行过程中都可能返回错误，生产环境中必须妥善处理这些错误。
- **同步性**：在`net.Conn`上进行并发的读写操作需要注意同步性问题，通常需要外部加锁或使用其他同步机制。

这些方法可以根据不同的使用场景灵活选择，从而高效地进行网络数据的传输。



## 标准库

### 字符串操作和字节操作官方库

对字符串相关的操作都在 ``` strings ``` 包中，其他基本类型转换成字符类型时，常使用 ```strconv ``` 包。

>  常用的函数

**Contains和ContainsAny**

```go
str1 := "ABCD"
str2 := "ABcd"
str3 := "ABCDabcd"
stand := "abcd"

fmt.Println(strings.Contains(stand, str1), strings.Contains(stand, str2))
fmt.Println(strings.ContainsAny(stand, str1), strings.ContainsAny(str1, stand))
fmt.Println(strings.ContainsAny(stand, str3), strings.Contains(stand, str3))
```

总结：当想匹配的字符串可能有不同的大小写时，使用**ContainsAny**，其中，将所有可能的组合都拼接上即可。其他情况下使用普通的**Contains**函数即可。



对字节数据进行的相关操作都在 ``` bytes ``` 包中。



### time包



### crypto包



### unsafe包



### reflect包



### signal包



### cgo



### context包



### 与IO有关的包



### net包

#### Socket编程



#### net/http包

##### net包



##### http包

创建http客户端

```go
/* 	创建transport，用以自定义请求
	以下代码的意思是，跳过TLS安全认证，当需要访问不安全的网站时需要被设置
*/
transport := &http.Transport{TLSClientConfig: &tls.Config{InsecureSkipVerify: true}}
// 将构造好的transport传入http客户端中初始化http客户端
client := http.Client{Transport: transport}
```

http客户端可以进行GET和POST请求



### Dockerfile



### 测试



### Go官方工具合集
