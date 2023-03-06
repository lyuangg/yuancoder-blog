---
title: "Go 中容易出错的知识点总结"
date: 2023-03-05T19:44:54+08:00
draft: false
tags:
- go
categories:
- 开发
---

<!--more-->

Go 学习起来虽然比较简单，但是如果对 Go 底层的知识不了解，很容易踩坑。  

总结了一下容易踩坑的地方:

### slice

slice 切片本质上就是一个结构体，切片的操作实际都是对这个结构体的操作。

切片的结构体如下： 

```go
type slice struct { 
	array unsafe.Pointer // 指向底层数组的指针
	len int          // 当前切片的长度
	cap int         // 当前切片的容量
}
```

**内存回收问题** 

对一个切片进行切片操作，只是修改了 slice
的结构体,不会创建新的底层数组, 所以内存会一直占用 。

如果想要释放，可以参考 go 源码中连接池的代码 (go/src/database/sql/sql.go): 

```go
// conn returns a newly-opened or cached *driverConn.
func (db *DB) conn(ctx context.Context, strategy connReuseStrategy) (*driverConn, error) {
    ...
    // Prefer a free connection, if possible.
	numFree := len(db.freeConn)
	if strategy == cachedOrNewConn && numFree > 0 {
		conn := db.freeConn[0]
		copy(db.freeConn, db.freeConn[1:])
		db.freeConn = db.freeConn[:numFree-1]
		conn.inUse = true
        ...
	}
    ...
}
```

这里使用 copy 方法覆盖了需要删掉的原有数组的值。

**线程安全**

slice 不是线程安全的，不支持并发读写，如果需要并发读写，需要加锁。

### for...range

**变量复用问题**

经常看到代码这么写:

```go
a := []int{1, 2, 3, 4, 5}
for _, a1 := range a {
    go func() {
        time.Sleep(time.Second)
        fmt.Println(a1)
    }()
}
time.Sleep(time.Second * 3)
```

期望输出是： 1，2，3，4，5  
实际输出是:  5, 5, 5, 5, 5

> 原因是 a1 这个变量只会在循环开始的时候声明一次。后面的每次迭代都是对 a1
这个变量进行赋值操作。  
> 循环中的闭包引用的是 a1 变量, a1 变量的值最后都被修改成了 5 。 

如果要解决这个问题可以这么写： 

```go
a := []int{1, 2, 3, 4, 5}
for _, a1 := range a {
    go func(b int) {
        time.Sleep(time.Second)
        fmt.Println(b)
    }(a1)
}
time.Sleep(time.Second * 3)
```

**range 副本**

看代码： 

```go
a := [5]int{1, 2, 3, 4, 5}
b := [5]int{}

for i, v := range a {
    if i == 0 {
        a[1] = 22
        a[2] = 33
    }
    b[i] = v
}

fmt.Println(a)
fmt.Println(b)
```

期望输出: 

```
[1 22 33 4 5]
[1 22 33 4 5]
```

实际输出: 

```
[1 22 33 4 5]
[1 2 3 4 5]
```
> range 循环变量 a 是一个副本, 每次迭代的值 v 都是副本的值，虽然修改了 a
本身，但是 v 的值并没有被修改。

可以使用切片:

```go
a := []int{1, 2, 3, 4, 5}
b := make([]int, 5)
for i, v := range a {
    if i == 0 {
        a[1] = 22
        a[2] = 33
    }
    b[i] = v
}
fmt.Println(a)
fmt.Println(b)
```

输出: 

```
[1 22 33 4 5]
[1 22 33 4 5]
```

**map**

map 使用 range 时，也会得到一个副本，map 的本质是一个 hmap
的结构体，所以在迭代时对 map 的操作也会影响到当前循环的副本。  

由于 map 的迭代顺序每次都不一样，所以在循环中对map 的修改，对后续迭代的影响也是不确定的。


### map

- map 是非线程安全的，所以 map 的操作需要加锁。
- 如果 map 的操作是读多写少的情况, 可以使用 `sync.map`。 `sync.map` 是线程安全的。

### 接口

接口类型变量在运行时表示为eface和iface: 

