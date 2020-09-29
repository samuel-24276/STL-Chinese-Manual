# chrono日期和时间库

chrono是header的名称，也是子命名空间的名称：该标头中的所有元素（common_type特殊化除外）不是直接在std名称空间下定义（像大多数标准库一样），而是 std::chrono 。

此header中的元素处理时间。这主要通过三个概念来完成：

- Durations 持续时间
  它们测量时间跨度，例如：1分钟，2小时或10毫秒。
  在此库中，它们用持续时间类模板的对象表示，该对象将计数表示形式和周期精度结合在一起（例如，十毫秒具有十个计数表示形式，而毫秒则作为周期精度）。
- Time points 时间点
  对特定时间点的参考，例如生日，今天的黎明或下一趟火车何时通过。
  在此库中，time_point类模板的对象通过使用相对于纪元的持续时间（这是使用同一时钟的所有time_point对象共有的固定时间点）来表示这一点。
- Clocks 钟表
  将时间点与实际物理时间相关联的框架。
  该库提供至少三个时钟，这些时钟提供了将当前时间表示为time_point的方式：system_clock，steady_clock和high_resolution_clock。

## 1.Classes

### 1.1.duration时间间隔——类模板

```c++
template <class Rep, class Period = ratio<1>>
class duration {
public:
  using rep = Rep;
  using period = Period;
public:
  // 构造/复制/销毁
  constexpr duration() = default;
  template <class Rep2>
  constexpr explicit duration(const Rep2& r);
  template <class Rep2, class Period2>
  constexpr duration(const duration<Rep2, Period2>& d);
  ~duration() = default;
  duration(const duration&) = default;
  duration& operator=(const duration&) = default;
  // 探察函数
  constexpr rep count() const;
  // 特殊值
  static constexpr duration zero();
  static constexpr duration min();
  static constexpr duration max();
};
```

duration 由 Rep 类型的计次数和计次周期组成，其中计次周期是一个编译期有理数常量，表示从一个计次到下一个的秒数。
存储于 duration 的数据仅有 Rep 类型的计次数。若 Rep 是浮点数，则 duration 能表示小数的计次数。 Period 被包含为时长类型的一部分，且只在不同时长间转换时使用。 

在内部，该对象将计数存储为成员类型rep（第一个模板参数Rep的别名）的对象，可以通过调用成员函数count来检索该对象。

此计数以周期表示。 周期的长度通过其第二个模板参数（Period）集成到类型中（在编译时），该参数是一种比率类型，表示每个周期中经过的秒数（或分数）。

- 1.Template parameters

  Rep是算术类型或模拟算术类型的类，将用作内部计数的类型。

  Period 是一种比率类型，以秒为单位表示时间段。

