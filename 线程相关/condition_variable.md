# condition_variable条件变量

A condition variable is an object able to block the calling thread until notified to resume.

条件变量是一个对象，该对象可以阻塞调用线程，直到被通知恢复。

It uses a unique_lock (over a mutex) to lock the thread when one of its wait functions is called. The thread remains blocked until woken up by another thread that calls a notification function on the same condition_variable object.

当调用其wait 函数之一时，它使用unique_lock（over a mutex）来锁定线程。 该线程将保持阻塞状态，直到被另一个在同一condition_variable对象上调用通知功能的线程唤醒为止。

Objects of type condition_variable always use unique_lock<mutex> to wait: for an alternative that works with any kind of lockable type, see condition_variable_any.

condition_variable类型的对象始终使用unique_lock <mutex>等待：有关可与任何类型的可锁定类型一起使用的替代方法，请参见condition_variable_any。

注： 用`std::unique_lock`而不使用`std::lock_guard`的原因——等待中的线程必须在等待期间解锁互斥量，并对互斥量再次上锁，而`std::lock_guard`没有这么灵活。

## 1.wait函数

- 1.`void wait (unique_lock<mutex>& lck);`

  当前线程（应已锁定lck的互斥对象）的执行被阻塞，直到得到通知。

  在阻塞线程的时刻，该函数自动调用lck.unlock（），从而允许其他锁定的线程继续执行。

  一旦得到通知（明确地由其他线程通知），该函数将取消阻塞并调用lck.lock（），使lck处于与调用该函数时相同的状态。然后函数返回（注意，最后的互斥锁可能会在返回之前再次阻塞线程）。

  通常，通过另一个线程调用成员函数`notify_one()`或`notify_all()`来通知该函数唤醒。但是某些实现可能会产生虚假的唤醒调用，而不会调用这些函数中的任何一个。因此，使用此功能的用户应确保满足其恢复条件。

- 2.`template <class Predicate>
    void wait (unique_lock<mutex>& lck, Predicate pred);`

  如果指定了pred（可以是Lambda表达式也可以是bool类型的函数），则该函数仅在pred返回false时才阻塞，并且通知只能在线程变为true时才取消阻塞线程（这对于检查伪唤醒调用特别有用）。此版本的行为就像是实现为：

  ```c++
  while (!pred()) wait(lck)
  ```

  调用wait()的过程中，在互斥量锁定时，可能会去检查条件变量若干次，当提供测试条件的函数返回true就会立即返回。当等待线程重新获取互斥量并检查条件变量时，并非直接响应另一个线程的通知，就是所谓的***伪唤醒***(spurious wakeup)。因为任何伪唤醒的数量和频率都是不确定的，所以不建议使用有副作用的函数做条件检查。 

如果任何参数的值对该函数无效（例如，调用线程未锁定lck的互斥对象），则它将导致未定义的行为。

- 3.`template <class Rep, class Period>
    cv_status wait_for (unique_lock<mutex>& lck,
                        const chrono::duration<Rep,Period>& rel_time);`

  当前线程（应已锁定lck的互斥对象）的执行在rel_time期间被阻止，或者直到被通知为止（如果后者先发生）。

  在阻塞线程时，该函数会自动调用lck.unlock（），从而允许其他锁定的线程继续执行。

  **一旦收到通知或rel_time过去，函数将取消阻塞并调用lck.lock（），使lck处于与调用函数时相同的状态**。 然后函数返回（注意，最后的互斥锁可能会在返回之前再次阻塞线程）。

  通常，通过另一个线程调用成员函数`notify_one()`或`notify_all()`来通知该函数唤醒。但是某些实现可能会产生虚假的唤醒调用，而不会调用这些函数中的任何一个。因此，使用此功能的用户应确保满足其恢复条件。

- 4.`template <class Rep, class Period, class Predicate>
         bool wait_for (unique_lock<mutex>& lck,
              const chrono::duration<Rep,Period>& rel_time, Predicate pred);`

  如果指定了pred，则该函数仅在pred返回false时才阻塞，并且通知只能在线程变为true时才取消阻塞线程（这对于检查虚假唤醒调用特别有用）。 它的行为就像实现为：

  `return wait_until (lck, chrono::steady_clock::now() + rel_time, std::move(pred));`

注：lck是一个unique_lock对象，其互斥对象当前已被该线程锁定。该对象的所有等待成员函数的所有并发调用均应使用相同的基础互斥对象（如lck.mutex（）返回）。 rel_time 是线程将阻塞等待通知的最长时间。持续时间（duration）是代表特定相对时间的对象。pred 是可调用的对象或函数，不带任何参数，并返回可以作为布尔值评估的值。反复调用它，直到评估为true。

- 5.`template <class Clock, class Duration>
    cv_status wait_until (unique_lock<mutex>& lck,
                          const chrono::time_point<Clock,Duration>& abs_time);`

  当前线程（应已锁定lck的互斥锁）的执行被阻塞，直到被通知或直到abs_time为止，以先到者为准。

  在阻塞线程时，该函数会自动调用lck.unlock（），从而允许其他锁定的线程继续执行。

  收到通知或到达abs_time后，该函数将取消阻塞并调用lck.lock（），使lck处于与调用该函数时相同的状态。 然后函数返回（注意，最后的互斥锁可能会在返回之前再次阻塞线程）。

  通常，通过另一个线程调用成员函数`notify_one()`或`notify_all()`来通知该函数唤醒。但是某些实现可能会产生虚假的唤醒调用，而不会调用这些函数中的任何一个。因此，使用此功能的用户应确保满足其恢复条件。

- 6.`template <class Clock, class Duration, class Predicate>
         bool wait_until (unique_lock<mutex>& lck,
                          const chrono::time_point<Clock,Duration>& abs_time,
                          Predicate pred);`

  如果指定了pred，则该函数仅在pred返回false时才阻塞，并且通知只能在线程变为true时才取消阻塞线程（这对于检查虚假唤醒调用特别有用）。 它的行为就像实现为：

  ```c++
  while (!pred())
    if ( wait_until(lck,abs_time) == cv_status::timeout)
      return pred();
  return true;
  ```

  注：abs_time是线程将停止阻塞的时间点，以允许函数返回。time_point是代表特定绝对时间的对象。

  该函数执行三个原子操作：

  - lck的初始解锁并同时进入等待状态。
  - 解除等待状态的阻塞。

  返回之前将lck锁定。
  对象上的原子操作是根据单个总顺序进行排序的，此函数中的三个原子操作以与上述相同的相对方式发生。

## 2.notify函数

- 7.`void notify_one() noexcept;`

  不再阻塞当前正在等待此条件的线程之一。

  如果没有线程在等待，则该函数不执行任何操作。

  如果超过一个，则未指定选择哪个线程。

  **无数据竞争，无异常抛出**。

- 8.`void notify_all() noexcept;`

  不再阻塞当前正在等待此条件的所有线程（所有等待此条件的线程执行顺序无法确定)。

  如果没有线程在等待，则该函数不执行任何操作。

  **无数据竞争，无异常抛出**。