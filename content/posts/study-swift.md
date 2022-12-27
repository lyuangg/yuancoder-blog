---
title: "swift 语法学习"
date: 2022-12-27T20:31:22+08:00
draft: false
tags:
- swift
- ios
categories:
- 开发
---

<!--more-->

```swift
let explicitDouble: Double = 70 // 常量
var myVariable = 42 // 变量
let widthLabel = label + String(22) // 类型转换
let appleSummary = "I have \(apples) apples."

// 多行
let quotation = """
I said "I have \(apples) apples."
And then I said "I have \(apples + oranges) pieces of fruit."
"""

// 数组
var shoppingList = ["catfish", "water", "tulips", "blue paint"]
let emptyArray: [String] = []

// 字典
var occupations = [
    "Malcolm": "Captain",
    "Kaylee": "Mechanic",
]
let emptyDictionary: [String: Float] = [:]

// 集合
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
favoriteGenres.insert("Jazz")


// if， for
for score in individualScores {
    if score > 50 {
        teamScore += 3
    } else {
        teamScore += 1
    }
    print(score)
}
for i in 0..<4 {
    total += i
}

// 可选(Optionals)类型, 默认值为 nil
var optionalString: String? = "Hello"
print(optionalString!)
var optionalString: String? = nil
print(optionalString ?? "hello")

// switch, 不需要 break
let vegetable = "red pepper"
switch vegetable {
case "celery":
    print("Add some raisins and make ants on a log.")
case "cucumber", "watercress":
    print("That would make a good tea sandwich.")
case let x where x.hasSuffix("pepper"):                   // where 语句可以用来设置约束条件、限制类型
    print("Is it a spicy \(x)?")
default:
    print("Everything tastes good in soup.")
}


// while
while n < 100 {
    n *= 2
}

repeat {
    m *= 2
} while m < 100


// 函数
func greet(person: String, day: String) -> String {
    return "Hello \(person), today is \(day)."
}
greet(person:"Bob", day: "Tuesday")

func greet(_ person: String, on day: String) -> String {
    return "Hello \(person), today is \(day)."
}
greet("John", on: "Wednesday")

// 返回函数
func makeIncrementer() -> ((Int) -> Int) {
    func addOne(number: Int) -> Int {
        return 1 + number
    }
    return addOne
}

// 使用 {} 来创建一个匿名闭包，使用 in 将参数和返回值类型的声明与闭包函数体进行分离。
// 无返回类型使用 () -> Void
// 闭包是引用类型
numbers.map({
    (number: Int) -> Int in
    let result = 3 * number
    return result
})

//闭包2： 如果一个闭包的类型已知，比如作为一个代理的回调，你可以忽略参数，返回值，甚至两个都忽略。
let mappedNumbers = numbers.map({ number in 3 * number })

//闭包3: 可以通过参数位置而不是参数名字来引用参数， 当一个闭包作为最后一个参数传给一个函数的时候，它可以直接跟在圆括号后面。当一个闭包是传给函数的唯一参数，你可以完全忽略圆括号。
let reversedNames = names.sorted(by: { $0 > $1 } )
let sortedNumbers = numbers.sorted { $0 > $1 }

// 逃逸闭包
// 当一个闭包作为参数传到一个函数中，但是这个闭包在函数返回之后才被执行，我们称该闭包从函数中逃逸
// 在参数名之前标注 @escaping，用来指明这个闭包是允许“逃逸”出这个函数的
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}


// 类
class Square: NamedShape {
    var sideLength: Double

    // getter 和 setter 的计算属性。
    // 只有 getter 没有 setter 的计算属性叫只读计算属性。可以省略 get 关键词
    var perimeter: Double {
        get {
            return 3.0 * sideLength
        }
        set {
            sideLength = newValue / 3.0
        }
    }

    // willSet 和 didSet
    // 写入的代码会在属性值发生改变时调用，但不包含构造器中发生值改变的情况。
    var s: String {
        willSet {
            print("willset")
        }
        didSet {
            print("didSet")
        }
    }

    // 构造器
    init(sideLength: Double, name: String) {
        self.sideLength = sideLength
        super.init(name: name)
        numberOfSides = 4
    }

    // 析构器
    deinit {
        print("deinit")
    }

    func area() ->  Double {
        return sideLength * sideLength
    }

    // 重写父类方法
    override func simpleDescription() -> String {
        return "A square with sides of length \(sideLength)."
    }
}
let test = Square(sideLength: 5.2, name: "my test square")
test.area()


// 枚举
// 枚举可以包含方法。
// 使用 init?(rawValue:) 初始化构造器来从原始值创建一个枚举实例。
// rawValue 访问原始值
enum Rank: Int {
    case ace = 1
    case two, three, four, five, six, seven, eight, nine, ten
    case jack, queen, king
    func simpleDescription() -> String {
        switch self {
        case .ace:
            return "ace"
        case .jack:
            return "jack"
        case .queen:
            return "queen"
        case .king:
            return "king"
        default:
            return String(self.rawValue)
        }
    }
}
let ace = Rank.ace
let aceRawValue = ace.rawValue
// 枚举关联值
// 枚举来存储任意类型的关联值
// 在枚举成员前加上 indirect 来表示该成员可递归
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}
var productBarcode = Barcode.upc(8, 85909, 51226, 3)
productBarcode = .qrCode("ABCDEFGHIJKLMNOP")

// 可以在 switch 的 case 分支代码中提取每个关联值作为一个常量（用 let 前缀）或者作为一个变量（用 var 前缀）来使用
switch productBarcode {
case let .upc(numberSystem, manufacturer, product, check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
case .qrCode(let productCode):
    print("QR code: \(productCode).")
}


// 结构体
// 结构体和类最大的一个区别就是结构体是传值，类是传引用。
// 所有结构体都有一个自动生成的成员逐一构造器，用于初始化新结构体实例中成员的属性
struct Resolution {
    var width = 0
    var height = 0
}
let someResolution = Resolution(width: 10, height: 20)

// 协议
// 类、枚举和结构体都可以遵循协议
// mutating 关键字用来标记一个会修改结构体的方法, 类中不需要这个关键字
protocol ExampleProtocol {
    var simpleDescription: String { get }
    mutating func adjust()
}
// 可选的协议
// 使用 optional 关键字作为前缀来定义可选要求, 遵循协议的类型可以选择是否实现这些要求
// 可选要求用在你需要和 Objective-C 打交道的代码中。协议和可选要求都必须带上 @objc 属性。
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}

// 使用 extension 来为现有的类型添加功能
extension Int: ExampleProtocol {
    var simpleDescription: String {
        return "The number \(self)"
    }
    mutating func adjust() {
        self += 42
    }
}
print(7.simpleDescription)

// 属性包装器
// 属性包装器在管理属性如何存储和定义属性的代码之间添加了一个分隔层。
// 1. 定义一个属性包装器，你需要创建一个定义 wrappedValue 属性的结构体、枚举或者类。
// 2. 通过在属性之前写上包装器名称作为特性的方式，你可以把一个包装器应用到一个属性上去。
// 当你把一个包装器应用到一个属性上时，编译器将合成提供包装器存储空间和通过包装器访问属性的代码。
@propertyWrapper
struct TwelveOrLess {
    private var number = 0
    var wrappedValue: Int {
        get { return number }
        set { number = min(newValue, 12) }
    }
}
struct SmallRectangle {
    @TwelveOrLess var height: Int
    @TwelveOrLess var width: Int
}

// 下标
// 下标可以定义在类、结构体和枚举中，是访问集合、列表或序列中元素的快捷方式。
// 定义下标使用 subscript 关键字，与定义实例方法类似
struct TimesTable {
    let multiplier: Int
    subscript(index: Int) -> Int {
        return multiplier * index
    }
}
let threeTimesTable = TimesTable(multiplier: 3)
print("six times three is \(threeTimesTable[6])")

// 错误处理
// 采用 Error 协议的类型来表示错误
// 使用 throw 来抛出一个错误和使用 throws 来表示一个可以抛出错误的函数。
// 使用 do-catch 进行错误处理,  try 来标记可以抛出错误的代码。在 catch 代码块中，除非你另外命名，否则错误会自动命名为 error 。
enum PrinterError: Error {
    case outOfPaper
    case noToner
}
func send(job: Int, toPrinter printerName: String) throws -> String {
    if printerName == "Never Has Toner" {
        throw PrinterError.noToner
    }
    return "Job sent"
}
do {
    let printerResponse = try send(job: 1040, toPrinter: "Bi Sheng")
    print(printerResponse)
} catch {
    print(error)
}
// 使用 try? 将结果转换为可选的
let printerSuccess = try? send(job: 1884, toPrinter: "Mergenthaler")

// 使用 defer 代码块来表示在函数返回前，函数中最后执行的代码。无论函数是否会抛出错误
func fridgeContains(_ food: String) -> Bool {
    fridgeIsOpen = true
    defer {
        fridgeIsOpen = false
    }
    
    let result = fridgeContent.contains(food)
    return result
}

// 断言
// assert 函数来写一个断言。向这个函数传入一个结果为 true 或者 false 的表达式以及一条信息，当表达式的结果为 false 的时候这条信息会被显示.
let age = -3
assert(age >= 0, "A person's age cannot be less than zero")


// guard
// 一个 guard 语句总是有一个 else 从句，如果条件不为真则执行 else 从句中的代码。
guard let name = person["name"] else {
    return
}

// 泛型
// 在尖括号里写一个名字来创建一个泛型函数或者类型
// 也可以创建泛型函数、方法、类、枚举和结构体。
func makeArray<Item>(repeating item: Item, numberOfTimes: Int) -> [Item] {
    var result: [Item] = []
    for _ in 0..<numberOfTimes {
        result.append(item)
    }
    return result
}
makeArray(repeating: "knock", numberOfTimes: 4)

// 在一个类型参数名后面放置一个类名或者协议名，并用冒号进行分隔，来定义类型约束。
// 在类型名后面使用 where 来指定对类型的一系列需求，比如，限定类型实现某一个协议，限定两个类型是相同的，或者限定某个类必须有一个特定的父类。
func allItemsMatch<C1: Container, C2: Container>
    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
    where C1.Item == C2.Item, C1.Item: Equatable {

        // 检查两个容器含有相同数量的元素
        if someContainer.count != anotherContainer.count {
            return false
        }

        // 检查每一对元素是否相等
        for i in 0..<someContainer.count {
            if someContainer[i] != anotherContainer[i] {
                return false
            }
        }

        // 所有元素都匹配，返回 true
        return true
}

// 关联类型
// 关联类型为协议中的某个类型提供了一个占位符名称，其代表的实际类型在协议被遵循时才会被指定。
// 关联类型通过 associatedtype 关键字来指定
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}


// 类型转换
// 使用 is 和 as 操作符。分别提供了一种简单达意的方式去检查值的类型或者转换它的类型。
if a is B {
    print("ok")
} else {
    print("no")
}
if let movie = item as? Movie {
    print("Movie: \(movie.name), dir. \(movie.director)")
} else if let song = item as? Song {
    print("Song: \(song.name), by \(song.artist)")
}
// Any 可以表示任何类型，包括函数类型。
// AnyObject 可以表示任何类类型的实例。


// 不透明类型
// 函数不再提供具体的类型作为返回类型，而是根据它支持的协议来描述返回值。
// 编译器能获取到类型信息，同时模块使用者却不能获取到
// 不透明类型是由函数实现者定义的，而不是由调用者定义的。
// 每次必须返回相同的不透明类型
// 语法:some Protocol
func bar() -> some Equatable { ... }
let z = bar()

// 不透明类型和协议区别: 
// 1. 不能从函数返回带有Self或associatedtype要求的协议。
// 2. 一个函数可以返回不同的协议类型。 相反，它每次必须返回相同的不透明类型。



// 并发 
// Swift 中的并发模型是基于线程的
// 异步函数或异步方法， 在它的声明中的参数列表后边加上 async 关键字
func listPhotos(inGallery name: String) async -> [String] {
    let result = // 省略一些异步网络请求代码
    return result
}
let photoNames = await listPhotos(inGallery: "Summer Vacation")

// 并行的调用异步方法
async let firstPhoto = downloadPhoto(named: photoNames[0])
async let secondPhoto = downloadPhoto(named: photoNames[1])
async let thirdPhoto = downloadPhoto(named: photoNames[2])
let photos = await [firstPhoto, secondPhoto, thirdPhoto]

// 任务组
// 要形成一个任务组，可以调用 withTaskGroup 或 withThrowingTaskGroup，这取决于是否希望可以选择在任务中抛出错误。


// Actor 模型
// actor 来解决数据竞争的问题
// Actor 是一种并发模型，由状态（State）、行为（Behavior）、邮箱（Mailbox）三者组成。
// 1. 状态：actor 持有的变量，由自身管理，避免并发环境下的锁问题；
// 2. 行为：actor 中的计算逻辑，通过 actor 接收到的消息来改变自身的状态；
// 3. 邮箱：actor 之间通讯的桥梁，内部使用 FIFO 队列来存储和处理消息，接收方从邮箱中获取消息。


```