- 2.Template instantiations模板实例化

  在此命名空间中还定义了以下方便的持续时间实例化typedef：

  | **type**     | **Representation**       | **Period**                                            |
  | ------------ | ------------------------ | ----------------------------------------------------- |
  | hours        | 至少23位的有符号整数类型 | [ratio](http://www.cplusplus.com/ratio)<3600,1>       |
  | minutes      | 至少29位的有符号整数类型 | [ratio](http://www.cplusplus.com/ratio)<60,1>         |
  | seconds      | 至少35位的有符号整数类型 | [ratio](http://www.cplusplus.com/ratio)<1,1>          |
  | milliseconds | 至少45位的有符号整数类型 | [ratio](http://www.cplusplus.com/ratio)<1,1000>       |
  | microseconds | 至少55位的有符号整数类型 | [ratio](http://www.cplusplus.com/ratio)<1,1000000>    |
  | nanoseconds  | 至少64位的有符号整数类型 | [ratio](http://www.cplusplus.com/ratio)<1,1000000000> |

- 3.Member types

  以下别名是持续时间的成员类型。 它们被成员函数广泛用作参数和返回类型：

  | member type | definition   | notes                            |
  | ----------- | ------------ | -------------------------------- |
  | `rep`       | 第一模板参数 | 表示类型用作内部计数对象的类型。 |
  | `period`    | 第二模板参数 | 比率类型，以秒为单位。           |

- 4.Member functions

  `构造和析构函数省略不写。`

  `constexpr rep count() const;`

  返回持续时间对象的内部计数（即表示值）。

  返回的值是用类的时间间隔表示的内部表示形式的当前值，不一定是秒。

- 5.static member functions

  `static constexpr duration zero();`返回持续时间值为零。

  `static constexpr duration min();`返回持续时间的最小值。

  `static constexpr duration max();`返回持续时间的最大值。

- 6.Non-member functions

  太多了，省略

### 1.2.time_point时间中的一个点——类模板

```c++
template <class Clock, class Duration = typename Clock::duration>
class time_point {
public:
  using clock = Clock;
  using duration = Duration;
  using rep = typename duration::rep;
  using period = typename duration::period;
public:
  // 构造
  constexpr time_point(); // 拥有纪元值
  constexpr explicit time_point(const duration& d); // 同 time_point() + d
  template <class Duration2>
  constexpr time_point(const time_point<clock, Duration2>& t);
  // 探察函数
  constexpr duration time_since_epoch() const;
  // 算术
  constexpr time_point& operator+=(const duration& d);
  constexpr time_point& operator-=(const duration& d);
  // 特殊值
  static constexpr time_point min();
  static constexpr time_point max();
};
```

time_point 被实现成如同存储一个 Duration 类型的自 Clock 的纪元起始开始的时间间隔的值。 

在内部，对象存储一个持续时间类型的对象，并使用Clock类型作为其时代的参考。

- 1.Template parameters

  Clock是时钟类别，例如system_clock，steady_clock，high_resolution_clock或自定义时钟类别。

  Duration 是持续时间类型。

- 2.Member types

  以下别名是time_point的成员类型。 它们被成员函数广泛用作参数和返回类型：

  | **member type** | **definition**   | **notes**                                                    |
  | --------------- | ---------------- | ------------------------------------------------------------ |
  | clock           | 第一模板参数     | 时钟类别（system_clock，steady_clock，high_resolution_clock或自定义时钟类别）。 |
  | duration        | 第二模板参数     | 用于表示时间点的持续时间类型。                               |
  | rep             | duration::rep    | Type returned by duration::count.                            |
  | period          | duration::period | The [ratio](http://www.cplusplus.com/ratio) type that represents the length of a *period* in seconds. |

- 3.Member functions

  `构造函数省略不写`

  `operators()省略不写`

  `duration time_since_epoch() const;`

  返回一个持续时间对象，其时间跨度值介于纪元和时间点之间。

  返回的值是内部持续时间对象的当前值。

- 4.static member functions

  `static constexpr time_point min();`

  返回time_point的最小值。该函数调用duration :: min来构造一个对象，使其内部持续时间对象的最小值尽可能小。

  `static constexpr time_point max();`

  返回time_point的最大值。该函数调用duration :: max来构造一个对象，该对象的内部持续时间对象可能具有最大值。

### 1.3.duration_cast函数模板

```c++
template <class ToDuration, class Rep, class Period>
  constexpr ToDuration duration_cast (const duration<Rep,Period>& dtn);
```

考虑到时段的差异，将dtn的值转换为其他持续时间类型。

该函数不使用隐式转换。 相反，所有计数值都在内部转换为最宽的表示形式（内部计数类型为common_type），然后转换为目标类型，所有转换均通过static_cast显式完成。

如果目标类型的精度较低，则该值将被截断。

示例代码：

```c++
// duration_cast
#include <iostream>     // std::cout
#include <chrono>       // std::chrono::seconds, std::chrono::milliseconds
                        // std::chrono::duration_cast

int main ()
{
  std::chrono::seconds s (1);             // 1 second
  std::chrono::milliseconds ms = std::chrono::duration_cast<std::chrono::milliseconds> (s);

  ms += std::chrono::milliseconds(2500);  // 2500 millisecond

  s = std::chrono::duration_cast<std::chrono::seconds> (ms);   // truncated

  std::cout << "ms: " << ms.count() << std::endl;
  std::cout << "s: " << s.count() << std::endl;

  return 0;
}
```

最终输出为：ms: 3500 s: 3，	ms转换为s会有精度损失。

### 1.4.1.8.time_point_cast函数模板

```c++
template <class ToDuration, class Clock, class Duration>
  time_point<Clock,ToDuration> time_point_cast (const time_point<Clock,Duration>& tp);
```

考虑到持续时间周期之间的差异，将tp的值转换为具有不同持续时间内部对象的time_point类型。

该函数使用duration_cast来转换内部持续时间对象。

请注意，函数的第一个模板参数不是返回类型，而是其持续时间分量。

示例代码：

```c++
// time_point_cast
#include <iostream>
#include <ratio>
#include <chrono>

int main ()
{
  using namespace std::chrono;
  typedef duration<int,std::ratio<60*60*24>> days_type;

  time_point<system_clock,days_type> today = time_point_cast<days_type>(system_clock::now());

  std::cout << today.time_since_epoch().count() << " days since epoch" << std::endl;

  return 0;
}
```

输出为：15490 days since epoch

### 1.5..system_clock类

```c++
class system_clock {
public:
  using rep = /* 见说明 */ ;
  using period = ratio</* 未指明 */, /* 未指明 */ >;
  using duration = chrono::duration<rep, period>;
  using time_point = chrono::time_point<system_clock>;
  static constexpr bool is_steady = /* 未指明 */ ;
  static time_point now() noexcept;
  // 以上是Clock必须满足的要求
  static time_t to_time_t (const time_point& t) noexcept;
  static time_point from_time_t(time_t t) noexcept;
};
```

时钟类提供对当前time_point的访问。

具体来说，system_clock是系统范围的实时时钟。

- 1.Clock properties

   **realtime** 它旨在表示实时时间，因此可以以某种方式在日历表示形式之间进行转换（请参见to_time_t和from_time_t成员函数）。

   **signed count**  它的time_point值可以引用纪元之前的时间（带有负值）。

   **system-wide**  系统上运行的所有进程都应使用此时钟来检索相同的time_point值。

- 2.Member types

  以下别名是steady_clock的成员类型：

  | member type | **definition**           | **notes**                    |
  | ----------- | ------------------------ | ---------------------------- |
  | rep         | 算术类型（或模拟它的类） | 用于存储期间计数             |
  | period      | 比率类型                 | 表示周期的长度（以秒为单位） |
  | duration    | duration<rep,period>     | 时钟的持续时间类型。         |
  | time_point  | time_point<steady_clock> | 时钟的时间点类型。           |

- 3.Member constants成员常量

  is_steady      true

- 4.Static member functions

  `static time_point now() noexcept;`

  返回system_clock框架中的当前time_point。

  `static time_t to_time_t (const time_point& tp) noexcept;`

  将tp转换为time_t类型的等效项。

  `static time_point from_time_t (time_t t) noexcept;`

  将t转换为其等效的成员类型time_point。

### 1.6.steady_clock类

时钟类提供对当前time_point的访问。

steady_clock是专门设计用于计算时间间隔的。

- 1.Clock properties

   **monotonic** 它的成员 now() 永远不会返回比上一个调用更低的值。

  **steady** 时钟每跳一次，就花费相同的时间量（就物理时间而言）。

- 2.Member types

  以下别名是steady_clock的成员类型：

  | member type | **definition**           | **notes**                    |
  | ----------- | ------------------------ | ---------------------------- |
  | rep         | 算术类型（或模拟它的类） | 用于存储期间计数             |
  | period      | 比率类型                 | 表示周期的长度（以秒为单位） |
  | duration    | duration<rep,period>     | 时钟的持续时间类型。         |
  | time_point  | time_point<steady_clock> | 时钟的时间点类型。           |

- 3.Member constants成员常量

  is_steady      true

- 4.Static member functions

  `static time_point now() noexcept;`

  返回steady_clock帧中的当前time_point。

### 1.7.high_resolution_clock类

时钟类的成员提供对当前time_point的访问。

high_resolution_clock是最短滴答周期的时钟。 它可能是system_clock或stable_clock的同义词。

- Clock properties

   **highest precision** 这是精度最高的时钟类型。

其他属性、成员函数等与steady_clock相同。

## 2.Class instantiation typedefs

### 2.1.hours 类

```c++
typedef duration < /*see rep below*/, ratio<3600,1> > hours
```

### 2.2.minutes  类

```c++
typedef duration < /*see rep below*/, ratio<60,1> > minutes;
```

### 2.3.seconds类

```c++
typedef duration < /*see rep below*/ > seconds;
```

### 2.4. milliseconds类

```c++
typedef duration < /* see rep below */, milli > milliseconds;
```

### 2.5.microseconds类

```c++
typedef duration < /* see rep below */, micro > microseconds;
```

### 2.6.nanoseconds  类

```c++
typedef duration < /* see rep below */, nano > nanoseconds;
```