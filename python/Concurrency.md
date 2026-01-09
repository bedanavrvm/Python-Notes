# Concurrency

Concurrency is the ability of a program to make progress on multiple tasks
in an overlapping way.

- **Concurrency** is about *structure* (handling more than one thing at a
  time).
- **Parallelism** is about *execution* (doing more than one thing at the
  same time on multiple CPU cores).

In Python, the best concurrency tool depends on whether your workload is:

- **I/O-bound**: waiting on network, disk, databases, subprocess output.
- **CPU-bound**: heavy computation in Python (parsing, simulation, math).

## The Global Interpreter Lock (GIL)

CPython (the standard Python implementation) has a **Global Interpreter
Lock (GIL)**: a mutex that allows only one thread to execute Python bytecode
at a time.

### Why the GIL exists

- **Simpler memory management**: reference counting is easier with a single
  thread mutating object refcounts at a time.
- **C-extension compatibility**: many native extensions assume certain thread
  safety properties.

### Impact of the GIL

- **Threads do not speed up CPU-bound Python code**.
- **Threads can still help I/O-bound code**, because while waiting on I/O the
  interpreter can switch to another thread.
- Some C-extensions (and some built-in operations) may temporarily
  **release the GIL** while doing work in C, which is why you might
  occasionally see speedups in threaded numeric workloads.

## Threading vs. Multiprocessing vs. Asyncio

### Threading (shared memory)

Threads share the same memory space.

- **Best for**: I/O-bound tasks.
- **Pros**: low overhead, easy sharing of data.
- **Cons**: race conditions are easy to introduce, and CPU-bound Python is
  still limited by the GIL.

### Multiprocessing (separate memory)

Each process has its own Python interpreter and memory space.

- **Best for**: CPU-bound tasks.
- **Pros**: true parallelism across CPU cores.
- **Cons**: higher overhead; data must be serialized (pickled) to move
  between processes.

On Windows, multiprocessing uses the **spawn** start method by default.
That means you must protect process-starting code with:

```python
if __name__ == "__main__":
    ...
```

### Asyncio (single-threaded cooperative concurrency)

`asyncio` provides concurrency using an **event loop** and **coroutines**.
It is especially good when you have many concurrent I/O operations.

- **Best for**: scalable I/O (many sockets, many requests, web servers).
- **Pros**: very low overhead per task compared to threads.
- **Cons**: CPU-heavy work blocks the event loop unless moved to a thread or
  process.

## Context switches

A **context switch** is when the operating system scheduler pauses one
thread/process and resumes another. Context switches are normal, but they
have overhead:

- Saving/restoring CPU state
- Cache effects

## Threading in Practice

In modern Python, prefer `concurrent.futures.ThreadPoolExecutor` for most
thread pooling.

An **executor** manages a pool of worker threads (or processes) and gives
you a higher-level API than manually creating `Thread`/`Process` objects.

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def io_task(i: int) -> str:
    time.sleep(0.5)
    return f"done {i}"

def main() -> None:
    with ThreadPoolExecutor(max_workers=5) as pool:
        futures = [pool.submit(io_task, i) for i in range(10)]
        for fut in as_completed(futures):
            print(fut.result())

if __name__ == "__main__":
    main()
```

### Shared state and race conditions

When multiple threads read/write shared data without coordination, you can get
a **race condition**.

Use synchronization primitives such as:

- `threading.Lock` (mutual exclusion)
- `threading.RLock` (re-entrant lock)
- `threading.Semaphore` (limit concurrency)
- `queue.Queue` (thread-safe message passing)

```python
import threading

counter = 0
lock = threading.Lock()

def increment() -> None:
    global counter
    for _ in range(100_000):
        with lock:
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(4)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(counter)
```

A **deadlock** happens when two (or more) threads wait on locks in a cycle.
To reduce deadlocks:

- Acquire locks in a consistent order.
- Keep lock-held sections small.
- Prefer message passing (`queue.Queue`) when possible.

## Multiprocessing in Practice

Prefer `concurrent.futures.ProcessPoolExecutor` for simple CPU-bound
parallelism.

```python
from concurrent.futures import ProcessPoolExecutor

def cpu_task(n: int) -> int:
    total = 0
    for i in range(n):
        total += (i * i) % 97
    return total

