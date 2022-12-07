---
title: "设计一个日志库-Ylog"
date: 2022-12-05T16:38:38+08:00
draft: false
tags:
- go
- log
- ylog
categories:
- 开源
---

<!--more-->

## 起因

日志库是最基础和最重要的库，go 官方的日志库功能太简陋，项目中经常使用的功能：日志级别，json 格式，按日期旋转文件等都不支持。 

而使用最多的 `logrus` 库也不支持按时间旋转文件，还要依赖另外三方包去实现。 
 
所以我决定重新设计一个简单易用的日志库 [ylog](https://github.com/lyuangg/ylog).

## 目标

一个日志库要保证最常用的功能和一定的灵活性， 所以实现的目标如下：

1. 支持日志级别
2. 支持 JSON 格式
3. 支持按日期旋转文件
4. 自定义格式化和输出

## 设计

#### 1. 日志记录

日志写入一条信息就是一条日志记录，日志记录应该是一个最基础和核心的结构体。

```go
type Record struct {
    Time   time.Time // 时间
    Msg    string // 原始消息
    Fmsg   string // 格式化后的消息内容
    Level  Level // 级别
    Fields Fields // 自定义字段
}
```

#### 2. 日志级别

日志级别类型 Level 应该是一个 int 类型，这样才能对级别进行对比和排序, 定义如下:

```go
type Level uint32

const (
	DebugLevel Level = iota
	InfoLevel
	WarnLevel
	ErrorLevel
)
```

> 为什么这里的 `Level` 使用的是 `uint32` 的类型，而不是 `uint8` 或者 `uint16` 或者其他, 这是因为 `atomic` 包对应的函数都是 32 或者 64 位的, 为了能够使用到原子操作相关的函数, 所以这里使用 `uint32`。

#### 3. 自定义字段

自定义字段本质就是 `key-value`, 使用 map 即可: 

```go
type Fields map[string]interface{}
```

#### 4. 接口

Logger 接口作为对外使用的接口，提供基本使用方法和设置。

```go
type Logger interface {
    // 写日志
    Info(msg string)
    Debug(msg string)
    Warn(msg string)
    Error(msg string)

    // 设置格式化
    SetFormatter(f Formatter)
    // 设置输出
    SetOuter(o Outer)
    // 设置级别
    SetLevel(l Level)

    // 设置自定义字段
    With(fields Fields) Logger
}
```

> With 方法，设置自定义字段后返回一个新的 Logger。 
> 自定义字段存储在 Logger 中， 在写日志的时候，新的 Logger 会复制自定义字段到日志记录(Record)中。  
> 由于自定义字段 Fields 本质是一个 map ，map 不是线程安全的，所以这里只对 map 进行复制操作。  
> 而同样的 Logger 还可以设置公共的字段，而不影响上层的 Logger。
 
   
Formatter 和 Outer 接口都比较简单，Formatter 格式化日志记录，Outer 负责输出日志。

```go
type (
	Formatter interface {
		Format(r Record) (string, error)
	}
    Outer interface {
		Write(r Record) error
	}
)
```

## 实现

接口定义完之后，实现对应的接口方法, 这里需要考虑线程安全，在该枷锁的地方要加锁。 看一下 Logger 接口的核心代码: 

```go
Log struct {
    f      Formatter
    o      Outer
    level  Level
    fields Fields
    mu     sync.RWMutex
}

func (l *Log) SetOuter(o Outer) {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.o = o
}
func (l *Log) GetOuter() Outer {
	l.mu.RLock()
	defer l.mu.RUnlock()
	return l.o
}
func (l *Log) SetLevel(level Level) {
	atomic.StoreUint32((*uint32)(&l.level), uint32(level))
}
func (l *Log) GetLevel() Level {
	return Level(atomic.LoadUint32((*uint32)(&l.level)))
}

func (l *Log) Info(msg string) {
	l.Write(msg, InfoLevel)
}

func (l *Log) Write(msg string, level Level) error {

	if level >= l.GetLevel() {
		record := Record{
			Time:   time.Now(),
			Msg:    msg,
			Level:  level,
			Fields: l.GetFields(),
		}

		// 格式化
		fmsg, err := l.GetFormatter().Format(record)
		if err != nil {
			return fmt.Errorf("format record err: %w", err)
		}
		record.Fmsg = fmsg

		// 输出
		return l.GetOuter().Write(record)
	}
	return nil
}

...

```

封装一些函数方便使用

```go
var defaultLog Logger

func init() {
	defaultLog = New()
	defaultLog.SetLevel(InfoLevel)
}
func Info(msg string) {
	defaultLog.Info(msg)
}
func Infof(format string, a ...interface{}) {
	defaultLog.Info(fmt.Sprintf(format, a...))
}
```

----

其他代码具体实现查看项目: [https://github.com/lyuangg/ylog](https://github.com/lyuangg/ylog)

## 性能测试

与 logrus 性能对比:

```text
goos: darwin
goarch: amd64
pkg: github.com/lyuangg/ylog/benchmarks
cpu: Intel(R) Core(TM) i5-8279U CPU @ 2.40GHz
BenchmarkDefaultLogger/logrus-8                  1528854              2652 ns/op             576 B/op         17 allocs/op
BenchmarkDefaultLogger/ylog-8                   11159875               307.4 ns/op           276 B/op          6 allocs/op
BenchmarkRoateLogger/logrus-8                     250870             13076 ns/op             945 B/op         21 allocs/op
BenchmarkRoateLogger/ylog-8                       622275              5385 ns/op             292 B/op          8 allocs/op
BenchmarkWithField/logrus-8                      1000000              3094 ns/op             881 B/op         18 allocs/op
BenchmarkWithField/ylog-8                        8655415               387.6 ns/op           500 B/op          7 allocs/op
BenchmarkJsonFormat/logrus-8                      879865              3747 ns/op            1682 B/op         29 allocs/op
BenchmarkJsonFormat/ylog-8                       3995971               999.0 ns/op          1585 B/op         25 allocs/op
PASS
ok      github.com/lyuangg/ylog/benchmarks      32.649s
```
## 项目

github: [https://github.com/lyuangg/ylog](https://github.com/lyuangg/ylog)