### Threading and Schedulers in Reactor

In Reactor, threading and concurrency are handled with flexibility, leaving the decision of how and where code executes to the developer. Unlike traditional models where a dedicated thread is automatically assigned, Reactor does not enforce any specific threading model. However, it provides mechanisms through **Schedulers** to control concurrency effectively. Here’s a breakdown of key concepts:

### 1. **Concurrency-Agnostic Nature**
- Reactor doesn’t assign threads automatically to a `Mono` or `Flux` by default.
- Unless explicitly stated, operations continue running on the same thread as the one that initiated the `subscribe()` call.
- Example:
  ```java
  Mono<String> mono = Mono.just("hello");
  Thread t = new Thread(() -> 
      mono.map(msg -> msg + "thread")
          .subscribe(v -> System.out.println(v + Thread.currentThread().getName()))
  );
  t.start();
  t.join();
  ```
  - In this example, `Mono` runs on a new thread (`Thread-0`), as `subscribe()` happens inside a separate thread.

### 2. **Schedulers: The Execution Context Providers**
Schedulers are abstractions similar to `ExecutorService`, allowing you to control where and how Reactor operators are executed. Reactor provides several types of Schedulers:

- **`Schedulers.immediate()`**: Executes tasks on the current thread (no execution context).
- **`Schedulers.single()`**: Reuses the same thread for all operations until the `Scheduler` is disposed.
- **`Schedulers.boundedElastic()`**: Uses a bounded elastic thread pool for blocking tasks, especially for I/O operations. It limits the number of threads to avoid overwhelming the system.
  - Two implementations:
    1. **ExecutorService-based**: Uses platform threads.
    2. **Thread-per-task-based**: Uses VirtualThreads (Java 21+).
- **`Schedulers.parallel()`**: Creates a fixed number of threads, equal to the number of CPU cores, for parallel work.
- **`Schedulers.fromExecutorService()`**: Allows using a custom `ExecutorService` as a Scheduler.

### 3. **Switching Execution Contexts: `publishOn` vs. `subscribeOn`**
- **`subscribeOn(Scheduler)`**: Changes the thread on which the subscription (starting point of the chain) happens. Placement in the chain doesn’t matter.
  - This operator affects **all operations** downstream, as it defines the thread on which the subscription is executed.
- **`publishOn(Scheduler)`**: Switches the execution context (thread) for all subsequent operators in the chain. Placement of this operator is crucial since it affects all operations after its invocation.

### 4. **Schedulers in Action**
- Schedulers give control over how reactive chains are executed, optimizing performance and resource usage.
- Example: `Flux.interval(Duration.ofMillis(300), Schedulers.newSingle("test"))` uses a custom Scheduler to handle intervals in a dedicated thread.

### 5. **Threading in Practice**
- By default, Reactor works sequentially on the calling thread unless a `Scheduler` is introduced.
- For blocking operations, `Schedulers.boundedElastic()` is often the recommended approach to prevent blocking the main threads or CPU-bound operations.
- Misusing blocking APIs (`block()`, `blockFirst()`, etc.) on inappropriate schedulers (e.g., `Schedulers.single()` or `parallel()`) can lead to exceptions like `IllegalStateException`.

### 6. **Custom Schedulers**
- You can create custom schedulers or wrap existing `ExecutorService` or `Executor` instances using `Schedulers.fromExecutorService()`.

This flexible model allows you to optimize thread usage based on the needs of your application, controlling how blocking and non-blocking tasks are handled, and preventing bottlenecks or resource overuse.

**In Reactor, when you chain operators, you can wrap as many Flux and Mono implementations inside one another as you need. 
Once you subscribe, a chain of Subscriber objects is created, backward (up the chain) to the first publisher. 
This is effectively hidden from you. All you can see is the outer layer of Flux (or Mono) and Subscription, but these intermediate operator-specific subscribers are where the real work happens. 
[here](https://spring.io/blog/2019/03/06/flight-of-the-flux-1-assembly-vs-subscription)**
