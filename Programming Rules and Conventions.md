[原文链接][http://www.erlang.se/doc/programming_rules.shtml]

[TOC]

> Author: EPK/NP Klas Eriksson, EUA/SU M Williams, J Armstrong
> Document: EPK/NP 95:035

作者: EPK/NP Klas Eriksson, EUA/SU M Williams, J Armstrong
文件: EPK/NP 95:035

> Program Development Using Erlang - Programming Rules and Conventions
# Erlang 程序开发 - 编程规则及约定

> Abstract
**摘要**

> This is a description of programming rules and advise for how to write systems using Erlang.
本文对编程规范的描述，并建议如何使用 **Erlang** 编写系统。


> Note: This document is a preliminary document and is not complete.
**备注: 本文档只是初步的文档，仍不完善。**

> The requirements for the use of EBC's "Base System" are not documented here, but must be followed at a very early design phase if the "Base System" is to be used. These requirements are documented in 1/10268-AND 10406 Uen "MAP - Start and Error Recovery".
`EBC's "Base Sytem"` 的使用要求在这里没有记录，如果要用到 `Base System` 的话，则必须在设计的初期就遵循其使用要求。其内容被记录在 `1/10268-AND 10406 Uen "MAP - Start and Error Recovery"`

> 1 Purpose
## 1 目的

> This paper lists some aspects which should be taken into consideration when specifying and programming software systems using Erlang. It does not attempt to give a complete description of general specification and design activities which are independent of the use of Erlang.
本文列举了在使用 Erlang 进行编码应考虑到的一些方面。不会试图描述一些和 Erlang 语言本身无关的，例如 完整系统的规范、和设计行为。


> 2 Structure and Erlang Terminology
## 2 结构和Eralng术语

> Erlang systems are divided into modules. Modules are composed of functions and attributes. Functions are either only visible inside a module or they are exported i.e. they can also be called by other functions in other modules. Attributes begin with "-" and are placed in the beginning of a module.
Erlang 程序由多个独立的模块(Modules)构成。模块则由一系列函数(Functions)和属性(Attributes)组成。函数要么仅在模块内部可见，要么被导出(Export)为外部可见的。导出后可以被其它模块的函数所调用。属性以 "-" 开头，并放在模块的开头。


> The work in a system designed using Erlang is done by processes. A process is a job which can use functions in many modules. Processes communicate with each other by sending messages. Processes receive messages which are sent to them, a process can decide which messages it is prepared to receive. Other messages are queued until the receiving process is prepared to receive them.
使用 Erlang 设计的系统中的任务是由进程(Process)完成的。进程是一个能执行多个模块中函数的任务，进程通过发送消息(Message Passing)彼此通信，进程能接收发送给它们的消息，进程也能决定它准备接收哪些消息。其他消息将会被排队，直到该进程准备接收它们。

> A process can supervise the existence of another process by setting up a link to it. When a process terminates, it automatically sends exit signals to the process to which it is linked. The default behavior of a process receiving an exit signal is to terminate and to propagate the signal to its linked processes. A process can change this default behavior by trapping exits, this causes all exit signals sent to a process to be turned into messages.
一个进程可以通过设置一个链接(Link)来监视另一个进程的存在。当一个进程终止时，它会自动向其链接的进程发送退出信号。当接收到退出信号时，进程默认将终止自身并将退出信号传递到它所链接的进程。进程可以通过捕获退出(Trapping exits)来改变此默认行为，这将导致所有发送到该进程的退出信号都将被转换成普通消息。


> A pure function is a function that returns the same value given the same arguments regardless of the context of the call of the function. This is what we normally expect from a mathematical function. A function that is not pure is said to have side effects.
纯函数(Pure Function)是指: 相同的参数会有相同的返回值，而与函数调用时的上下文无关。这就是我们通常所期望的数学函数。非纯函数则是指有副作用(Side effects)的函数。


> Side effects typically occur if a function a) sends a message b) receives a message c) calls exit d) calls any BIF which changes a processes environment or mode of operation (e.g. get/1, put/2, erase/1, process_flag/2 etc).
副作用通常出现在，如果一个函数具有:
- 发送一个消息
- 接收一个信息
- 调用 exit 方法
- 调用任何改变进程的上下文环境、模式的 BIF 方法（例如 get/1, get/2, put/1, erase/1，process_flag/2 等）


> Warning: This document contains examples of bad code.
**警告：此文档包含的代码示例不能直接被编译运行。**

>> 3 SW Engineering Principles
>> 3.1 Export as few functions as possible from a module
## 3 软件工程的原则
### 3.1 尽可能少的导出函数

> Modules are the basic code structuring entity in Erlang. A module can contain a large number of functions but only functions which are included in the export list of the module can be called from outside the module.
模块是 Erlang 最基本的代码结构实体。一个模块可以包含大量的函数，但只有包含在 **export** 列表中的函数，才能被模块外部所调用。


> Seen from the outside the complexity of a module depends upon the number of functions which are exported from the module. A module which exports one or two functions is usually easier to understand than a module which exports dozens of functions.
从外部看，模块的复杂性取决于从模块导出的函数数量。导出一个或两个函数的模块通常比导出许多个函数的模块更容易理解。


> Modules where the ratio of exported/non-exported functions is low are desirable in that a user of the module only needs to understand the functionality of the functions which are exported from the module.
导出与未导出函数比例相对低的模块是更合适的，因为模块的用户只需要理解从模块中导出的函数的功能。

> In addition, the writer or maintainer of the code in the module can change the internal structure of the module in any appropriate manner provided the external interface remains unchanged.
此外，如果外部接口保持不变，模块代码的编写者或维护者可以以任何适当的方式更改模块的内部结构。

> 3.2 Try to reduce intermodule dependencies
### 3.2 尽量减少模块间的依赖

> A module which calls functions in many different modules will be more difficult to maintain than a module which only calls functions in a few different modules.
一个模块被许多不同的模块调用要比只在几个不同模块中调用更难维护


> This is because each time we make a change to a module interface, we have to check all places in the code where this module is called. Reducing the interdependencies between modules simplifies the problem of maintaining these modules.
这是因为每次我们更改一个模块接口时，我们必须检查代码中这个模块被调用的所有位置。所以，减少模块之间的相互依赖关系简化了维护这些模块的问题。


> We can simplify the system structure by reducing the number of different modules which are called from a given module.
我们可以通过减少给定模块被不同模块调用的数量，来简化系统结构。


