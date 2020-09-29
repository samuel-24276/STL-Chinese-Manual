# thread线程类

A thread of execution is a sequence of instructions that can be executed concurrently with other such sequences in *multithreading* environments, while sharing a same address space.

执行线程是指令序列，可以在多线程环境中与其他此类序列同时执行，同时共享相同的地址空间。

An initialized [thread](http://www.cplusplus.com/thread) object represents an active thread of execution; Such a [thread](http://www.cplusplus.com/thread) object is *[joinable](http://www.cplusplus.com/thread::joinable)*, and has a unique *[thread id](http://www.cplusplus.com/thread::get_id)*.

初始化的线程对象表示活动的执行线程； 这样的线程对象是可连接的，并且具有唯一的线程ID。

A default-constructed (non-initialized) [thread](http://www.cplusplus.com/thread) object is *not [joinable](http://www.cplusplus.com/thread::joinable)*, and its *[thread id](http://www.cplusplus.com/thread::get_id)* is common for all *non-[joinable](http://www.cplusplus.com/thread::joinable)* threads.

默认构造（未初始化）的线程对象不可连接，并且其线程ID对于所有不可连接的线程都是通用的。

A *[joinable](http://www.cplusplus.com/thread::joinable)* thread becomes *not [joinable](http://www.cplusplus.com/thread::joinable)* if *[moved from](http://www.cplusplus.com/thread::operator=)*, or if either [join](http://www.cplusplus.com/thread::join) or [detach](http://www.cplusplus.com/thread::detach) are called on them.

如果从中移出可连接线程，或者如果在它们上调用了join或detach，则该线程将不可连接。

## 1.Member types

- [**id**](http://www.cplusplus.com/reference/thread/thread/id/) Thread id (public member type )

  该类型的值由thread :: get_id和this_thread :: get_id返回以标识线程。

  默认构造的thread :: id对象的值标识不可连接的线程，因此，其值等于任何此类线程的成员thread :: get_id返回的值。

  对于可连接线程，thread :: get_id返回此类型的唯一值，该值与其他任何可连接或不可连接线程返回的值不相等。

  请注意，某些库实现可能会重新利用无法再加入的终止线程的thread :: id值。

- [**native_handle_type**](http://www.cplusplus.com/reference/thread/thread/native_handle_type/) Native handle type (public member type )

  如果库实现支持，则该成员类型仅出现在类线程中。

  它是thread :: native_handle返回的类型，具有有关线程的特定于实现的信息。

## 2.Member functions

- [**(constructor)**](http://www.cplusplus.com/reference/thread/thread/thread/) 构造一个线程对象

   - 1.`thread() noexcept; `默认构造函数 
     构造一个不代表任何执行线程的线程对象。
   - 2.`template <class Fn, class... Args>
     explicit thread (Fn&& fn, Args&&... args);` 初始化构造函数
     构造一个代表新的可连接执行线程的线程对象。
     新的执行线程调用fn传递args作为参数（使用其左值或右值引用的衰减副本）。
     该构造的完成与该fn副本的调用开始同步。
   - 3.`thread (const thread&) = delete;` 复制构造函数
     **删除的构造函数形式（无法复制线程对象）**。
   - 4.`thread (thread&& x) noexcept;` 移动构造函数
     构造一个线程对象，该对象获取x表示的执行线程（如果有）。 此操作不会以任何方式影响已移动线程的执行，它只是传输其处理程序。
     x对象不再代表任何执行线程。

  注：一，fn为指向函数的指针，指向成员的指针或任何类型的可移动构造的函数对象（即，其类定义了operator（）的对象，包括闭包和函数对象）。返回值（如果有）将被忽略；二，args是传递给fn调用的参数（如果有）。 它们的类型应是可移动构造的。如果fn是成员指针，则第一个参数应是为其定义了该成员的对象（或其引用或指向它的指针）；三，x是线程对象，其状态已移至构造对象。

- [**(destructor)**](http://www.cplusplus.com/reference/thread/thread/~thread/) 销毁一个线程对象

  如果线程在销毁时是可连接的，则调用terminate（）。不会抛出异常以保证线程安全。

- std::thread [**operator=**](http://www.cplusplus.com/reference/thread/thread/operator=/) () 赋值运算符

  - 1.`thread& operator= (thread&& rhs) noexcept;`

    如果对象当前不可连接，则它将获取由rhs表示的执行线程（如果有）。

    如果它是可连接的，则调用Terminate（）。

    调用之后，rhs不再代表任何执行线程（就像默认构造的一样）。

  - 2.`thread& operator= (const thread&) = delete;`

-  thread::id  [**get_id**](http://www.cplusplus.com/reference/thread/thread/get_id/) ()

  返回线程ID。

  如果线程对象是可连接的，则该函数返回一个唯一标识线程的值。

  如果线程对象不可连接，则该函数返回成员类型为thread :: id的默认构造的对象。

- bool [joinable](http://www.cplusplus.com/reference/thread/thread/joinable/)()

  返回线程对象是否可连接。

  如果线程对象表示执行线程，则该对象是可连接的。

  在以下任何情况下，线程对象均不可连接：

  - 如果它是默认构造的。
  - 如果已将其移出（构造另一个线程对象或对其进行分配）。
  - 如果已调用其成员join或detach。

- void [**join**](http://www.cplusplus.com/reference/thread/thread/join/) () 线程连接

  线程执行完成后，该函数返回。

  这使该函数返回的时刻与线程中所有操作的完成同步：阻塞调用该函数的线程的执行，直到构造函数上调用的函数返回为止（如果尚未返回）。

  调用此函数后，线程对象变得不可连接，并且可以安全地销毁。

  **注意**：如果此成员函数引发异常，则线程对象将保持有效状态。如果调用失败，将引发system_error异常：

  | 异常类型                                              | error condition                                              | 描述                                                         |
  | ----------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | [system_error](http://www.cplusplus.com/system_error) | [errc::invalid_argument                                      | 线程对象不可连接                                             |
  | [system_error](http://www.cplusplus.com/system_error) | [errc::no_such_process](http://www.cplusplus.com/errc)       | 线程对象无效                                                 |
  | [system_error](http://www.cplusplus.com/system_error) | [errc::resource_deadlock_would_occur](http://www.cplusplus.com/errc) | 当前线程与尝试加入的线程相同，或者-检测到死锁（实现可能检测到某些死锁情况）。 |

- void [**detach**](http://www.cplusplus.com/reference/thread/thread/detach/)() 线程分离

  从调用线程中分离对象所代表的线程，从而使它们彼此独立执行。

  两个线程继续运行，而不会阻塞或以任何方式进行同步。 请注意，当任何一个执行结束时，其资源将被释放。

  调用此函数后，线程对象变得不可连接，并且可以安全地销毁。

  **注意**：如果此成员函数引发异常，则线程对象将保持有效状态。如果调用失败，将引发system_error异常：

  | exception type                                        | error condition                                         | description      |
  | ----------------------------------------------------- | ------------------------------------------------------- | ---------------- |
  | [system_error](http://www.cplusplus.com/system_error) | [errc::invalid_argument](http://www.cplusplus.com/errc) | 线程对象不可连接 |
  | [system_error](http://www.cplusplus.com/system_error) | [errc::no_such_process](http://www.cplusplus.com/errc)  | 线程对象无效     |

- `void swap (thread& x) noexcept;`

  用x交换线程对象this的状态。不会抛出异常。

- native_handle_type [**native_handle**](http://www.cplusplus.com/reference/thread/thread/native_handle/) () 

  如果库实现支持，则该成员函数仅存在于类线程中。

  如果存在，它将返回一个值，该值用于访问与线程关联的特定于实现的信息。

- `static unsigned hardware_concurrency() noexcept;`

   返回并发线程的数量。例如，多核系统中，返回值可以是CPU核芯的数量。 

  此值的解释是特定于系统和实现的，可能不准确，而只是一个近似值。

  请注意，这不需要与系统中可用的处理器或内核的实际数量相匹配：系统可以为每个处理单元支持多个线程，或限制对程序的资源访问。

  如果此值不可计算或定义不正确，则该函数返回0。

## 3.Non-member overloads

- `void swap (thread& x, thread& y) noexcept;`

  交换线程对象x和y的状态。

  这是`void swap (thread& x) noexcept;`的重载，其行为就像调用了x.swap（y）一样。

