```go
type iface struct {
    tab  *itab
    data unsafe.Pointer
}

type eface struct {
    _type *_type
    data  unsafe.Pointer
}
```
- eface用于表示空接口类型变量，iface用于表示非空接口类型变量；
- 这两种结构的共同点是都有两个指针字段，第二个指针字段都指向当前赋值给该接口类型变量的动态类型变量的值。
- 不同点在于eface所表示的空接口类型并无方法列表
- iface的第一个字段指向一个itab类型结构,存储接口本身的信息以及所实现的方法的信息
- 当两个接口类型变量的类型信息相同，且数据指针所指数据相同时，两个接口类型才是相等的

例子： 

```go
type (
	MyError struct {
		error
	}
)

func retErr() error {
	var e *MyError = nil
	return e
}

func main() {
	e := retErr()
	if e != nil {
		fmt.Println("error:", e)
	} else {
		fmt.Println("ok")
	}
}
```

期望输出是： ok
实际输出是： error: nil

> 根据结果输出，代码走了 e != nil 分支. 
> 这里的 e 是一个 eface 类型 _type 指向了 *MyError 类型信息，data 指向了 nil。
> 所以 e 本身不为空，但是值 data 为 nil。

例子2: 

```go
var i interface{}
var e error
fmt.Println("i = e:", i == e)
```

output:

```
i = e: true
```

> 这里 i 为 eface， e 为 iface, 虽然结构不一样，但是他们内部两个字段都指向了 nil
> 因此上面的变量判断为 true


### 方法

- 方法的本质可以看成是以方法所绑定类型实例为第一个参数的普通函数。
- 在调用类型方法时，编译器会为我们自动转换对应的类型。

```go
func Get(t T) int {
    return t.a
}

func Set(t *T, a int) int {
    t.a = a
    return t.a
}

var t T
t.Get()
t.Set(1)

// 可以等价转换为

var t T
T.Get(t)
(*T).Set(&t, 1)
``` 

**一个例子** 

```go
type (
	Student struct {
		age int
	}
)

func (s *Student) printAge() {
	fmt.Println(s.age)
}

func main() {

	s1 := []*Student{{1}, {2}, {3}}
	fmt.Println("s1:")

	for _, v := range s1 {
		go v.printAge()
	}

	time.Sleep(time.Second)

	s2 := []Student{{1}, {2}, {3}}
	fmt.Println("s2:")
	for _, v := range s2 {
		go v.printAge()
	}

	time.Sleep(time.Second)
}
```

期望输出: 
```
s1:
3
2
1
s2:
3
2
1
```

实际输出: 
```
s1:
3
2
1
s2:
3
3
3
```

为什么 s2 打印的都是 3 ? 我们按照方法的本质转换一下代码： 

```go
// s2
s1 := []*Student{{1}, {2}, {3}}
fmt.Println("s1:")

for _, v := range s1 {
    go (*Student).printAge(v)
}

time.Sleep(time.Second)

s2 := []Student{{1}, {2}, {3}}
fmt.Println("s2:")
for _, v := range s2 {
    go (*Student).printAge(&v)
}
```

> 根据转换后的代码，range s2 的循环中，printAge 传递的是 变量 v
的地址，所以最后打印的结果是一样的。

> 解决方法是可以修改方法 printAge 的接受者类型为: `s Student`

### defer

- defer在 return 之后执行，但在函数退出之前，defer可以修改**带命名的返回值**。

```go
func main() {
    fmt.Println(test1())
}
func test1() (result int) {
	defer func() {
		result = 22
	}()
	result = 100
	return
}
```

输出: 22

- defer 会在 panic 之前执行

```go
func test1() int {
	defer func() {
		fmt.Println("defer run...")
	}()

	panic("panic")
	return 0
}
```

output: 

```
defer run...
panic: panic

goroutine 1 [running]:
main.test1()
```

- 如果在函数里执行了os.Exit而退出, defer 不会执行。

- 被defer的函数或方法的参数的值在执行到defer语句的时候就被确定下来了。

```go
func main() {
    fmt.Println(test1())
}
func test1() (a int) {
	defer func(a int) {
		fmt.Println("defer: ", a)
	}(a)
	a = 100
	return a
}
```

output: 

```
defer:  0
100
```
- defer 不可以捕获子goroutine 的 panic