> Note also that it is desirable that the inter-module calling dependencies form a tree and not a cyclic graph. Example:
还要注意的是，模块间的调用依赖，应该是形成一颗树而不是循环图。 例：
![module-dep-ok](http://www.erlang.se/images/module-dep-ok.gif)

> But not
而不是

![module-dep-bad](http://www.erlang.se/images/module-dep-bad.gif)

> 3.3 Put commonly used code into libraries
### 3.3 将常用代码放入库中

> Commonly used code should be placed into libraries. The libraries should be collections of related functions. Great effort should be made in ensuring that libraries contain functions of the same type. Thus a library such as lists containing only functions for manipulating lists is a good choice, whereas a library, lists_and_maths containing a combination of functions for manipulating lists and for mathematics is a very bad choice.
通用的代码应该放在库(Libraries)中。每个库应该是相关函数的集合。应该尽力确保库包含相同类型的函数。 因此，列表库只包含列表操作相关的函数是一个好的选择，而一个 lists_and_matchs 库中混合了列表操作和数学函数是一个非常糟糕的选择。

> The best library functions have no side effects. Libraries with functions with side effects limit the re-usability.
最好的库函数没有副作用。具有副作用函数的库限制了其可重用性。


> 3.4 Isolate "tricky" or "dirty" code into separate modules
### 3.4 分离“干净”和“脏”代码到不同的模块

> Often a problem can be solved by using a mixture of clean and dirty code. Separate the clean and dirty code into separate modules.
通常可以通过使用干净(Tricky)和脏(Dirty)的代码混合来解决问题。将干净和脏的代码分离成单独的模块。

> Dirty code is code that does dirty things. Example:
>  Uses the process dictionary. 
>  Uses erlang:process_info/1 for strange purposes. 
>  Does anything that you are not supposed to do (but have to do). 
"脏代码" 是指做一些 "脏的事情"。例：
- 使用进程字典
- 为了奇怪的目的，使用 `erlang:process_info/1`
- 有什么你不应该做的（但必须做）。

> Concentrate on trying to maximize the amount of clean code and minimize the amount of dirty code. Isolate the dirty code and clearly comment or otherwise document all side effects and problems associated with this part of the code.
集中精力试图最大限度地提高干净的代码量，并尽量减少脏代码的数量。隔离脏代码，并清楚地注释或以其他方式记录与这部分代码相关的所有副作用和问题。

> 3.5 Don't make assumptions about what the caller will do with the results of a function
### 3.5 不要假设调用者会对函数的结果做什么

> Don't make assumptions about why a function has been called or about what the caller of a function wishes to do with the results.
不要对函数为什么被调用和调用者希望如何处理结果做出假设

> For example, suppose we call a routine with certain arguments which may be invalid. The implementer of the routine should not make any assumptions about what the caller of the function wishes to happen when the arguments are invalid.
例如，假设我们使用一些可能无效的参数去调用一个函数。常规的实现者在参数无效时，不应该对函数的调用者希望发生什么做出任何假设。

> Thus we should not write code like
所以我们不应该这样写代码：

```erlang
do_something(Args) -> 
  case check_args(Args) of 
    ok -> 
      {ok, do_it(Args)}; 
    {error, What} -> 
      String = format_the_error(What), 
      io:format("* error:~s\n", [String]), %% Don't do this
      error 
  end.
```

> Instead write something like:
而是写一些像：

```erlang
do_something(Args) ->
  case check_args(Args) of
    ok ->
      {ok, do_it(Args)};
    {error, What} ->
      {error, What}
  end.

error_report({error, What}) ->
  format_the_error(What).
```

> In the former case the error string is always printed on standard output, in the latter case an error descriptor is returned to the application. The application can now decide what to do with this error descriptor.
在前一种情况下，错误字符串总是打印在标准输出上，在后一种情况下，错误描述符返回给应用程序。应用程序现在可以决定如何处理这个错误描述符。


> By calling error_report/1 the application can convert the error descriptor to a printable string and print it if so required. But this may not be the desired behavior - in any case the decision as to what to do with the result is left to the caller.
通过调用 `error_report/1`，应用程序可以将错误描述符转换为可打印的字符串，并在需要时打印出来。但是这可能不是预期的行为 —— 但无论如何，决定如何处理结果都应该留给调用者。

> 3.6 Abstract out common patterns of code or behavior
### 3.6 抽象出常见的代码或行为模式

> Whenever you have the same pattern of code in two or more places in the code try to isolate this in a common function and call this function instead of having the code in two different places. Copied code requires much effort to maintain.
每当你的代码在两个或两个以上的地方有相同模式时，试着用一个普通的函数来隔离这个代码，然后调用这个函数，而不是让代码在两个不同的地方。复制式的代码需要很多功夫来维护。

> If you see similar patterns of code (i.e. almost identical) in two or more places in the code it is worth taking some time to see if one cannot change the problem slightly to make the different cases the same and then write a small amount of additional code to describe the differences between the two.
如果在代码中的两个或两个以上的地方看到类似的代码模式（即几乎相同），那么值得花一些时间来看看是否能稍微改变下问题以使不同的情况相同，然后再写入少量的附加代码来描述两者之间的差异。


> Avoid "copy" and "paste" programming, use functions!
使用函数！避免 "复制" 和 "粘贴" 式编程。

> 3.7 Top-down
### 3.7 自上而下

> Write your program using the top-down fashion, not bottom-up (starting with details). Top-down is a nice way of successively approaching details of the implementation, ending up with defining primitive functions. The code will be independent of representation since the representation is not known when the higher levels of code are designed.
用自上而下的方式编写程序，而不是自下而上（从细节开始）。 自上而下是一个不断接近实现细节的好方法，结束于定义原函数。 代码将与（底层）表示无关，因为在设计更高级别的代码时，表示形式是不知道的。

> 3.8 Don't optimize code
### 3.8 不要优化代码

> Don't optimize your code at the first stage. First make it right, then (if necessary) make it fast (while keeping it right).
不要在第一阶段优化你的代码。首先做对，然后（如果有必要的话）再让他变快（保持正确）。

> 3.9 Use the principle of "least astonishment"
### 3.9 采用 “最少惊讶” 的原则

> The system should always respond in a manner which causes the "least astonishment" to the user - i.e. a user should be able to predict what will happen when they do something and not be astonished by the result.
系统应该总是以对用户 “最小惊讶” 的方式进行响应 —— 即用户应该能够预测当他们做某事时会发生什么，而不会被结果惊讶。

> This has to do with consistency, a consistent system where different modules do things in a similar manner will be much easier to understand than a system where each module does things in a different manner.
这与一致性有关，一个一致的系统，其中不同的模块以类似的方式进行操作的系统比其中每个模块以不同方式执行操作的系统更容易理解。


> If you get astonished by what a function does, either your function solves the wrong problem or it has a wrong name.
如果你对某个函数的功能感到惊讶，那你的函数要么解决了错误的问题，要么函数的命名存在错误。

> 3.10 Try to eliminate side effects
### 3.10 尽量消除副作用

> Erlang has several primitives which have side effects. Functions which use these cannot be easily re-used since they cause permanent changes to their environment and you have to know the exact state of the process before calling such routines.
Erlang 存在几个有副作用的原语。使用这些原语的函数不能轻易的被重用，因为它们会造成进程环境永久的更改，并且在调用此类例程之前你必须知道进程的确切状态。

> Write as much as possible of the code with side-effect free code.
尽可能多地编写无副作用的代码。

> Maximize the number of pure functions.
最大限度地提高纯函数的数量。

> Collect together the functions which have side effect and clearly document all the side effects.
收集有副作用的函数，清楚地（用文档/注释）记录所有的副作用。

> With a little care most code can be written in a side-effect free manner - this will make the system a lot easier to maintain, test and understand.
稍微注意一下，大多数代码可以用无副作用的方式编写 —— 这将使系统更容易维护，测试和理解。

> 3.11 Don't allow private data structure to "leak" out of a module
### 3.11 不要让私有数据结构暴露到模块外

> This is best illustrated by a simple example. We define a simple module called queue - to implement queues:
最好用一个简单的例子来说明。我们定义一个简单的模块 `queue`，来实现队列：

```erlang
-module(queue).
-export([add/2, fetch/1]).

add(Item, Q) -> 
  lists:append(Q, [Item]).

fetch([H|T]) -> 
  {ok, H, T}; 
fetch([]) -> 
  empty.
```

> This implements a queue as a list, unfortunately to use this the user must know that the queue is represented as a list. A typical program to use this might contain the following code fragment:
这里用列表实现了一个队列，不幸的是使用这个的用户必须知道该队列被表示为一个列表。使用它的一个典型例程可能包含以下代码片段：

```erlang
NewQ = [], % Don't do this
Queue1 = queue:add(joe, NewQ), 
Queue2 = queue:add(mike, Queue1), ....
```

> This is bad - since the user a) needs to know that the queue is represented as a list and b) the implementer cannot change the internal representation of the queue (this they might want to do later to provide a better version of the module).
这是不好的实现，因为用户：
- 需要知道队列被表示为列表
- 并且，实现者不能改变队列的内部表示（可能以后他们想要提供更好的版本）。

