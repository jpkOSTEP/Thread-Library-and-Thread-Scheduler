# 🧵 Thread Library and Scheduler

A threading library implemented in C, featuring support for cooperative threading, mutexes, and multiple scheduling policies: Round Robin (RR) and Multilevel Feedback Queue (MLFQ).

## 📦 Key Components

### `TCB` (Thread Control Block)
Defined in `worker.h`, this structure maintains metadata about each thread:
- `worker_t id`: Unique thread ID
- `thread_status status`: READY / RUNNING / BLOCKED / FINISHED
- `ucontext_t* context`: Execution context
- `char* stack`: Thread stack
- `uint priority`: Scheduling priority
- `worker_t waiting_thread`: Dependent thread
- `struct timeval created, first_run, finished`: Timestamps
- `double time_on_cpu`: CPU time used
- `int mutex_blocked`: Mutex blocking flag

### `worker_mutex_t` (User-Level Mutex)
Custom mutex implementation with:
- `int is_locked`
- `worker_t owner`
- `queue* wait_queue`

---

## ⚙️ Core Functions

### Thread Lifecycle
- `worker_create()`: Initializes and schedules new threads.
- `worker_yield()`: Voluntarily yields CPU to the scheduler.
- `worker_exit(void* retval)`: Terminates a thread and returns a value.
- `worker_join(worker_t thread, void** retval)`: Waits for a thread to finish.

### Synchronization
- `worker_mutex_init(worker_mutex_t* mutex)`
- `worker_mutex_lock(worker_mutex_t* mutex)`
- `worker_mutex_unlock(worker_mutex_t* mutex)`
- `worker_mutex_destroy(worker_mutex_t* mutex)`

---

## 🧠 Scheduling Policies

### ⏱️ Round Robin (RR)
- Single ready queue.
- Time-sliced scheduling.
- Threads re-added to the queue after each quantum.

### 🔀 Multilevel Feedback Queue (MLFQ)
- Multiple queues (`MLFQ_LEVELS`), each with increasing time slices.
- Threads are demoted if they exceed their time quantum.
- Priority boost occurs every `PRIORITY_BOOST_CYCLES` to prevent starvation.

---

## ⏲️ Timer and Signals

- `ITIMER_PROF` is used for preemptive scheduling.
- `SIGPROF` handler switches context to the scheduler.
- Tracks average turnaround and response times.

---

## 📊 Benchmarks

Tested on:
- `external_cal.c`
- `parallel_cal.c`
- `vector_multiply.c`

For each, performance was measured with 25 threads under both RR and MLFQ schedulers. Metrics include:
- Turnaround Time
- Response Time
- CPU Time

---

## 🛠️ Build Instructions

```bash
make  # Builds the thread library
```

- No external libraries required. Compiled in **32-bit** mode (`-m32`).