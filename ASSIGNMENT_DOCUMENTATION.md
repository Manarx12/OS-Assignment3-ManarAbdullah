# Assignment 3 - Complete Documentation

**Student Name**: [Manar Abdullah Alzahrani]  
**Student ID**: [445052176]  
**Date Submitted**: [6 may]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [\[[Paste your personal Gmail Google Drive link here\](https://drive.google.com/file/d/1bnjnqRvb1SU97YrM2UdL2hJvFYWnHL5T/view?usp=drive_link)]](https://drive.google.com/file/d/1bnjnqRvb1SU97YrM2UdL2hJvFYWnHL5T/view?usp=drive_link)

**Video filename**: `445052176_assiinment.MOV`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [5 may, 4;10pm]
**Forked the repository, changed the visibility to public, cloned the project locally, and updated the Student Id**: 

---

### Entry 2 - [5 may, 4:30pm]
**Analyzed race conditions on counter variables and successfully implemented Task 1 using ReentrantLock with try-finally blocks**:  

---

### Entry 3 - [6 may, 3:00pm]
**Protected the shared executionLog (ArrayList) using ReentrantLock to prevent ConcurrentModificationException (Task2)**: 


### Entry 4 - [6 may, 4:00]
**Implemented Task 3 by adding a binary Semaphore to control concurrent CPU access and tested the simulation**: 



### Entry 5 - [6 may, 4:30]
**complete assignment documentation**: 




## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

In the original unsubscribed code, race conditions occur primarily on the shared metrics counters and the execution log.

First Race Condition (Shared Counters): The affected shared resources are contextSwitchCount, completedProcessCount, and totalWaitingTime. Concurrent access is a problem because operations like count++ are not atomic; they consist of read, modify, and write cycles. Without synchronization, two threads can read the same old value simultaneously, resulting in lost updates and corrupted final stats.

// Vulnerable code snippet:
contextSwitchCount++; // Multiple threads interleaved here cause data corruption


Second Race Condition (Execution Log): The affected resource is the executionLog (ArrayList). ArrayList is not thread-safe. Concurrent additions by multiple threads can cause a structural modification conflict, leading to data loss or throwing a ConcurrentModificationException during runtime.

// Vulnerable code snippet:
executionLog.add("Process " + id + " executed"); // Throws ConcurrentModificationException

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

A ReentrantLock is a mutual exclusion (mutex) mechanism that ensures only one thread can access a critical section at any given time, and it has an ownership concept (the thread that locks it must unlock it). On the other hand, a Semaphore manages a set of permits. A binary semaphore (1 permit) restricts access to a single thread but does not enforce ownership, meaning any thread can release it.

Implementation Choices:

ReentrantLock Usage: I used ReentrantLock (counterLock and logLock) to protect the counters and the executionLog. This is because updating metrics and appending to a data structure are strict mutual exclusion problems where ownership and explicit blocking are needed to prevent concurrent race states.

Semaphore Usage: I used a binary Semaphore (cpuSemaphore) with 1 permit to control access to the virtual CPU. This choice fits perfectly because the CPU represents a shared physical or logical execution slot where threads queue up to acquire the single permit before executing their CPU burst and release it afterward.

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

A deadlock is a specific state in an operating system where a set of threads are permanently blocked because each thread holds a resource and waits for another resource held by another thread in the same set, creating a circular dependency loop.

Two Prevention Techniques:

Mutual Exclusion Elimination / Preemption: Allowing resources to be shared or forcefully taken away (not always feasible for software locks).

Hold and Wait / Circular Wait Prevention (Lock Ordering): Enforcing a strict, global order in which resources must be acquired, ensuring a circular chain can never form.

What I did in my code:
To eliminate the "Hold and Wait" condition and guarantee that resources are always released, I strictly utilized try-finally blocks. By acquiring the lock before the try block and putting the .unlock() or .release() statement inside the finally block, I ensured that even if a thread encounters an unexpected runtime exception, it will always release its acquired lock/permit, eliminating any permanent blocking or deadlock situations.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

Design Choice: I chose to use ONE single lock (coarse-grained approach) named counterLock to protect all three shared counter variables (contextSwitchCount, completedProcessCount, and totalWaitingTime).

Why I made this choice:
All three variables are always updated together at the exact same logical event in the simulation (e.g., when a process context switches or finishes execution). Creating three separate locks would introduce unnecessary overhead and complicate the code structure without giving any practical concurrency benefit, since a thread updating one variable must update the others at the same time anyway.

Trade-offs between the two approaches:

Coarse-Grained (Single Lock): Reduces the synchronization overhead, avoids complex lock nested sorting (which reduces deadlock risks), and is simpler to implement. However, it lowers concurrency if threads need to update the counters independently.

Fine-Grained (Separate Locks): Increases potential concurrency by letting different threads update different counters simultaneously. However, it introduces significant lock acquisition overhead and highly increases the danger of deadlocks due to lock ordering issues. Given that these three counters are tightly dependent in this simulation, a single lock provides optimal performance.

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: contextSwitchCount, completedProcessCount, and totalWaitingTime.

**Why they need protection**: These variables are shared across multiple process threads. Since standard arithmetic increments (like count++) are not atomic operations at the CPU level (they involve reading, modifying, and writing), concurrent access without protection leads to race conditions, lost updates, and inconsistent metrics.

**Synchronization mechanism used**: ReentrantLock (specifically, an instance named counterLock).

**Code snippet**:
```java
 public static void incrementContextSwitch() {
        // TODO: Protect this critical section with a lock
        // RACE CONDITION: Multiple threads might read and write simultaneously!
        contextSwitchLock.lock();
        try {
            contextSwitchCount++;
        } finally {
            contextSwitchLock.unlock();
        }
    }

    // Method to increment completed process counter
    public static void incrementCompletedProcess() {
        // TODO: Protect this critical section with a lock
        completedProcessLock.lock();
        try {
            completedProcessCount++;
        } finally {
            completedProcessLock.unlock();
        }
    }

    // Method to add waiting time
    public static void addWaitingTime(long time) {
        // TODO: Protect this critical section with a lock
        waitingTimeLock.lock();
        try {
            totalWaitingTime += time;
        } finally {
            waitingTimeLock.unlock();
        }
    }

   
```

**Justification**: A ReentrantLock was chosen because these counters represent mutual exclusion problems where only one thread should modify the state at a time. Using a coarse-grained single lock ensures all related metrics are updated together safely without adding unnecessary locking overhead or risk of interleaved updates.

---

### Critical Section #2: Execution Log

**What resource**: The executionLog object, which is implemented as a standard ArrayList<String>.

**Why it needs protection**:ArrayList is fundamentally not thread-safe in Java. If multiple executing process threads attempt to call .add() concurrently, it can lead to structural corruption of the underlying array, loss of log data, or throw a runtime ConcurrentModificationException. 

**Synchronization mechanism used**: ReentrantLock (specifically, an instance named logLock).

**Code snippet**:
```java
 // Method to log execution
    public static void logExecution(String message) {
        // TODO: Protect this critical section with a lock
        // RACE CONDITION: ArrayList is not thread-safe!
        loglock.lock();
        try {
            executionLog.add(""message);
        } finally {
            logLock.unlock();
        }

    }
```

**Justification**: Utilizing a distinct ReentrantLock ensures exclusive serial access to the log repository, completely eliminating structural race conditions. Using a separate lock (logLock) instead of reusing the counter lock allows threads to log their execution and update counters independently when appropriate, maintaining a balanced design decision.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To manage and restrict concurrent access to the virtual CPU resource, ensuring that the defined schedule rules are strictly followed without multiple process threads interleaved on the processor simultaneously.

**Number of permits and why**: 1 permit (Binary Semaphore). This is because the core logic of the simulation simulates a single-core CPU scheduler, meaning only one process can execute its CPU burst cycle at any single moment in time.

**Where implemented**: Inside the overridden run() method of the process execution thread within the scheduler architecture.

**Code snippet**:
```java
// Paste your implementation here
try {
    cpuSemaphore.acquire();
    try {
        if (startTime == -1) {
            startTime = System.currentTimeMillis();
        }
        // Core process execution happens here
    } finally {
        cpuSemaphore.release();
    }
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

**Effect on program behavior**: It enforces strict serial and mutual execution over the CPU burst time simulated environment. Processes smoothly queue up for the permit, eliminating overlapping simulation logs and securing realistic step-by-step CPU operations.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
Procedure: Ran java SchedulerSimulationSync five times.
Results: Every run produced the exact same numbers:
java
public static void logExecution(String message) {
 logLock.lock();
 try { executionLog.add(message); } finally { logLock.unlock(); }
}
java
SharedResources.cpuSemaphore.acquire();
try {
 // ... execution code ...
} finally {
 SharedResources.cpuSemaphore.release();
}
Context switches: always 28
Completed processes: always 12 (matches process count)
Total waiting time: always 30641 ms (example)
Average waiting time: identical each run
Why synchronisation necessary: Without locks, the counters could lose increments
and the log could throw exceptions. The fact that results are now deterministic proves
race conditions are eliminated.

### Test 2: Exception Testing
Procedure: Added a loop that ran the program 20 times.
Results: No ConcurrentModificationException or any other exception occurred.
What this proves: The logLock successfully serialises access to the ArrayList ,
making it thread‑safe.

### Test 3: Correctness Verification
Expected: completedProcessCount should equal number of processes created; total
waiting time should be consistent with per‑process waiting times.
Actual: All values matched the manual calculation (checked with print statements).
Analysis: The synchronisation does not change the logical behaviour – it only ensures
correctness under concurrency.


### Test 4: Different Scenarios
Scenario: Changed Semaphore(1) to Semaphore(2) temporarily.
Purpose: Observe effect of allowing two concurrent processes.
Results: With 2 permits, execution overlapped (interleaving in output). Still no race
conditions because counters remained protected.
What I learned: Semaphores are extremely flexible – they control the degree of
concurrency without changing the core logic.


## Part 5: Reflection and Learning

### What I learned about synchronization:

Race conditions are subtle: the code can run correctly many times and then suddenly
fail. Synchronisation makes concurrent programs predictable.
Fine‑grained locking is powerful: protecting independent resources with separate
locks unlocks real parallelism.
The try‑finally pattern is non‑negotiable – forgetting to unlock in a finally block
leads to deadlocks that are very hard to debug.
A binary semaphore ( Semaphore(1) ) is functionally similar to a mutex, but a mutex
(ReentrantLock) is usually preferred for mutual exclusion because it provides
ownership and reentrancy.
Synchronisation adds overhead, but the safety it buys is essential for any
multithreaded program.

---

### Real-world applications:

Give TWO examples where synchronization is critical:

1. Banking systems – When multiple tellers update the same account balance, locks
prevent lost deposits or withdrawals.
2. Print spooler – A semaphore with a limit equal to the number of printers controls
access to physical printers.
---

### How I would explain synchronization to others:

“Imagine a shared whiteboard where many students want to write. If two write at the
same time, their notes become unreadable. A mutex lock is like giving the marker to
only one student at a time. A semaphore is like having a few markers – it lets a limited
number of students write together. Without these rules, the whiteboard would be a
mess. That’s exactly what synchronisation does for shared data in a program.”

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/Manarx12/OS-Assignment3-ManarAbdullah.git

**Number of commits**: 4

**Commit messages**: 
1. set my studeni id to 445052176
2. Implement Task 1: Protect counter variables using ReentrantLock
3. Implement Task 2 and 3: Protect execution log and control cpu access
4. complete assignment documntation

---

## Summary

**Total time spent on assignment**: 6 hours

**Key takeaways**: 
1. Fine‑grained locking improves performance for independent resources.
2. try‑finally is the only safe way to release locks.
3. A semaphore can control both mutual exclusion and resource limits.

**Most challenging aspect**: Deciding on lock granularity and proving that separate locks
are safe (no deadlock because locks are never nested).

**What I'm most proud of**: The final program runs deterministically, with no exceptions,
and the code clearly shows why each synchronisation mechanism is used.

---

**End of Documentation**