> Better is:
更好的实现:

```erlang
-module(queue).
-export([new/0, add/2, fetch/1]).

new() -> 
  [].

add(Item, Q) -> 
  lists:append(Q, [Item]).

fetch([H|T]) -> 
  {ok, H, T}; 
fetch([]) -> 
  empty.
```
> Now we can write:
现在，我们可以这样去使用：

```erlang
NewQ = queue:new(), 
Queue1 = queue:add(joe, NewQ), 
Queue2 = queue:add(mike, Queue1), ...
```

> Which is much better and corrects this problem. Now suppose the user needs to know the length of the queue, they might be tempted to write:
这一个更好，而且纠正了上述的问题。现在假设用户需要知道队列的长度，他们可能会试图写：

```erl
Len = length(Queue) % Don't do this
```

> since they know that the queue is represented as a list. Again this is bad programming practice and leads to code which is very difficult to maintain and understand. If they need to know the length of the queue then a length function must be added to the module, thus:
因为他们知道队列被表示为一个列表。这又是一个糟糕的编程习惯，导致代码难以维护和理解。如果他们需要知道队列的长度，则只须将长度函数添加到模块中，从而：

```erl
-module(queue).
-export([new/0, add/2, fetch/1, len/1]).

new() -> [].

add(Item, Q) ->
  lists:append(Q, [Item]).

fetch([H|T]) -> 
  {ok, H, T}; 

fetch([]) -> 
  empty.

len(Q) -> 
  length(Q).
```

> Now the user can call `queue:len(Queue)` instead.
现在用户可以调用 `queue:len(Queue)` 来代替。

> Here we say that we have "abstracted out" all the details of the queue (the queue is in fact what is called an "abstract data type").
在这里，我们已经 “抽象出” 队列的所有细节（`queue` 实际上是所谓的 “抽象数据类型”）。

> Why do we go to all this trouble? - the practice of abstracting out internal details of the implementation allows us to change the implementation without changing the code of the modules which call the functions in the module we have changed. So, for example, a better implementation of the queue is as follows:
为什么我们要去面对这些麻烦？ —— 抽象出内部细节的实现的这种做法，允许我们在改变实现时，而不改变调用这些函数的模块的代码。所以，比如更好的队列实现如下：

```erl
-module(queue).
-export([new/0, add/2, fetch/1, len/1]).

new() -> 
  {[],[]}.

add(Item, {X,Y}) -> % Faster addition of elements
  {[Item|X], Y}.

fetch({X, [H|T]}) -> 
  {ok, H, {X,T}}; 

fetch({[], []) -> 
  empty; 

fetch({X, []) -> 
  % Perform this heavy computation only sometimes.
  fetch({[],lists:reverse(X)}).

len({X,Y}) -> 
  length(X) + length(Y).
```

> 3.12 Make code as deterministic as possible
### 3.12 增加代码的确定性

> A deterministic program is one which will always run in the same manner no matter how many times the program is run. A non-deterministic program may deliver different results each time it is run. For debugging purposes it is a good idea to make things as deterministic as possible. This helps make errors reproducible.
一个确定性的程序是指无论程序运行多少次，它都总是以相同的方式运行。非确定性程序每次运行时可能会产生不同的结果。出于调试的目的，尽可能的增加确定性是一个好主意。这也有助于错误重现。

> For example, suppose one process has to start five parallel processes and then check that they have started correctly, suppose further that the order in which these five are started does not matter.
例如，假设一个进程必须启动五个并行进程，然后检查它们是否已经正确启动，进一步假设这五个启动顺序并不重要。

> We could then choose to either start all five in parallel and then check that they have all started correctly but it would be better to start them one at a time and check that each one has started correctly before starting the next one.
所以，我们可以选择并行启动所有的五个进程，然后检查它们是否全部启动正确，但是更好的方式是一次启动一个，并在启动下一个之前检查每个启动是否正确。

> 3.13 Do not program "defensively"
### 3.13 拒绝 “防御式” 编程

> A defensive program is one where the programmer does not "trust" the input data to the part of the system they are programming. In general one should not test input data to functions for correctness. Most of the code in the system should be written with the assumption that the input data to the function in question is correct. Only a small part of the code should actually perform any checking of the data. This is usually done when data "enters" the system for the first time, once data has been checked as it enters the system it should thereafter be assumed correct.
防御性程序是指程序员由于不 "信任" 输入数据到正在编写的系统，而编写的部分程序。一般来说，不应该测试输入函数的数据是否正确。系统中的大部分代码应该假定输入的数据是正确的。只有一小部分代码应该实际执行任何数据检查。这通常是在数据首次进入系统时完成的，一旦数据进入系统后，数据就已经被检查好了，其后都应假定其为正确的数据。

> Example:
例如:

```erl
%% Args: Option is all|normal
get_server_usage_info(Option, AsciiPid) ->
  Pid = list_to_pid(AsciiPid),
  case Option of
    all -> get_all_info(Pid);
    normal -> get_normal_info(Pid)
  end.
```

> The function will crash if Option neither normal nor all, and it should do that. The caller is responsible for supplying correct input.
如果 Option 既不是 `normal` 也不是 `all`，那么函数就会崩溃，也应该这样做。调用者应该负责提供正确的输入。


> 3.14 Isolate hardware interfaces with a device driver
### 用设备驱动程序隔离硬件接口

> Hardware should be isolated from the system through the use of device drivers. The device drivers should implement hardware interfaces which make the hardware appear as if they were Erlang processes. Hardware should be made to look and behave like normal Erlang processes. Hardware should appear to receive and send normal Erlang messages and should respond in the conventional manner when errors occur.
硬件(Hardware)应该通过使用设备驱动(Device driver)程序与系统隔离。设备驱动程序应该实现硬件接口(Hardware Interface)，使硬件看起来像是 Erlang 进程。 驱动程序也应该被设计为看起来和正常的Erlang进程一样，它能接收和发送正常的 Erlang 消息，并且在发生错误时，以传统的方式作出响应。

> 3.15 Do and undo things in the same function
### 3.15 保持操作的对称

> Suppose we have a program which opens a file, does something with it and closes it later. This should be coded as:
假设我们有用来打开一个文件的程序，做一些事情并稍后关闭它。这应该编码为：

```erl
do_something_with(File) -> 
  case file:open(File, read) of, 
    {ok, Stream} ->
      doit(Stream), 
      file:close(Stream) % The correct solution
    Error -> Error
  end.
```

