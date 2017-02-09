---
title: Lock implementation in operation system
date: 2017-02-09 23:25:35
tags:
---
Take a look at pthread_mutex_t

``` c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_lock(&mutex);
balance = balance + 1;
pthread_mutex_unlock(&mutex);
```
Locks(mutex) are widely used in concurrent programming.

Are you curious about how OS implement it? Efficient locks provide mutual exclusion at low cost. What hardware support is needed? What OS support?

Some points:
- mutual exclusion
- fairness
- performance

# Approach 0: Interrupts
``` c
void lock() {
    DisableInterrupts();
}
void unlock() {
    EnableInterrupts();
}
```
## pros
- simplicity.


## cons
- monopolize cpu, no priority schedules
- OS can not regain control
- does not work in multiprocessors
- miss interrupts, for example, disk read finish
- inefficient


# Approach 1: Test and Set (Atomic Exchange), Compare and Swap
The simplest bit of hardware support to understand is what is known
as a test-and-set instruction, also known as atomic exchange. On x86 architecture, it is the xchg instrution. In following section, we define what test-and-set does with C code snippet:
``` c
int test_and_set(int* old_ptr, int new_val) {
    int old_val = *old_ptr;
    *old_ptr = new_val;
    return old_val; // store new value and return old value
}

typedef struct __lock_t {
    int flag;
} lock_t;

void init(lock_t* lock) {
    lock->flag = 0;
}

void lock(lock_t* lock) {
    while (test_and_set(&lock->flag, 1) == 1)
        ; // spin-wait, do nothing
}

void unlock(lock_t* lock) {
    lock->flag = 0;
}

```
Similar approach would be compare-and-exchange(x86), C pseudocode for this single instruction is shown below.
``` c
int compare_and_swap(int* ptr, int expected, int new_val) {
    int actual = *ptr;
    if (actual == expected) {
        *ptr = new_val;
    }
    return actual;
}
void lock(lock_t* lock) {
    while(compare_and_swap(&lock->flag, 0, 1) ==  1)
        ; //spin wait
}
```
*ptr is initialed as 0. First thread lock successfully get 0(actual) equals to 0(expected), and set actual equals to 1.
When other threads enter critial section, 1(actual) is not equal to 0(expected) and return 1, threads will spin wait until unlock which set actual to 0. 

## pros
- simple and correct
## cons
- no fairness
- poor performance. Consider multiple CPUs, thread A(CPU1) grabs lock, and then thread B trys to. B will spin-wait on CPU2, wasting computation.


# Approach 2: Yield
It is a simple approach to release spin-wait CPUs. Yield is a method which could be called by threads when they want to give up the CPU and let another thread run.
``` c
void lock() {
    while (test_and_set(&flag, 1) == 1)
        yield(); // release CPU to other threads
}
```
## pros
- better performance
## cons
- still no fairness, not tackled starvation problem

# Approach 3: Using queues, sleeping instead of spinning
``` c
typedef struct __lock_t {
    int flag;
    int guard;
    queue_t* q;
} lock_t;

void init(lock_t* lock) {
    lock->flag = 0;
    lock->guard = 0;
    queue_init(lock->q);
}

void lock(lock_t* lock) {
    while (test_and_set(&lock->guard, 1) == 1)
        ; // guard lock for grab lock, get old value and set new value(1)
    if (lock->flag == 0) {
        lock->flag = 1;
        lock->guard = 0;
    }
    else {
        queue_add(lock->q, gettid()); // put thread id into queue
        lock->guard = 0;
        park();// release CPU, wake up here
    }
    
}

void unlock(lock_t* lock) {
    while (test_and_set(lock->guard, 1) == 1)
        ;
    if (queue_empty(lock->q) {
        lock->flag = 0;
    }
    else {
        unpark(lock->q); //hold lock, and wake up next thread
    }
    lock->guard = 0;
}

```
## pros
- correctness, schedulable
- good performance




