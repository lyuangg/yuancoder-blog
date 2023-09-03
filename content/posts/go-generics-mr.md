---
title: "Golang 的泛型能干什么"
date: 2023-09-03T10:03:44+08:00
draft: false
tags:
- go
- mr
- generics
categories:
- 开发
---

<!--more-->

Golang在1.18版本支持了泛型, 在工作中还没有真正用过， 这几天研究了一下，脑子里最先想到的使用场景就是集合类型的处理。 

常用的集合处理函数 `map`, `redude`  等， 在别的语言中都是直接支持的，例如 PHP 中的 `array_map`, `array_reduce`。

但是 golang 之中只能使用 for 循环，现在有了泛型，让实现 `map`, `reduce` 变的简单了， 相比使用 `interface`，性能也更好。

**代码实现：**

```go
func Map[T any, R any](ts []T, f func(T) R) []R {
	nt := make([]R, 0, len(ts))
	for _, t := range ts {
		nt = append(nt, f(t))
	}
	return nt
}

func Reduce[T any](ts []T, f func(T, T) T, init T) T {
	t := init
	for _, tt := range ts {
		t = f(t, tt)
	}
	return t
}
```

**使用**

```go
type Person struct {
    ID   int
    Name string
}

personList := []Person{
    { 1,"zhangsan"},
    { 2,"lisi"},
}

names := Map(personList, func(p Person) string { return p.Name })
fmt.Println(names) // [zhangsan lisi]

nums := []int{1,2,3,4}
total := Reduce(nums, func(a, b int) int {
    return a+b
}, 0)
fmt.Println(total) // 10
```

除了 `map`, `reduce` 还有其他的函数也可以使用泛型实现。  

我写了一个这样的工具包： [https://github.com/lyuangg/mr](https://github.com/lyuangg/mr)

实现了：`map`, `reduce`, `Contains`, `ToMap`, `Filter`, `Diff`, `Intersect`, `Unique`, `Merge` 等函数。 