> Note the symmetry in opening the file (file:open)and closing it (file:close) in the same routine. The solution below is much harder to follow and it is not obvious which file that is closed. Don't program it like this:
注意在同一函数中保持打开文件 `file:open` 和关闭文件 `file:close` 的对称性。下面的实现更加难以使用，且不清楚哪个文件被关闭。不要这样编程：

```erl
do_something_with(File) -> 
  case file:open(File, read) of, 
    {ok, Stream} ->
      doit(Stream)
    Error -> Error
  end.

doit(Stream) -> 
  ...., 
  func234(...,Stream,...).
  ...

func234(..., Stream, ...) ->
  ...,
  file:close(Stream) %% Don't do this
```

> 4 Error Handling
## 错误处理

> 4.1 Separate error handling and normal case code
### 分离错误处理和正常代码

> Don't clutter code for the "normal case" with code designed to handle exceptions. As far as possible you should only program the normal case. If the code for the normal case fails, your process should report the error and crash as soon as possible. Don't try to fix up the error and continue. The error should be handled in a different process (See "Each process should only have one "role"" on page 15.).
不要因设计处理异常的代码而扰乱了 “正常情况” 的代码。只要可能，你只应该处理正常情况。如果正常情况下的代码失败，你的进程应该尽快报告错误和崩溃。不要试图修复错误并继续。这个错误应该在其他的进程中去处理（请参阅 5.5 "Each process should only have one role"）。

> Clean separation of error recovery code and normal case code should greatly simplify the overall system design.
清晰分离错误恢复的代码和正常情况的代码，将大大简化整个系统的设计。

> The error logs which are generated when a software or hardware error is detected will be used at a later stage to diagnose and correct the error. A permanent record of any information that will be helpful in this process should be kept.
在检测到软件或硬件错误时生成的错误日志，可以用于诊断和纠正错误。对这个过程有帮助的任何信息的记录，都应该被永久的保留。

> 4.2 Identify the error kernel
### 确认错误内核

> One of the basic elements of system design is identifying which part of the system has to be correct and which part of the system does not have to be correct.
系统设计的基本要素之一是确定系统的哪个部分必须是正确的，系统的哪个部分不必是正确的。

> In conventional operating system design the kernel of the system is assumed to be, and must be, correct, whereas all user application programs do not necessarily have to be correct. If a user application program fails this will only concern the application where the failure occurred but should not affect the integrity of the system as a whole.
在传统的操作系统设计中，假定系统的内核是正确的，而所有的用户应用程序不一定是正确的。如果用户应用程序失败，则只会涉及发生故障的应用程序，但不应影响整个系统的完整性。

> The first part of the system design must be to identify that part of the system which must be correct; we call this the error kernel. Often the error kernel has some kind of real-time memory resident data base which stores the state of the hardware.
系统设计的第一部分必须是鉴别系统中必须正确的部分；我们称之为错误内核(error kernel)。错误内核通常有一些实时的内存驻留数据库，用来存储硬件的状态。

> 5 Processes, Servers and Messages
> 5.1 Implement a process in one module
## 5 进程，服务器和消息
### 5.1 仅在单个模块中实现进程

> Code for implementing a single process should be contained in one module. A process can call functions in any library routines but the code for the "top loop" of the process should be contained in a single module. The code for the top loop of a process should not be split into several modules - this would make the flow of control extremely difficult to understand. This does not mean that one should not make use of generic server libraries, these are for helping structuring the control flow.
实现单个的进程的代码应包含在一个模块中。进程可以调用任何库中的函数，但是该进程的 “顶层循环(top loop)” 代码应该包含在单个模块中。进程顶层循环的代码不应该被分成几个模块 —— 这将使控制流程非常难以理解。这并不意味着不应该使用通用服务器库(译注: OTP gen_server 之类的库)，通用服务器库只是为了帮助构建控制流。

> Conversely, code for no more than one kind of process should be implemented in a single module. Modules containing code for several different processes can be extremely difficult to understand. The code for each individual process should be broken out into a separate module.
相反，在一个模块中应该实现不超过一种进程的代码。包含几个不同进程的代码的模块可能极难理解。每个进程的代码应该分解成一个单独的模块。

> 5.2 Use processes for structuring the system
### 5.2 使用进程构建系统

> Processes are the basic system structuring elements. But don't use processes and message passing when a function call can be used instead.
进程是基本的系统结构元素。但是，可以使用函数调用时，不要使用进程和消息传递。

> 5.3 Registered processes
### 注册进程

> Registered processes should be registered with the same name as the module. This makes it easy to find the code for a process.
注册的进程名称应与模块同名。这就能很容易地找到该进程的代码。

> Only register processes that should live a long time.
只应注册能长时间存活的进程。

> 5.4 Assign exactly one parallel process to each true concurrent activity in the system
### 5.4 为系统中每一个真正的并发活动分配一个并行进程

> When deciding whether to implement things using sequential or parallel processes then the structure implied by the intrinsic structure of the problem should be used. The main rule is:
在决定是否使用顺序或并行进程来实现时，应该使用问题内在结构所隐含的方式。主要规则是：

> "Use one parallel process to model each truly concurrent activity in the real world"
“使用一个并行进程来模拟真实世界中的每个真正的并发活动”

> If there is a one-to-one mapping between the number of parallel processes and the number of truly parallel activities in the real world, the program will be easy to understand.
如果在并行进程的数量与现实世界中真正并行活动的数量之间存在一对一的映射关系，程序将会很容易理解。

> 5.5 Each process should only have one "role"
### 5.5 每个进程应该只有一个 “角色”

> Processes can have different roles in the system, for example in the client-server model.
进程在系统中可以有不同的角色，例如在 `client-server` 模型中。

> As far as possible a process should only have one role, i.e. it can be a client or a server but should not combine these roles.
一个进程应尽可能只有一个角色，即，它可以是一个客户端或一个服务器角色，但不应该组合这些角色。

> Other roles which process might have are:
>  Supervisor: watches other processes and restarts them if they fail. 
>  Worker: a normal work process (can have errors). 
>  Trusted Worker: not allowed to have errors.

进程可能具有的其他角色是：
- Supervisor：监视其他进程并在失败时重新启动它们。
- Worker：一个正常的工作过程（可能有错误）。
- Trusted Worker：不允许有错误。

> 5.6 Use generic functions for servers and protocol handlers wherever possible
### 5.6 尽可能使用通用的服务器和协议处理程序

> In many circumstances it is a good idea to use generic server programs such as the generic server implemented in the standard libraries. Consistent use of a small set of generic servers will greatly simplify the total system structure.
在许多情况下，使用通用服务器程序（例如: gen_server）是一个好主意。 一致地使用小部分的通用服务器将大大简化整个系统结构。

> The same is possible for most of the protocol handling software in the system.
系统中的大部分协议处理程序也是尽可能这样去实现的。

> 5.7 Tag messages
### 5.7 标记消息

> All messages should be tagged. This makes the order in the receive statement less important and the implementation of new messages easier.
所有的消息应该被标记。这使得接收语句中的匹配顺序不那么重要，并且更容易扩展实现接收新格式的消息。

> Don't program like this:
不要像这样：

```erl
loop(State) ->
  receive
    ...
    {Mod, Funcs, Args} -> % Don't do this
      apply(Mod, Funcs, Args},
      loop(State);
    ...
  end.
```

> The new message {get_status_info, From, Option} will introduce a conflict if it is placed below the {Mod, Func, Args} message.
如果将新消息的代码 `{get_status_info, From, Option}` 置于 `{Mod, Func, Args}` 消息之后，将引发错误。

