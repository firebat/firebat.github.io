+++
date = '2022-08-22T00:00:00+08:00'
title = 'Go简明教程'
tags = ["Go"]
+++
本教程为《[Go语言圣经](https://gopl-zh.github.io/)》内容简化，作为开发同学快速上手用。

# 语言基础
- 没有异常、宏
- 常量编译时计算
- 包、变量 声明后必须使用，否则报错
- 使用`_`丢弃数据，防止编译报错（变量、包）
- 包初始化默认执行`init()`
- 首字母大写可外部访问，否则仅内部访问
- `switch`默认`break`，相邻`case`执行需加`fallthrough` (用得少）

# 关键字
25个，仅用于语法结构
```
break    default        func      interface   select
case     defer          go        map         struct
chan     else           goto      package     switch
const    fallthrough    if        range       type
continue for            import    return      var
```

# 预定义名
30多个，用于常量、类型、函数
```
内建常量 true false iota nil
内建类型 int int8 int16 int32 int64
        uint uint8 uint16 uint32 uint64 uintptr
        float32 float64 complex128 complex64
        bool byte rune string error
内建函数 make len cap new append copy close delete
        complex real(实部) imag(虚部)
        panic recover
```
# 包
```go
package 包名

import (
  [别名] 包名
  _ 包名        // 触发init() 如: 注册驱动
)
```
# 常量
```go
const 常量名 常量类型 = 表达式
const PI int = 3.14159
const (
  // 批量定义
)
```
# 变量
```go
var 变量名 类型 = 表达式
var name string = "alice"
var name = "alice"     // 类型自动识别
name := "alice"        // 函数内简写

var (
  // 批量定义
)
```
# 数组切片哈希
- 数组，定长
- 切片，变长，通过data、len、cap 定义一个数据空间
- 哈希，无序
```go
var 数组 [长度]类型 
array := [2]int{1,2}
array := [...]int{1,2,3}  // ... 自适应长度
len(array)                // 长度
for i, value := range array {
}

var 切片 []类型
slice := make([]T, len)
slice := make([]T, len, cap)
slice := []string{
  "Red",
  "Green",
  "Blue"
}
len(slice)            // 长度
cap(slice)            // 容量
append(slice, value)  // 自动扩容
for i, value := range slice {
}

var 哈希 map[键类型]值类型
map := make(map[string]int)
map := map[string]int{
  "Red": 0,
  "Green": 1,
  "Blue": 2
}
delete(map, key)
for key, value := range map {    // 迭代遍历
}
```
# 定义类型
```go
type 类型名 底层类型
type Celsius float6

type Point struct {
  X int
  Y int
}

type Circle struct {
  Point                  // 匿名嵌入，类似继承
  Radius int
}

type Node struct {
  Value int,
  Next  *Node            // 同类型指针
}
```
# make和new
- make 初始化内置类型(仅`slice/hash/channel`) ，返回引用
- new 根据类型分配内存，返回指针
- 若分不清，可统一用 `Type{}` 和 `&Type{}` 方式
```go
func make(t Type, size ...IntegerType) Type

func new(Type) *Type

默认空值 "" 0 nil false

&变量    # 取地址
*指针    # 取变量
```
# 类型转换
```go
// 显示转换
var 目标变量 = 目标类型(源变量)
s = :int64(x)

// 断言转换
var 目标变量 = 源变量.(目标类型)
s, ok := x.(T)

// 识别转换
switch 目标变量 := 源变量.(type) {
  case nil:                 // nil
  case bool:                // 数据类型
  case func(int) float64:   // 函数类型
  default:                  // 未知类型
}

// 强制转换
var 目标变量 = *(*目标类型)(unsafe.Pointer(&源变量))
```
# 函数
- 一等公民，可闭包、可作为参数、返回值传递
- 支持多返回值
- 没有重载和默认值
```go
func 函数名(参数列表) (返回列表) {
}

var Upper(s string)        // 函数声明
Upper = func(s string) {   // 函数赋值
}

func () {                  // 立即执行匿名函数
}()

func sum(vals ...int) int   // 可变参数
```
# Error
- 通常作为函数最后一个返回值
- 与异常思路略有不同，异常默认返回，需处理则catch；Error默认继续，需处理则if
```go
func MustDoSomething() (any, error) {
    ...
    return nil, errors.New("Error happened")
}

# 自定义错误 带编码
type CodeError struct {
    Code    int `json:"code"`
    Message string `json:"message"`
}

func (c *CodeError) Error() string { // 含有Error()方法的结构均可作为error使用
    return c.Message
}
```
# Context
- - 调用链上下文管理，通常作为函数第一个参数
```go
ctx :=context.Background()
ctx = context.WithValue(parent, key, value) // 装饰器模式，内部是个链，性能不好

func GetUser(ctx context.Context, name string)(UserInfo, error) {
    value := ctx.Value(key) 
}
```
# Panic
- 严重错误、无法恢复时使用，否则使用error机制 `if err != nil`
- 立即中断程序运行，执行对应的延时函数（defer机制）
```go
func MustCompile(expr string) {
  re, err := Compile(expr)
  if err != nil {
    panic(err)
  }
  return re
}
```

# Defer
- 延时处理函数，函数执行后、内存释放前执行
- 可以写多个，先入后出
- `os.Exit`、`log.Fatal` 会导致`Defer`函数不执行，慎用
```go
func ReadFile(filename string) {
  f, err := os.Open(filename)
  if err != nil {
    return nil, err
  }
  defer f.Close()
  return ReadAll(f)
}
```

# Recover
- 试图恢复panic
```go
defer func() {
  switch p := recover(); p {
    case nil:         # no panic
    case "x":         # expected panic
    default:          # keep going
        panic(p)
    }
  }
}()

panic("x")
```
# 方法 Method
- 绑定类型与函数，面向对象编程
- 接收器通常用指针（引用传递），否则为副本（值传递）
- 接收器变量名为类型首字母小写
```go
func (p *Point) ScaleBy(factor float64) {
  p.X *= factor
  p.Y *= factor
}

p := &Point{1,2}
p.ScaleBy(2)
```

# 接口 Interface
- 以行为定类型，面向行为编程
- 任意类型`interface {}` 1.18版本后内建`type any interface{}`
```go
package io

type Reader interface {
  Read(p []byte)(n int, e error)
}
type Writer interface {
  Write(p []byte) (n int, err error)
}
type Closer interface {
  Close() error
}

type ReadWriter interface {
  Reader                    // 内嵌继承
  Writer
}

type ByteCounter int
func (b *ByteCounter) Write(p []byte)(int error) {  // 满足约定即可使用
  ...
}
```

# 范型
- 同类数据，逻辑复用
```go
type RestDTO [T any] struct {
    Code    int     `json:"code"`
    Message string  `json:"message"`
    Data    T       `json:"data"`
}

type IdProvider [T int | string] interface {
    Next() T
}
```
# 并发
- `goroutine` 协程并发执行，没有唯一标识（即无法实现ThreadLocal，需要Context机制）
- `channel` 协程间通信
```go
done := make(chan string, [缓冲大小])

go func() {            // 并发执行
  time.Sleep(3000)
  done <- "done"       // 发送信息
}()

<-done                 // 阻塞等待
close(done)
```

# 多路复用
```go
var done = make(chan struct{})

func cancelled() bool {
  select {                // Select 多路复用
  case <-done:
    return true
  default:
    return false
  }
}
```
# 懒处理
- 仅执行一次，单例模式常用
```go
var (
    once       sync.Once
    instance   *Type
)

func GetInstance() *Type {
    once.Do(func() {
        instance = &Type{}
    })

    return instance
}
```
# 互斥锁
```
mu := sync.Mutex              // 不可重入

func() {
  mu.Lock()                   // 临界区
  defer mu.Unlock()
  ...
}
```

# 读写锁
```go
mu := sync.RWMutex

mu.RLock()
defer mu.RUnlock()
```
# 反射
```go
import reflect

t := reflect.TypeOf(i) // 类型元信息
v := reflect.ValueOf(i) // 实际数值

tag := t.Elem().Field(0).Tag         // 获取标签 如`json:name`
val := v.Elem().Field(0).String()    // 获取字段值
```
# 测试
- 与源文件同包，以`_test.go`结尾
- 功能测试以Test开头
- 基准测试以Benchmark开头，默认进行功能测试，可通过`-run=NONE`禁止
```sh
$ go test [package]
  -v                 输出测试函数与时间
  -run=""            测试名（正则匹配）

  -bench=测试名（正则匹配）
  -benchmem 统计内存分配
  
  -coverprofile=cover.out   覆盖率分析
  -cpuprofile=cpu.out
  -blockprofile=block.out
  -memprofile=mem.out

$ go tool cover -html=覆盖率文件
```
```go
import  "testing"

// 功能测试
func TestName(t *testing.T) {
  tests := []struct {        // Case表
    input 输入类型
    want 结果类型
  }{
    {case1, want1},
    ...
  }
  
  for _, test := range tests {
    got := xxx(input)
    if got != test.want {
      t.Errorf(".....")
    }
  }
}

// 性能测试
func BenchmarkName(b *testing.B) {
  for i:=0; i<b.N; i++ {
    //
  }
}
```
