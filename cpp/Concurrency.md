# Concurrency

C++11 introduced standard threading support.

## Threads (`std::thread`)

```cpp
#include <thread>

void worker(int id) {
    cout << "Thread " << id << " running\n";
}

int main() {
    std::thread t1(worker, 1);
    std::thread t2(worker, 2);

    t1.join(); // Wait for t1 to finish
    t2.join();
    return 0;
}
```

## Mutexes and Locks

Prevent data races with `std::mutex`. Use `std::lock_guard` for RAII-style locking (never forget to unlock).

```cpp
#include <mutex>

std::mutex mtx;
int shared_resource = 0;

void safe_increment() {
    std::lock_guard<std::mutex> lock(mtx); // Locks here
    shared_resource++;
    // Unlocks automatically when 'lock' is destroyed
}
```

## Async Tasks (`std::async`)

Higher-level abstraction for running tasks and getting results via `std::future`.

```cpp
#include <future>

int compute() { return 42; }

int main() {
    std::future<int> result = std::async(compute);
    cout << result.get(); // Blocks until result is ready
    return 0;
}
```