> If messages are synchronous, the return message should be tagged with a new atom, describing the returned message. Example: if the incoming message is tagged get_status_info, the returned message could be tagged status_info. One reason for choosing different tags is to make debugging easier.
如果消息是同步的，返回消息应该用一个新的原子标记，来描述返回的消息。 例如：如果传入消息被标记为 `get_status_info`，则返回的消息可能被标记为 `status_info`。 选择不同标签的原因之一是让调试更加容易。

> This is a good solution:
一个好的实现是：

```erl
loop(State) ->
  receive
    ...
    {execute, Mod, Funcs, Args} -> % Use a tagged message.
      apply(Mod, Funcs, Args},
      loop(State);
    {get_status_info, From, Option} ->
      From ! {status_info, get_status_info(Option, State)},
      loop(State);    
    ...
  end.
```

> 5.8 Flush unknown messages
### 5.8 刷新(Flush)未知消息

> Every server should have an Other alternative in at least one receive statement. This is to avoid filling up message queues. Example:
在至少一个接收语句中，每个服务器都应该有另一个通配的选项。这是为了避免填满消息队列。 例：

```erl
main_loop() ->
  receive
    {msg1, Msg1} -> 
      ...,
      main_loop();
    {msg2, Msg2} ->
      ...,
      main_loop();
    Other -> % Flushes the message queue.
      error_logger:error_msg(
          "Error: Process ~w got unknown msg ~w~n.", 
          [self(), Other]),
      main_loop()
  end.
```

> 5.9 Write tail-recursive servers
### 5.9 编写尾递归服务器

> All servers must be tail-recursive, otherwise the server will consume memory until the system runs out of it.
所有的服务器必须是尾递归的，否则服务器将消耗内存，直到耗尽为止。

Don't program like this:

```erl
loop() ->
  receive
    {msg1, Msg1} -> 
      ...,
      loop();
    stop ->
      true;
    Other ->
      error_logger:log({error, {process_got_other, self(), Other}}),
      loop()
  end,
  io:format("Server going down").                % Don't do this! 
                % This is NOT tail-recursive
```

> This is a correct solution:
正确解决方案是：

```erl
loop() ->
  receive
    {msg1, Msg1} -> 
      ...,
      loop();
    stop ->
      io:format("Server going down");
    Other ->
      error_logger:log({error, {process_got_other, self(), Other}}),
      loop()
  end. % This is tail-recursive
```

> If you use some kind of server library, for example generic, you automatically avoid doing this mistake.
如果你使用某种服务器库（例如 gen_server），则会自动地避免出现这种错误。

> 5.10 Interface functions
### 5.10 接口函数

> Use functions for interfaces whenever possible, avoid sending messages directly. Encapsulate message passing into interface functions. There are cases where you can't do this.
尽可能使用接口函数（interface functions），避免直接发送消息。将消息传递的细节封装至接口函数内部。有些情况下你不能这样做。

> The message protocol is internal information and should be hidden to other modules.
消息协议是内部信息，应该隐藏到其他模块。

> Example of interface function:
接口函数示例：

```erl
-module(fileserver).
-export([start/0, stop/0, open_file/1, ...]).

open_file(FileName) ->
  fileserver ! {open_file_request, FileName},
  receive
    {open_file_response, Result} -> Result
  end.

...<code>...
```

> 5.11 Time-outs
### 5.11 超时

> Be careful when using after in receive statements. Make sure that you handle the case when the message arrives later (See "Flush unknown messages" on page 16.).
在消息接收语句中使用 `after` 子句应该小心。当消息到达后，需确保你处理了对应的情况（请参阅 5.8 "Flush unknown messages"）。

> 5.12 Trapping exits
### 5.12 捕获退出

> As few processes as possible should trap exit signals. Processes should either trap exits or they should not. It is usually very bad practice for a process to "toggle" trapping exits.
尽可能少的进程应该捕获退出信号(trap exit)。一个进程应该是（处于）捕获或者不捕获退出信号的。通常在进程运行中来回切换(toggle)捕获行为是非常糟糕的做法。

> 6 Various Erlang Specific Conventions
> 6.1 Use records as the principle data structure
## 6 各种Erlang特定约定
### 6.1 使用记录作为主数据结构

> Use records as the principle data structure. A record is a tagged tuple and was introduced in Erlang version 4.3 and thereafter (see EPK/NP 95:034). It is similar to struct in C or record in Pascal.
使用记录(record)作为主的数据结构。记录是一个标记元组，并在 Erlang/R4.3 版本及其以后的版本中引入（参见 EPK/NP 95:034）。它类似于 C 中的 `struct` 或者 Pascal 中的 `record`。

> If the record is to be used in several modules, its definition should be placed in a header file (with suffix .hrl) that is included from the modules. If the record is only used from within one module, the definition of the record should be in the beginning of the file the module is defined in.
如果要在多个模块中使用记录，则应将其定义放在头文件（后缀为.hrl）中，然后在模块头部引入。如果记录仅在一个模块中使用，那么记录应该在模块文件的开头进行定义。

> The record features of Erlang can be used to ensure cross module consistency of data structures and should therefore be used by interface functions when passing data structures between modules.
Erlang 记录的特性可用于确保数据结构跨模块的一致性，因此，在使用接口函数封装消息传递时，这个消息的结构也应当是这种数据结构（标记元组）。

> 6.2 Use selectors and constructors
### 6.2 使用选择器和构造函数

> Use selectors and constructors provided by the record feature for managing instances of records. Don't use matching that explicitly assumes that the record is a tuple. Example:
使用记录（record）提供的选择器和构造函数来管理记录的实例。不要显示的假定记录是元组，去进行匹配。例：

```erl
demo() ->
  P = #person{name = "Joe", age = 29},
  #person{name = Name1} = P,% Use matching, or...
  Name2 = P#person.name. % use the selector like this.
```

> Don't program like this:
不要像这样编写：

```erl
demo() ->
  P = #person{name = "Joe", age = 29},
  {person, Name, _Age, _Phone, _Misc} = P. % Don't do this
```

> 6.3 Use tagged return values
### 6.3 使用带标记的返回值

> Use tagged return values.
使用被标记的元组作为返回值。

> Don't program like this:
不要像这样编写：

```erl
keysearch(Key, [{Key, Value}|_Tail]) ->
  Value; %% Don't return untagged values!
keysearch(Key, [{_WrongKey, _WrongValue} | Tail]) ->
  keysearch(Key, Tail);
keysearch(Key, []) ->
  false.
```

> Then the {Key, Value} cannot contain the false value. This is the correct solution:
这样的话，参数 `{Key，Value}` 的 `Value` 就不能包含 `false` 的值（这样函数会出现歧义）。而正确的解决方案应该是：

```erl
keysearch(Key, [{Key, Value}|_Tail]) ->
  {value, Value}; %% Correct. Return a tagged value.
keysearch(Key, [{_WrongKey, _WrongValue} | Tail]) ->
  keysearch(Key, Tail);
keysearch(Key, []) ->
  false.
```

> 6.4 Use catch and throw with extreme care
### 6.4 谨慎使用 catch 和 throw

> Do not use catch and throw unless you know exactly what you are doing! Use catch and throw as little as possible.
除非你确切地知道你在做什么，否则不要使用 `catch` 和 `throw`。尽可能少地使用 `catch` 和 `throw`。

