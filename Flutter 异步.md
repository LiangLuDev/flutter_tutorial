- [Flutter 中的异步编程总结](https://juejin.im/post/5d7a4bd2518825680725bb18)
- [译：Flutter 异步编程：Future、Isolate 和事件循环](https://juejin.im/post/5c898b4af265da2de25bcc2d)


#### Dart 是一种单线程语言

Dart 是单线程的，并且 Flutter 依赖于 Dart，Dart 同一时刻只执行一个操作，其他操作在该操作之后执行。如果某个方法非常耗时，那么在这个方法执行期间，整个应用会被阻塞。

#### Dart 执行模型

当你启动一个 Flutter 应用时，将会创建一个新的线程进程（Dart 中叫做 Isolate），该线程启动后，Dart 会

1. 初始化两个队列（MicroTask 和 Event）
2. 当上面方法执行完成后，执行 main() 方法
3. 启动事件循环

![image](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_future.png?raw=true)

事件循环是一种无限循环，由一个内部时钟控制，在一个时钟周期内，如果没有其它 Dart 代码执行，则执行下面操作

```
void eventLoop(){
    while (microTaskQueue.isNotEmpty){
        fetchFirstMicroTaskFromQueue();
        executeThisMicroTask();
        return;
    }

    if (eventQueue.isNotEmpty){
        fetchFirstEventFromQueue();
        executeThisEventRelatedCode();
    }
}
```

> 可以看到 MicroTask 队列是优于 Event 队列的。

##### MicroTask 队列

MicroTask 队列用于非常简短且需要异步执行的内部动作。例如：你需要在资源关闭后立即释放它，可以这样实现

```
MyResource myResource;

...

void closeAndRelease() {
    // 添加任务到MicroTask
    scheduleMicroTask(_dispose);
    // 或者使用 Future.microtask(_dispose) 添加
    _close();
}

void _close(){
    // 代码以同步的方式运行
    // 以关闭资源
    ...
}

void _dispose(){
    // 代码在
    // _close() 方法
    // 完成后执行
}
```

> 大多数时候我们是不需要使用 MicroTask 的，整个 Flutter 源码中 scheduleMicroTask() 仅被引用了 7 次，优先考虑使用 Event 队列。


##### Event 队列

Event 队列适用于以下模型

- 外部事件如：I/O，手势，绘图，计时器，流
- futures：Future 操作也是通过 Event 队列处理。

> micro task 没有运行时，事件循环将考虑 Event 队列的第一项并执行它。

##### Future

**Future 是一个异步执行并且在未来某一时刻完成（或失败）的任务。**

当实例化一个 Future 时：

- 该 Future 的一个实例被创建并记录在由 Dart 管理的内部数组中；
- 需要由此 Future 执行的代码直接推送到 Event 队列中去；
- 该 future 实例 返回一个状态（= incomplete）；
- 如果存在下一个同步代码，执行它（非 Future 的执行代码）

只要事件循环从 Event 循环中获取它，被 Future 引用的代码将像其他任何 Event 一样执行。
当该代码将被执行并将完成（或失败）时，then() 或 catchError() 方法将直接被触发。

例：

```
void main(){
    print('Before the Future');
    Future((){
        print('Running the Future');
    }).then((_){
        print('Future is complete');
    });
    // 非 Future 执行代码执行
    print('After the Future');
}
```

运行结果

```
Before the Future
After the Future
Running the Future
Future is complete
```

1. print(‘Before the Future’)
1. 将 (){print(‘Running the Future’);} 添加到 Event 队列；
1. print(‘After the Future’)
1. 事件循环获取（在第二步引用的）代码并执行它
1. 当代码执行时，它会查找 then() 语句并执行它


> Future 并非并行执行，而是遵循事件循环的顺序执行规则执行。


##### Async 方法

使用 async 关键字作为方法声明的后缀时，Dart 会将其理解为：

- 该方法的返回值是一个 Future；
- 它同步执行该方法的代码直到第一个 await 关键字，然后它暂停该方法其他部分的执行；
- 一旦由 await 关键字引用的 Future 执行完成，下一行代码将立即执行。

> await 不会暂停整个执行流程直到他执行结束

例：

```
void main() async {
  methodA();
  await methodB();
  await methodC('main');
  methodD();
}

methodA(){
  print('A');
}

methodB() async {
  print('B start');
  await methodC('B');
  print('B end');
}

methodC(String from) async {
  print('C start from $from');

  Future((){                // 该代码将在未来的某个时间段执行
    print('C running Future from $from');
  }).then((_){
    print('C end of Future from $from');
  });

  print('C end from $from');
}

methodD(){
  print('D');
}
```

执行顺序

```
A
B start
C start from B
C end from B
B end
C start from main
C end from main
D
C running Future from B
C end of Future from B
C running Future from main
C end of Future from main
```

如果你希望在所有代码的末尾执行 methodD(),那么你需要做如下修改

```
void main() async {
  methodA();
  await methodB();
  await methodC('main');
  methodD();
}

methodA(){
  print('A');
}

methodB() async {
  print('B start');
  await methodC('B');
  print('B end');
}

methodC(String from) async {
  print('C start from $from');

  await Future((){                  // 在此处进行修改
    print('C running Future from $from');
  }).then((_){
    print('C end of Future from $from');
  });
  print('C end from $from');
}

methodD(){
  print('D');
}
```

输出序列为：

```
A
B start
C start from B
C running Future from B
C end of Future from B
C end from B
B end
C start from main
C running Future from main
C end of Future from main
C end from main
D
```

例：method1 和 method2 的执行结果会一样吗

```
void method1(){
  List<String> myArray = <String>['a','b','c'];
  print('before loop');
  myArray.forEach((String value) async {
    await delayedPrint(value);
  });
  print('end of loop');
}

void method2() async {
  List<String> myArray = <String>['a','b','c'];
  print('before loop');
  for(int i=0; i<myArray.length; i++) {
    await delayedPrint(myArray[i]);
  }
  print('end of loop');
}

Future<void> delayedPrint(String value) async {
  await Future.delayed(Duration(seconds: 1));
  print('delayedPrint: $value');
}
```

执行结果：

method1() | method2()
- |-
1.  before loop|1.  before loop
2.  end of loop | 2.  delayedPrint: a (after 1 second)
3.  delayedPrint: a (after 1 second) |3.  delayedPrint: b (1 second later)
4.  delayedPrint: b (directly after)|4.  delayedPrint: c (1 second later)
5.  delayedPrint: c (directly after)|5.  end of loop (right after)

method1 使用 forEach() 函数来遍历数组。每次迭代时，它都会调用一个被标记为 async（因此是一个 Future）的新回调函数。执行该回调直到遇到 await，而后将剩余的代码推送到 Event 队列。一旦迭代完成，它就会执行下一个语句：“print(‘end of loop’)”。执行完成后，事件循环 将处理已注册的 3 个回调。


对于 method2，所有的内容都运行在一个相同的代码块中，因此能够一行一行按照顺序执行

#### 多线程

Isolate 是 Dart 中的线程，它与 Java 中的线程有较大差异，Isolate 在 Flutter 中并不共享内存，不同 Isolate 通过消息进行通信。


##### 每个 Isolate 有自己独立的事件循环

每个 Isolate 都有自己的事件循环，所以一个 Isolate 中运行的代码与另一个不存在任何关联，所以我们可以做并行处理。


##### 启动 Isolate

方法1： 通过 Dart 提供的 API

- 第一步：创建并握手，Isolate 通过消息进行交互，因此我们需要建立调用者 Isolate 与新的 Isolate 之间的通信。每个 Isolate 都暴露了一个将消息传递给 Isolate 的被称为 SendPort 的端口，意味着调用者和新 Isolate 需要知道彼此的端口才能进行通信。这个过程如下：


```
// 新的 isolate 端口,该端口将在未来使用用来给 isolate 发送消息
SendPort newIsolateSendPort;

// 新 Isolate 实例
Isolate newIsolate;

// 启动一个新的 isolate 然后开始第一次握手
void callerCreateIsolate() async {
    // 本地临时 ReceivePort用于检索新的 isolate 的 SendPort
    ReceivePort receivePort = ReceivePort();

    // 初始化新的 isolate
    newIsolate = await Isolate.spawn(
        callbackFunction,
        receivePort.sendPort,
    );

    // 检索要用于进一步通信的端口
    newIsolateSendPort = await receivePort.first;
}

// 新 isolate 的入口
static void callbackFunction(SendPort callerSendPort){
    // 一个 SendPort 实例，用来接收来自调用者的消息
    ReceivePort newIsolateReceivePort = ReceivePort();

    // 向调用者提供此 isolate 的 SendPort 引用
    callerSendPort.send(newIsolateReceivePort.sendPort);

    // 进一步流程
}
```

> 约束： isolate 的入口必须是顶级函数或者静态方法

- 第二步：向 Isolate 提交消息

```
// 向新 isolate 发送消息并接收回复的方法

// 在该例中，使用字符串进行通信操作
Future<String> sendReceive(String messageToBeSent) async {
    // 创建一个临时端口来接收回复
    ReceivePort port = ReceivePort();

    // 发送消息到 Isolate，并且通知该 isolate 哪个端口是用来提供回复的
    newIsolateSendPort.send(
        CrossIsolatesMessage<String>(
            sender: port.sendPort,
            message: messageToBeSent,
        )
    );

    // 等待回复并返回
    return port.first;
}

// 扩展回调函数来处理接输入报文
static void callbackFunction(SendPort callerSendPort){
    // 初始化一个 SendPort 来接收来自调用者的消息
    ReceivePort newIsolateReceivePort = ReceivePort();

    // 向调用者提供该 isolate 的 SendPort 引用
    callerSendPort.send(newIsolateReceivePort.sendPort);

    // 监听输入报文、处理并提供回复的 Isolate 主程序
    newIsolateReceivePort.listen((dynamic message){
        CrossIsolatesMessage incomingMessage = message as CrossIsolatesMessage;

        // 处理消息
        String newMessage = "complemented string " + incomingMessage.message;

        // 发送处理的结果
        incomingMessage.sender.send(newMessage);
    });
}

// 帮助类
class CrossIsolatesMessage<T> {
    final SendPort sender;
    final T message;

    CrossIsolatesMessage({
        @required this.sender,
        this.message,
    });
}
```

- 第三步：销毁这个新的 Isolate 实例

```
void dispose(){
    newIsolate?.kill(priority: Isolate.immediate);
    newIsolate = null;
}
```

##### 一次性计算

如果你只需要运行一些代码来完成一些特定的工作，并且在工作完成之后不需要与 Isolate 进行交互，那么这里有一个非常方便的称为 [compute](https://docs.flutter.io/flutter/foundation/compute.html) 的 Helper。
主要包含以下功能：

- 产生一个 Isolate，
- 在该 isolate 上运行一个回调函数，并传递一些数据，
- 返回回调函数的处理结果，
- 回调执行后终止 Isolate。


> 约束「回调」函数必须是顶级函数并且不能是闭包或类中的方法（静态或非静态）。

##### Future 和 Isolate 的选择

- 如果一个方法需要几毫秒 => Future
- 如果一个处理流程需要几百毫秒 => Isolate
