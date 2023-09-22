---
title: Swift中的异步编程方式
date: 2023-09-22
categories: Swift
tags: []
---
# Swift中的异步编程方式

## 引

说到异步编程，我们很容易想到的编译回调。无论是需要并行的耗时任务，还是允许串行的简单任务，都通过回调的方式返回结果。回调也是在开发中使用最为广泛的一种异步编程方式。回想一下，通常的网络请求，文件操作等函数都会提供一个回调参数。回调使用起来虽然方便，但其并不利于进行程序流程的控制，仅仅从代码层面看，也很难组织清楚代码的执行顺序和逻辑。

Swift从代码层面提供了结构化的方式来支持异步编程，在Swift5.5中引入了async和await相关的关键字。需要注意，异步和并行本身是两个概念，在Swift中，异步编程模型已经建立在线程调度之上，这也就是说，我们无需关心其中线程的调用，异步的函数本身就是在子线程中并行执行的，线程切换和调度全有语言本身控制。但是Swift不会保证函数会在哪个特定的线程上执行。

## 异步函数

在尝试Swift中提供的异步编程方式外，可以先回想下对于异步并行的场景，之前是如何处理的，例如下面的代码：

```swift
func test(callback: @escaping (_ success: Bool)->Void) {
    DispatchQueue.global().async {
        print("Test任务完成", Thread.current)
        callback(true)
    }
}
print("Begin", Thread.current)
test { success in
    print("EndTest")
}
print("End", Thread.current)
```

其中test函数所执行的任务是手动放在全局队列中执行的，DispatchQueue会自动的进行线程的调度，将其分配到空闲的子线程来执行。打印结果如下：

```
Begin <_NSMainThread: 0x600002310100>{number = 1, name = main}
End <_NSMainThread: 0x600002310100>{number = 1, name = main}
Test任务完成 <NSThread: 0x600002300300>{number = 5, name = (null)}
EndTest
```

上面的示例代码比较简单，如果有更多的异步任务是依赖test任务的，则闭包回调的写法就会变得非常丑陋，代码难以阅读和维护。

在Swift5.5之后，我们可以使用async关键字来定义异步函数，编程模型会自动分配线程执行，例如：

```swift
func test1() async -> Bool {
    print("ts1", Thread.current)
    return true
}

func test2() async -> Bool {
    print("ts2", Thread.current)
    return true
}

print("Begin", Thread.current)
async let a = test1()
async let b = test2()
print("End", Thread.current)
```

上面代码中，async关键字将函数标记为异步的，异步函数是一种特殊的函数，其支持在执行过程中被暂时的挂起，即暂停。对于普通的函数来说，会有3种状态：

1\. 执行完成

2\. 抛出异常

3\. 永不返回

异步函数对应的也会有这3种状态，不同的是，当需要做某些等待操作时，其可以暂时的挂起。运行上面的代码，打印效果如下：

```
Begin <_NSMainThread: 0x60000329c3c0>{number = 1, name = main}
End <_NSMainThread: 0x60000329c3c0>{number = 1, name = main}
ts1 <NSThread: 0x60000328cc40>{number = 4, name = (null)}
ts2 <NSThread: 0x600003282d40>{number = 6, name = (null)}
```

可以看到，test1和test2两个任务是异步执行，且被分配在了不同的线程。需要注意，理论上在异步函数中是不允许使用Thread相关接口的，因为任务的挂起和恢复所在线程都是由系统调度的，逻辑上开发者无需关心线程问题，在Swift6版本中继续这样使用将会报错。

上面演示的代码中，test1和test2任务的执行并不依赖关系，如果test2和执行是需要test1执行结束的，则可以直接使用await关键字来将运行挂起，直到结果返回。例如：

```swift
func test1() async -> Bool {
    print("ts1", Thread.current)
    return true
}

func test2() async -> Bool {
    print("ts2", Thread.current)
    return true
}

print("Begin", Thread.current)
let a = await test1()
let b = await test2()
print("End", Thread.current)
```