> Catch and throw can be useful when the program handles complicated and unreliable input (from the outside world, not from your own reliable program) that may cause errors in many places deeply within the code. One example is a compiler.
当程序处理复杂和不可靠的输入时（来自外部世界，而不是来自你自己可靠的程序）时，`catch` 和 `throw` 可能是有用的，因为这些输入可能在代码中的许多地方引起错误。一个例子就是编译器

> 6.5 Use the process dictionary with extreme care
### 6.5 谨慎使用进程字典

> Do not use get and put etc. unless you know exactly what you are doing! Use get and put etc. as little as possible.
不要使用 `get` 和 `put` 等，除非你确切地知道你在做什么！尽可能少地使用 `get` 和 `put` 等。

> A function that uses the process dictionary can be rewritten by introducing a new argument.
使用进程字典的函数也可以通过引入新的参数来重写。

>Example: 
> Don't program like this:
例如，不要像这样编写：

```erl
tokenize([H|T]) ->
  ...;
tokenize([]) ->
  case get_characters_from_device(get(device)) of % Don't use get/1!
    eof -> [];
    {value, Chars} ->
      tokenize(Chars)
  end.
```

> The correct solution:
正确的方案：

```erl
tokenize(_Device, [H|T]) ->
  ...;
tokenize(Device, []) ->
  case get_characters_from_device(Device) of     % This is better
    eof -> [];
    {value, Chars} ->
      tokenize(Device, Chars)
  end.
```

> The use of get and put will cause a function to behave differently when called with the same input at different occasions. This makes the code hard to read since it is non-deterministic. Debugging will be more complicated since a function using get and put is a function of not only of its argument, but also of the process dictionary. Many of the run time errors in Erlang (for example bad_match) include the arguments to a function, but never the process dictionary.
使用 `get` 和 `put` 会导致一个函数在不同场合调用，有相同的输入时，出现不同的行为。由于它的非确定性，使得代码很难阅读。调试也将变得更复杂，因为使用了 `get` 和 `put`，它不仅是有参数的函数，还是使用了进程字典的函数。Erlang 中的许多运行时错误内容（例如：bad_match）包含了传递给函数的参数，但不包括当前的进程字典状态。

> 6.6 Don't use import
### 6.6 不要使用import

> Don't use -import, using it makes the code harder to read since you cannot directly see in what module a function is defined. Use exref (Cross Reference Tool) to find module dependencies.
不要使用 `-import`，使用它会使代码更难阅读，因为你无法直接看到在哪个模块中定义了这个函数。使用 **exref**（交叉引用工具）来查找模块依赖关系。

> 6.7 Exporting functions
### 6.7 导出函数

> Make a distinction of why a function is exported. A function can be exported for the following reasons (for example):
>  It is a user interface to the module. 
>  It is an interface function for other modules. 
>  It is called from apply, spawn etc. but only from within its module. 
区分为什么要导出一个函数。一个函数需要被导出的原因如下（例如）：
- 它是模块的接口。
- 这是提供给其他模块的接口函数。
- 被`apply`，`spawn`等调用，但只能在其模块内调用。

> Use different -export groupings and comment them accordingly. Example:
使用多个不同的 `-export` 进行分组，并给它们添加相应的注释。 例：

```erl
%% user interface
-export([help/0, start/0, stop/0, info/1]).

%% intermodule exports
-export([make_pid/1, make_pid/3]).
-export([process_abbrevs/0, print_info/5]).

%% exports for use within module only
-export([init/1, info_log_impl/1]).
```

> 7 Specific Lexical and Stylistic Conventions
> 7.1 Don't write deeply nested code
## 7 特定的词汇和文体惯例
### 7.1 不要编写深层嵌套代码

> Nested code is code containing case/if/receive statements within other case/if/receive statements. It is bad programming style to write deeply nested code - the code has a tendency to drift across the page to the right and soon becomes unreadable. Try to limit most of your code to a maximum of two levels of indentation. This can be achieved by dividing the code into shorter functions.
嵌套代码是指 `case/if/receive` 语句包含在其他的 `case/if/receive` 语句内的代码。编写深度嵌套的代码是不好的编程风格 —— 代码倾向于跨页面向右移动，很快变得不可读。尝试将大部分代码限制为最多两个缩进级别。可以通过将代码分成更短的函数来实现。

> 7.2 Don't write very large modules
### 7.2不要编写非常大的模块

> A module should not contain more than 400 lines of source code. It is better to have several small modules than one large one.
一个模块不应该包含超过400行的源代码。最好有几个小模块，而不是一个大模块。

> 7.3 Don't write very long functions
### 7.3 不要写太长的函数

> Don't write functions with more than 15 to 20 lines of code. Split large function into several smaller ones. Don't solve the problem by writing long lines.
不要用超过15到20行的代码编写函数。将大的函数拆分成几个较小的函数。不要通过写很长的一行来解决问题。

> 7.4 Don't write very long lines
### 7.4 不要写很长的行

> Don't write very long lines. A line should not have more than 80 characters. (It will for example fit into an A4 page.)
不要写很长的行，一行不能超过80个字符。(这将例如适合A4页面)

> In Erlang 4.3 and thereafter string constants will be automatically concatenated. Example:
在 Erlang/R4.3 和之后的版本，字符串常量将被自动连接。例：

```erl
io:format("Name: ~s, Age: ~w, Phone: ~w ~n" 
      "Dictionary: ~w.~n", [Name, Age, Phone, Dict])
```

> 7.5 Variable names
### 7.5 变量命名

> Choose meaningful variable names - this is very difficult.
选择有意义的变量名 —— 这是非常困难的。

> If a variable name consists of several words, use "_" or a capitalized letter to separate them. Example: My_variable or MyVariable.
如果一个变量名由几个单词组成，请使用 "_" 或大写字母来分隔它们。例如：`My_variable` 或 `MyVariable`。

> Avoid using "_" as don't care variable, use variables beginning with "_" instead. Example: _Name. If you at a later stage need the value of the variable, you just remove the leading underscore. You will not have problem finding what underscore to replace and the code will be easier to read.
避免使用 "_" 作为不关心变量，而是使用以 "_" 开头的变量。例如：`_Name`。如果稍后阶段需要使用该变量的值，则只需删除前置的下划线。你不会有什么问题就会找到下划线所以替换的变量是什么，且更容易阅读。

> 7.6 Function names
### 7.6 函数命名

> The function name must agree exactly with what the function does. It should return the kind of arguments implied by the function name. It should not surprise the reader. Use conventional names for conventional functions (start, stop, init, main_loop).
函数名称必须与函数完全一致。它应该返回函数名称所暗含的那种参数。读者不应该感到惊讶。常规函数使用常规名称（start，stop，init，main_loop）。

> Functions in different modules that solves the same problem should have the same name. Example: Module:module_info().
解决相同问题的不同模块中的函数应该具有相同的名称。例如：Module：module_info（）。

> Bad function names is one of the most common programming errors - good choice of names is very difficult!
错误的函数名称是最常见的编程错误之一 —— 选择一个好的名字是非常困难的！

> Some kind of naming convention is very useful when writing lots of different functions. For example, the name prefix "is_" could be used to signify that the function in question returns the atom true or false.
在编写大量不同的函数时，某种命名约定是非常有用的。例如，名称前缀 “is_” 可以用来表示提问的函数，并返回原子 `true` 或 `false`。

