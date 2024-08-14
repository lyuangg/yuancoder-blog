---
title: "Go 1.23 的 range over func 自定义迭代器"
date: 2024-08-14T14:27:22+08:00
draft: false
tags:
- go
- golang
- range over func
- iterator
categories:
- 开发
---

<!--more-->


Go 1.23 增加了一个语法特性, 就是 "[range over func 试验特性](https://go.dev/wiki/RangefuncExperiment)", 这个特性在 Go 1.22 中就已经存在，只是在这个版本转正了。

“range over func”，就是在 `for-range` 循环中可以直接迭代函数。

这个特性带来以下好处：

- 提供了一种统一高效的迭代方式。
- 可以通过迭代器的方式提高性能。
- 可以为函数式编程风格提供标准迭代机制。

### 使用

例子： 

```go
var fn = func(yield func(k int, v byte) bool) {
	for i := 0; i < 26; i++ {
		if !yield(i, byte('a'+i)) {
			return
		}
	}
}
for k, v := range fn {
	fmt.Printf("%d: %c\n", k, v)
}
```

输出:

```
0: a
1: b
2: c
3: d
4: e
5: f
6: g
...
25: z
```

> 注意： yield 不是关键字, 只是一个参数名， 只是为了模仿其他语言，使用了一个这样的名字。

fn 可以是以下几种格式：

```
for x, y := range fn { ... }
for x, _ := range fn { ... }
for _, y := range fn { ... }
for x := range fn { ... }
for range fn { ... }
```

实现了这几种格式的函数类型都可以使用 `for-range` 迭代。

### 原理


`rang over func`机制的实现是通过编译器在源码层面的转换，其转换形式大致如下：


```go
var fn = func(yield func(k int, v byte) bool) {
	for i := 0; i < 26; i++ {
		if !yield(i, byte('a'+i)) {
			return
		}
	}
}
for k, v := range fn {
	fmt.Printf("%d: %c\n", k, v)
}
```

转换为： 


```go
var fn = func(yield func(k int, v byte) bool) {
	for i := 0; i < 26; i++ {
		if !yield(i, byte('a'+i)) {
			return
		}
	}
}
fn(func(k int, v byte) bool {
	fmt.Printf("%d: %c\n", k, v)
	return true
})
```

其实就是把 `range` 结构 body 里面的内容生成了 `yield` 函数,  如果 `range` 结构里面没有 `return` ，默认就是 `return true`。

break 语句会转换为 `return false`。
 
### 总结

 - 只要实现了相关函数就可以实现自己的迭代器。
 - 现实原理是通过编译器在源码层面的转换。
 - 使用起来很简单。