def main() -> None:
    work = [500_000, 500_000, 500_000, 500_000]
    with ProcessPoolExecutor() as pool:
        results = list(pool.map(cpu_task, work))
    print(results)

if __name__ == "__main__":
    main()
```

### Serialization (pickling) and IPC

Processes do not share memory by default. Communication often uses:

- **IPC** via `multiprocessing.Queue` / `Pipe`
- **Shared memory** via `multiprocessing.shared_memory`
- File-based communication (slower, but sometimes simplest)

If something cannot be pickled (for example, open file handles, some
closures, lambdas), you may get errors when passing it to a process pool.

## Asyncio in Practice

In `asyncio`, an `async def` function defines a **coroutine**.

- You use `await` to pause until an awaitable completes.
- The event loop switches to other ready tasks at `await` points.

```python
import asyncio

async def fetch_data(i: int) -> dict:
    await asyncio.sleep(0.5)
    return {"id": i}

async def main() -> None:
    results = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3),
    )
    print(results)

asyncio.run(main())
```

### Tasks, cancellation, and timeouts

A **task** is a scheduled coroutine.

```python
import asyncio

async def slow() -> None:
    await asyncio.sleep(10)

async def main() -> None:
    task = asyncio.create_task(slow())
    try:
        await asyncio.wait_for(task, timeout=0.5)
    except asyncio.TimeoutError:
        task.cancel()

asyncio.run(main())
```

### Async synchronization

`asyncio` has its own primitives for coordination:

- `asyncio.Lock`
- `asyncio.Semaphore`
- `asyncio.Queue` (useful for backpressure in producer/consumer pipelines)

## Choosing the Right Tool

| Method | Best for | Parallel? | Typical overhead |
|---|---|---:|---:|
| Threading | I/O-bound (network/disk) | No (GIL) | Low |
| Multiprocessing | CPU-bound (pure Python) | Yes | High |
| Asyncio | Lots of I/O at scale | No (single thread) | Very low |

## Summary

- Use **threads** for I/O-bound concurrency and shared-memory coordination.
- Use **processes** for CPU-bound parallelism.
- Use **asyncio** for high-scale I/O with an event-loop model.
- Be mindful of **race conditions**, **deadlocks**, and (on Windows)
  `if __name__ == "__main__":` when using multiprocessing.

## Important Keywords

### **Concurrency**

Running multiple tasks in overlapping time, even if not truly simultaneous.

### **Parallelism**

Running tasks simultaneously, usually on multiple CPU cores.

### **I/O-bound**

Work dominated by waiting on external resources (network/disk).

### **CPU-bound**

Work dominated by computation (especially in Python code).

### **Global Interpreter Lock (GIL)**

CPython mutex that allows only one thread to execute Python bytecode at a
time.

### **Thread**

Lightweight unit of execution within a process; threads share memory.

### **Process**

Independent program execution context with its own memory and interpreter.

### **Context Switch**

Switching the CPU from running one thread/process to another.

### **Race Condition**

Bug caused by unsafely accessing shared state from multiple threads.

### **Deadlock**

Situation where tasks wait on each other indefinitely (often due to locks).

### **Lock / Mutex**

Synchronization primitive that enforces mutual exclusion for shared data.

### **Semaphore**

Synchronization primitive that limits concurrent access to a resource.

### **Queue (Message Passing)**

Thread/process-safe structure for producer/consumer workflows.

### **Executor**

High-level API (`concurrent.futures`) that runs callables in threads or
processes.

### **Future**

Handle representing an asynchronous result (thread/process pool work).

### **Asyncio**

Python library for asynchronous I/O using `async`/`await` and an event loop.

### **Event Loop**

Scheduler that runs tasks and resumes them when awaited operations complete.

### **Coroutine**

Function defined with `async def` that can be paused/resumed with `await`.

### **Task**

A scheduled coroutine managed by the asyncio event loop.

### **await**

Keyword that suspends a coroutine until an awaitable completes.

### **asyncio.gather**

Utility to run multiple awaitables concurrently and collect results.

### **Cancellation**

Stopping an asyncio task (usually via `task.cancel()`).

### **Timeout**

Limiting how long to wait for an operation (e.g., `asyncio.wait_for`).

### **Pickling / Serialization**

Converting Python objects to bytes so they can be sent to other processes.

### **IPC (Inter-Process Communication)**

Communication between processes (queues, pipes, shared memory).