```erl
is_...() -> true | false
check_...() -> {ok, ...} | {error, ...}
```

> 7.7 Module names
### 7.7 模块命名

> Erlang has a flat module structure (i.e. there are not modules within modules). Often, however, we might like to simulate the effect of a hierarchical module structure. This can be done with sets of related modules having the same module prefix.

Erlang有一个扁平的模块结构（即模块内没有模块）。但是，通常我们可能想要模拟分层模块结构的效果。这可以通过具有相同模块前缀的相关模块组来完成。

> If, for example, an ISDN handler is implemented using five different and related modules. These module should be given names such as:
例如，如果使用五个不同的相关模块来实现ISDN处理程序。 这些模块应该被赋予如下的名字：

isdn_init 
isdn_partb 
isdn_...

> 7.8 Format programs in a consistent manner
### 7.8 以一致的方式格式化程序

> A consistent programming style will help you, and other people, to understand your code. Different people have different styles concerning indentation, usage of spaces etc.
一致的编程风格将帮助你和其他人了解你的代码。不同的人在缩进、空间使用等方面有不同的风格。

> For example you might like to write tuples with a single comma between the elements:
例如，你可能希望在元素之间用逗号来书写元组：

```erl
{12,23,45}
```

> Other people might use a comma followed by a blank:
其他人可能使用逗号，加空格的方式：

```erl
{12, 23, 45}
```

> Once you have adopted style - stick to it.
一旦你采用了某种风格 —— 坚持下去。

> Within a larger project, the same style should be used in all parts.
在一个更大的项目中，所有部分都应该使用相同的风格。

> 8 Documenting Code
> 8.1 Attribute code
## 8 文档化代码
### 8.1 属性代码

> You must always correctly attribute all code in the module header. Say where all ideas contributing to the module came from - if your code was derived from some other code say where you got this code from and who wrote it.
你必须始终正确地将模块头中的所有代码属性化。说出对模块有贡献的所有想法是从哪里来的 —— 如果你的代码是从其他代码派生出来的，就说你从哪里得到了这个代码，谁写的。

> Never steal code - stealing code is taking code from some other module editing it and forgetting to say who wrote the original.
从不窃取代码 —— 窃取代码是指从其他地方获取、编辑模块代码，并忘记表明原件的作者。

> Examples of useful attributes are:
有用属性的示例如下：

```erl
-revision('Revision: 1.14 ').
-created('Date: 1995/01/01 11:21:11 ').
-created_by('eklas@erlang').
-modified('Date: 1995/01/05 13:04:07 ').
-modified_by('mbj@erlang').
```

> 8.2 Provide references in the code to the specifications
### 8.2 在代码中提供规范的引用

> Provide cross references in the code to any documents relevant to the understanding of the code. For example, if the code implements some communication protocol or hardware interface give an exact reference with document and page number to the documents that were used to write the code.
在代码中应提供多种文档的引用，只要是任意与理解代码相关的文档。例如，如果代码实现了一些通信协议或硬件接口，则模块代码中，应将其文档和页码进行明确的引用。

> 8.3 Document all the errors
### 8.3 记录所有错误

> All errors should be listed together with an English description of what they mean in a separate document (See "Error Messages" on page 32.)
所有的错误都应该与一个单独的文档中用英文描述一起列出（参见 10.4 “Error Messages”）。

> By errors we mean errors which have been detected by the system.
错误是指系统能检测到的错误。


> At a point in your program where you detect a logical error call the error logger thus:
在你的程序中检测到逻辑错误的某个时刻，请调用错误记录器（error logger）：

```erl
error_logger:error_msg(Format, {Descriptor, Arg1, Arg2, ....})
```

> And make sure that the line {Descriptor, Arg1,...} is added to the error message documents.
并确保将行 `{Descriptor，Arg1，...}` 添加到错误消息的文档中。

> 8.4 Document all the principle data structures in messages
### 8.4 记录消息中所有的数据结构

> Use tagged tuples as the principle data structure when sending messages between different parts of the system.
在系统的不同部分之间发送消息时，使用标记元组作为主数据结构。

> The record features of Erlang (introduced in Erlang versions 4.3 and thereafter) can be used to ensure cross module consistency of data structures.
Erlang的记录特性（在Erlang/R4.3及以后版本中引入）可用于在跨模块时确保数据结构的一致性。

> An English description of all these data structure should be documented (See "Message Descriptions" on page 32.).
应记录所有这些数据结构的英文说明（请参阅 10.2 “Message Descriptions”）。


> 8.5 Comments
### 8.5 注释

> Comments should be clear and concise and avoid unnecessary wordiness. Make sure that comments are kept up to date with the code. Check that comments add to the understanding of the code. Comments should be written in English.
注释应该清晰简洁，避免不必要的啰嗦。确保注释与代码保持同步。检查添加的注释对代码的理解。注释应该用英文写成。


> Comments about the module shall be without indentation and start with three percent characters (%%%), (See "File Header, description" on page 29.).
关于模块的注释应该没有缩进，并以3个百分号（%%%）开始，（请参见 8.10 “File Header, description”）。


> Comments about a function shall be without indentation and start with two percent characters (%%), (See "Comment each function" on page 27.).
关于函数的注释应该没有缩进，并以2个百分号（%%）开始，（请参见 8.6 “Comment each function”）。


> Comments within Erlang code shall start with one percent character (%). If a line only contains a comment, it shall be indented as Erlang code. This kind of comment shall be placed above the statement it refers to. If the comment can be placed at the same line as the statement, this is preferred.
Erlang 代码中的注释应以1个百分好（％）开始。如果一行只包含一个注释，它应该缩写为 Erlang 代码。这种注释应放在它所指的语句之上。如果注释可以与语句放在同一行，那么这是首选。

```erl
%% Comment about function
some_useful_functions(UsefulArgugument) ->
  another_functions(UsefulArgugument),    % Comment at end of line
  % Comment about complicated_stmnt at the same level of indentation
  complicated_stmnt,
......
```

> 8.6 Comment each function
### 注释每一个函数

> The important things to document are:
>  The purpose of the function. 
>  The domain of valid inputs to the function. That is, data structures of the arguments to the functions together with their meaning. 
>  The domain of the output of the function. That is, all possible data structures of the return value together with their meaning. 
>  If the function implements a complicated algorithm, describe it. 
>  The possible causes of failure and exit signals which may be generated by exit/1, throw/1 or any non-obvious run time errors. Note the difference between failure and returning an error. 
>  Any side effect of the function. 
注释文档中重要的是：
- 函数的目的。
- 该函数的有效输入。也就是说，函数参数的数据结构及其含义。
- 函数输出。 也就是说，返回值所有可能的数据结构及其含义。
- 如果函数实现了一个复杂的算法，请描述它。
- 可能由于 `exit/1`, `throw/1` 或任何非明显的运行时错误，而产生的失败和退出信号的原因。应标注失败和返回错误之间的区别。
- 函数任何的副作用

> Example:
例如：
```erl
%%----------------------------------------------------------------------
%% Function: get_server_statistics/2
%% Purpose: Get various information from a process.
%% Args:   Option is normal|all.
%% Returns: A list of {Key, Value} 
%%     or {error, Reason} (if the process is dead)
%%----------------------------------------------------------------------
get_server_statistics(Option, Pid) when pid(Pid) ->
  ......
```