打印效果如下：

```
Begin <_NSMainThread: 0x600002180140>{number = 1, name = main}
ts1 <NSThread: 0x600002198100>{number = 6, name = (null)}
ts2 <NSThread: 0x6000021accc0>{number = 8, name = (null)}
End <_NSMainThread: 0x600002180140>{number = 1, name = main}
```

使用await关键字标记的地方为程序的挂起点，此时会停止当前线程上代码的执行，并等待异步函数的返回，在程序中，支持await进行挂起的场景包括：

1.异步的方法，属性或函数中。

2.main代码块中。

3.非结构化的子任务代码块中。

通常，我们直接使用await调用异步函数时，当前执行会被挂起，更多时候可以使用如下方式来同时执行多个异步函数，使用await来最终获得结果：

```swift
func test1() async -> Bool {
    print("ts1", Thread.current)
    return true
}

func test2() async -> Bool {
    print("ts2", Thread.current)
    return true
}

print("Begin", Thread.current)
async let a = test1()
async let b = test2()
let res = await [a ,b]
print(res)
print("End", Thread.current)
```

## 异步序列

Swift中的迭代也支持异步返回，通过AsyncIteratorProtocol协议来定义异步的迭代器，示例如下：

```swift
struct Group: AsyncSequence {
    typealias Element = Int

    let limit: Int

    struct AsyncIterator : AsyncIteratorProtocol {
        let limit: Int
        var current = 1
        mutating func next() async -> Int? {
            guard !Task.isCancelled else {
                return nil
            }

            guard current <= limit else {
                return nil
            }

            let result = current
            current += 1
            return result
        }
    }

    func makeAsyncIterator() -> AsyncIterator {
        return AsyncIterator(limit: limit)
    }
}
print("Begin")
let group = Group(limit: 10)

for await i in group {
    print(i)
}
print("End")
```

在对异步迭代器进行遍历时，需要使用for await in的语法方式。

## 任务组与任务

当有多个异步任务需要执行时，可以将其添加到一个任务组中，当任务组所有任务完成后再进行统一的返回。例如：

```swift
let res = await withTaskGroup(of: Bool.self, returning: [Bool].self, body: { taskGroup in
    taskGroup.addTask {
        let a = await test1()
        return a
    }
    taskGroup.addTask {
        let b = await test1()
        return b
    }
    var datas:[Bool] = []
    for await data in taskGroup {
        datas.append(data)
    }
    return datas
})

print(res)
```

其中，withTaskGroup方法将创建一个异步的父任务，其中可以添加多个子任务，任务组之间有非常明确的关系，这种编程方式也被称为结构化编程，当然，Swift也提供了非结构化的编程方式，即需要开发者处理任务之间的关系。这非常有用，有时我们需要在非并发的环境中调用异步函数，例如在iOS Application中ViewController的viewDidLoad方法中调用一个异步的函数，此时就需要为其包装一个并发环境，使用Task来创建任务即可：

```swift
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        Task(priority: .background) {
            await test()
            self.view.backgroundColor = .red
            print("Finish")
        }
        // 上面task的执行不会影响当前线程
        print("Continue")
    }

    func test() async {
        try? await Task.sleep(for: .seconds(10))
    }

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        print("touch")
        print(Thread.current)
    }
}
```

这里再强调一下，所谓执行任务的挂起和线程的阻塞完全不同，当并发环境中当前任务被挂起时，线程资源会被释放去执行其他任务，直到异步任务有结果后，在恢复执行。上面代码中并没有记录Task实例，其实此实例可以控制任务的取消，获取任务返回值等操作，例如：

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    let task = Task(priority: .background) {
        await test()
        self.view.backgroundColor = .red
        print("Finish")
        return "result"
    }
    // 上面task的执行不会影响当前线程
    print("Continue")
    
    // 取消任务
    task.cancel()
}
```