> 8.7 Data structures
### 8.7 数据结构

> The record should be defined together with a plan text description. Example:
记录应与其描述一起定义。 例：

```erl
%% File: my_data_structures.h

%%---------------------------------------------------------------------
%% Data Type: person
%% where:
%%    name: A string (default is undefined).
%%    age: An integer (default is undefined).
%%    phone: A list of integers (default is []).
%%    dict:     A dictionary containing various information about the person. 
%%       A {Key, Value} list (default is the empty list).
%%----------------------------------------------------------------------
-record(person, {name, age, phone = [], dict = []}).
```

> 8.8 File headers, copyright
### 8.8 文件头，版权

> Each file of source code must start with copyright information, for example:
每个源代码文件都必须以版权信息开始，例如：

```erl
%%%--------------------------------------------------------------------- 
%%% Copyright Ericsson Telecom AB 1996
%%%
%%% All rights reserved. No part of this computer programs(s) may be 
%%% used, reproduced,stored in any retrieval system, or transmitted,
%%% in any form or by any means, electronic, mechanical, photocopying,
%%% recording, or otherwise without prior written permission of 
%%% Ericsson Telecom AB.
%%%--------------------------------------------------------------------- 
```

> 8.9 File headers, revision history
### 8.9 文件头，修订历史

> Each file of source code must be documented with its revision history which shows who has been working with the files and what they have done to it.
源代码的每个文件都必须记录其修订历史记录，以显示谁在使用这些文件以及他们对这些文件所做的工作。

```erl
%%%--------------------------------------------------------------------- 
%%% Revision History
%%%--------------------------------------------------------------------- 
%%% Rev PA1 Date 960230 Author Fred Bloggs (ETXXXXX)
%%% Intitial pre release. Functions for adding and deleting foobars
%%% are incomplete
%%%--------------------------------------------------------------------- 
%%% Rev A Date 960230 Author Johanna Johansson (ETXYYY)
%%% Added functions for adding and deleting foobars and changed 
%%% data structures of foobars to allow for the needs of the Baz
%%% signalling system
%%%--------------------------------------------------------------------- 
```

> 8.10 File Header, description
### 8.10 文件头，描述

> Each file must start with a short description of the module contained in the file and a brief description of all exported functions.
每个文件必须以其所包含模块的简短描述，以及所有导出函数的简要描述开始。

```erl
%%%--------------------------------------------------------------------- 
%%% Description module foobar_data_manipulation
%%%--------------------------------------------------------------------- 
%%% Foobars are the basic elements in the Baz signalling system. The
%%% functions below are for manipulating that data of foobars and for
%%% etc etc etc
%%%--------------------------------------------------------------------- 
%%% Exports
%%%--------------------------------------------------------------------- 
%%% create_foobar(Parent, Type)
%%%   returns a new foobar object
%%%   etc etc etc
%%%--------------------------------------------------------------------- 
```

> If you know of any weakness, bugs, badly tested features, make a note of them in a special comment, don't try to hide them. If any part of the module is incomplete, add a special comment. Add comments about anything which will be of help to future maintainers of the module.If the product of which the module you are writing is a success, it may still be changed and improved in ten years time by someone you may never meet.
如果你知道任何弱点，错误，经测试为坏的功能，请在特别注释中记下它们，不要试图隐藏它们。如果模块的任何部分不完整，请添加特殊注释。如果你正在编写的模块的产品是成功的，那么在十年之后，你可能永远也不会遇到这样的问题。

> 8.11 Do not comment out old code - remove it
### 8.11 不要注释旧代码 - 删除它

> Add a comment in the revision history to that effect. Remember the source code control system will help you!
应在修订历史记录（版本管理系统）中添加一条记录。记住源代码控制系统会帮助你！

> 8.12 Use a source code control system
### 8.12 使用源代码控制系统

> All non trivial projects must use a source code control system such as RCS, CVS or Clearcase to keep track of all modules.
所有非平凡的项目都必须使用 RCS，CVS 或 Clearcase 等源代码控制系统来跟踪所有模块。

> 9 The Most Common Mistakes:
## 9 最常见的错误

> Writing functions which span many pages (See "Don't write very long functions" on page 23.). 
> Writing functions with deeply nested if's receive's, case's etc (See "Don't write deeply nested code" on page 23.). 
> Writing badly typed functions (See "Use tagged return values" on page 19.). 
> Function names which do not reflect what the functions do (See "Function names" on page 24.). 
> Variable names which are meaningless (See "Variable names" on page 23.). 
> Using processes when they are not needed (See "Assign exactly one parallel process to each true concurrent - activity in the system" on page 14.). 
> Badly chosen data structures (Bad representations). 
> Bad comments or no comments at all (always document arguments and return value). 
> Unindented code. 
> Using put/get (See "Use the process dictionary with extreme care" on page 20.). 
> No control of the message queues (See "Flush unknown messages" on page 16. and See "Time-outs" on page 18.). 

- 编写多页的函数（参见 7.3 "Don't write very long functions"）
- 编写多层嵌套的函数，如 `receive`, `case`等（参见 7.1 "Don't write deeply nested code"）
- 编写格式错误的函数（参见 6.3 "Use tagged return values"）
- 函数名不反映函数的功能（参见 7.6 "Function names"）
- 变量名是没有意义的（参见 7.5 "Variable names"）
- 在不需要的时候使用进程（参见 5.4 "Assign exactly one parallel process to each true concurrent activity in the system"）
- 错误的选择数据结构（坏的表示）
- 错误注释或根本没有注释（总是记录参数和返回值）
- 不缩进的代码
- 使用 `put/get`（参见 6.5 "Use the process dictionary with extreme care"）
- 不控制消息队列（参见 5.8 "Flush unknown messages" 并看看 5.11 "Time-outs"）

> 10 Required Documents
## 10 必要文档

> This section describes some of the system level documents which are necessary for designing and maintaining system programmed using Erlang.
本节介绍一些使用 Erlang 编程和维护系统所需的系统级文档。

> 10.1 Module Descriptions
### 10.1 模块描述

> One chapter per module. Contains description of each module, and all exported functions as follows:
>  the meaning and data structures of the arguments to the functions 
>  the meaning and data structure of the return value. 
>  the purpose of the function 
>  the possible causes of failure and exit signals which may be generated by explicit calls to exit/1. 
每个模块一个章节。包含每个模块的说明，以及所有导出的功能如下：
- 函数参数的含义和数据结构
- 返回值的含义和数据结构
- 函数的目的
- 可能失败和和显式调用 `exit/1` 产生退出信号的原因

> Format of document to be defined later:
稍后定义的文件格式： ...

> 10.2 Message Descriptions
### 10.2 消息描述

> The format of all inter-process messages except those defined inside one module.
除了在一个模块内定义的，其他所有进程间消息的格式（都应该被记录到文档）。

> Format of document to be defined later:
稍后定义的文件格式：

> 10.3 Process
### 10.3 进程

> Description of all registered servers in the system and their interface and purpose.
系统中所有注册的服务器及其接口和用途的说明。

> Description of the dynamic processes and their interfaces.
动态（创建的）进程及其接口的描述。

> Format of document to be defined later:
稍后定义的文件格式：

> 10.4 Error Messages
### 10.4 错误消息

> Description of error messages
错误消息的描述

> Format of document to be defined later:
稍后定义的文件格式：

**Updated: 2000-02-08**
原文更新于: 2000-02-08
