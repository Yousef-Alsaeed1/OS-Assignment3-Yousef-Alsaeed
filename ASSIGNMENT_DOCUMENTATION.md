# Assignment 3 - Complete Documentation

**Student Name**: [Yousef Alsaeed]  
**Student ID**: [445050012]  
**Date Submitted**: [May 2 2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [[Paste your personal Gmail Google Drive link here](https://drive.google.com/file/d/14VTKSgBS1SQ93E_9FHTe9F-qQ1W_Tw1l/view?usp=sharing)]

**Video filename**: `445050012_Assignment3_Synchronization`

**Verification**:
- [x] Link is accessible (tested in incognito mode)
- [x] Video is 3-5 minutes long
- [x] Video shows code walkthrough and commits
- [x] Video has clear audio
- [x] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [May 2, 2026, 9:00 AM]
**What I implemented**: forked the repository - cloned it in VS Code - and set my student ID to 445050012

**Challenges encountered**: had to make the repository public and rename it correctly

**How I solved it**: followed the README steps to change visibility in GitHub Settings.

**Testing approach**: Verified the repository appeared public on GitHub.

**Time spent**: 15 minutes

---

### Entry 2 - [May 2, 2026, 9:30 AM]
**What I implemented**: 
Task 1 - Added ReentrantLock (counterLock) to protect the three shared counter methods: incrementContextSwitch() - incrementCompletedProcess() - and addWaitingTime()

**Challenges encountered**: Understanding where exactly to place the lock and unlock calls.

**How I solved it**: Followed the README pattern

**Testing approach**: Compiled the code and ran it to check for errors.

**Time spent**:  25 minutes 

---

### Entry 3 - [May 2, 2026, 10:00 AM]
**What I implemented**: Task 2 - Added a separate logLock ReentrantLock to protect the executionLog ArrayList in the logExecution() method

**Challenges encountered**: Understanding why ArrayList needs its own separate lock instead of reusing counterLock

**How I solved it**: Realized that the log is an independent resource from the counters so a separate lock avoids unnecessary contention

**Testing approach**: Ran the program multiple times and confirmed no ConcurrentModificationException occurred


**Time spent**: 10 minutes

---

### Entry 4 - [May 2, 2026, 10:30 AM]
**What I implemented**: Task 3 - Added a binary Semaphore (cpuSemaphore) with 1 permit to control CPU access in both run() and runToCompletion() methods.

**Challenges encountered**: i had a proplem with the (try-catch-finally) structure  I placed catch after finally which caused a error.

**How I solved it**: fixed the structure so that acquire() is in the outer try, the process execution is in the inner try, release() is in the inner finally and InterruptedException is caught in the outer catch.


**Testing approach**: Compiled and ran the program multiple times to verify consistent results.

**Time spent**: 45 minutes

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[Your answer here - 4-6 sentences with code examples]

**Race Condition 1 - contextSwitchCount counter**: The variable contextSwitchCount is shared among all threads. The operation contextSwitchCount++ is not atomic it involves three steps: read, modify, and write. If two threads execute this simultaneously, both may read the same value, both increment it, and both write back the same result, causing one increment to be lost. For example, if the value is 5 and two threads read it at the same time, both write 6 instead of the correct value 7.


**Race Condition 2 - executionLog ArrayList**: The ArrayList is not thread safe. If multiple threads call executionLog.add() at the same time, the internal structure of the ArrayList can become corrupted, leading to a ConcurrentModificationException or lost log entries. This is because ArrayList resizes and shifts elements internally, which is not safe when done by multiple threads simultaneously.

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[Your answer here - explain your implementation choices]

A ReentrantLock is a mutual exclusion lock — only one thread can hold it at a time, and it is either locked or unlocked. I used ReentrantLock for counterLock (protecting the three counters) and logLock (protecting the ArrayList), because these resources should only be modified by one thread at a time to prevent data corruption.


A Semaphore maintains a count of permits and allows a specified number of threads to access a resource concurrently. I used a binary Semaphore (Semaphore(1)) called cpuSemaphore to control CPU access, ensuring only one process executes at a time. The key difference is that a semaphore can allow multiple threads (by setting permits > 1), while a lock is strictly binary.


---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Your answer here - reference try-finally blocks, lock ordering, etc.]

A deadlock occurs when two or more threads are waiting for each other to release a lock, causing all of them to be stuck forever and the program to hang.

**Technique 1 - Always unlock in finally block**: I used try finally blocks throughout my code to ensure that locks and semaphores are always released even if an exception occurs. Without finally, an exception could leave a lock permanently acquired, causing all other threads to wait forever.

**Technique 2 - Consistent lock ordering**: I never acquire multiple locks at the same time in my code. Each method acquires only one lock at a time (either counterLock or logLock), which eliminates the possibility of circular waiting that causes deadlocks.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[Your answer here - explain coarse-grained vs fine-grained locking, independence of counters, concurrency implications. Show understanding of when to use each approach. 5-8 sentences expected.]


I chose coarse-grained locking — using ONE lock (counterLock) for all three counter methods. This approach is simpler to implement and easier to manage since all three counter operations share the same lock. While fine-grained locking (separate lock per counter) would allow higher concurrency since the three counters are independent of each other, in this scheduler the threads execute sequentially (one at a time due to the semaphore), so the performance difference is minimal. The coarse-grained approach still correctly protects all critical sections and prevents all race conditions. If this were a highly concurrent system where many threads updated different counters simultaneously, fine-grained locking would be the better choice for performance.



---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: contextSwitchCount, completedProcessCount, totalWaitingTime

**Why they need protection**: These variables are shared across all threads. The increment and addition operations are not atomic, so concurrent access can cause lost updates and incorrect final values.

**Synchronization mechanism used**: ReentrantLock (counterLock)

**Code snippet**:
```java

public static final ReentrantLock counterLock = new ReentrantLock();

public static void incrementContextSwitch() {
    counterLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        counterLock.unlock();
    }
}

// Paste your implementation here
```

**Justification**: 

Using a single lock for all three counters ensures mutual exclusion. The finally block guarantees the lock is always released even if an exception occurs.

---

### Critical Section #2: Execution Log

**What resource**: executionLog (ArrayList<String>)

**Why it needs protection**: ArrayList is not thread-safe. Concurrent modifications can cause ConcurrentModificationException or data corruption.

**Synchronization mechanism used**: ReentrantLock (logLock)

**Code snippet**:
```java
// Paste your implementation here

public static final ReentrantLock logLock = new ReentrantLock();

public static void logExecution(String message) {
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }
}

```

**Justification**: A separate lock for the log avoids unnecessary contention with the counter lock, since the log and counters are independent resources.


---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To control how many processes can execute on the CPU simultaneously.


**Number of permits and why**: 1 permit — this creates a binary semaphore that ensures only one process runs at a time, simulating a single-core CPU.


**Where implemented**: In both run() and runToCompletion() methods of the Process class.

**Code snippet**:
```java
// Paste your implementation here

public static final Semaphore cpuSemaphore = new Semaphore(1);

public void run() {
    try {
        SharedResources.cpuSemaphore.acquire();
        try {
            // process execution code
        } finally {
            SharedResources.cpuSemaphore.release();
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}


```

**Effect on program behavior**: Only one process can execute at a time. Other processes must wait until the semaphore permit is released, preventing concurrent CPU access.


---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)

javac SchedulerSimulationSync.java
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
```

**Results**: 
(Show that running multiple times produces consistent, correct results)

The context switch count = 26, completed process count = 14, total waiting time = 576117ms ish, and average waiting time = 41146ms were identical across all 5 runs.



**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)

Without synchronization, multiple threads could increment contextSwitchCount simultaneously causing lost updates, or modify the ArrayList concurrently causing exceptions. Even if race conditions don't always manifest visibly, they can produce incorrect results unpredictably.


**Conclusion**: 
Synchronization ensures deterministic and correct results every time the program runs.

---

### Test 2: Exception Testing
**What I tested**: Checking that no ConcurrentModificationException occurs.

**Testing procedure**: Ran the program 5 times and monitored the output for any exceptions.


**Results**: No exceptions occurred in any run.

**What this proves**: The logLock successfully protects the ArrayList from concurrent modifications.


---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: completedProcessCount should equal numProcesses at the end.

**Actual values**: completedProcessCount matched numProcesses in all runs.

**Analysis**: The counterLock successfully prevents lost updates to the completedProcessCount variable.


---

### Test 4: Different Scenarios
**Scenario tested**: The program automatically uses different time quantum and process counts based on student ID.


**Purpose**: To verify the synchronization works correctly regardless of the number of processes.

**Results**: Program ran correctly and produced consistent results.


**What I learned**: Synchronization mechanisms work correctly regardless of the number of threads or time quantum values.


---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[6-8 sentences about key concepts, challenges, insights]

Before this assignment, I understood threads conceptually but did not fully appreciate the problems that arise when multiple threads share data. Through implementing ReentrantLock and Semaphore, I learned that even simple operations like incrementing a counter are not safe in a multithreaded environment. The try finally pattern is critical   without it, a single exception could leave a lock permanently acquired and hang the entire program. I also learned the difference between locks (binary, mutual exclusion) and semaphores (permit-based, resource pooling). Understanding lock granularity choosing between one lock for all resources versus separate locks per resource showed me that synchronization design involves trade offs between simplicity and performance. This assignment made concurrent programming concepts concrete and practical.

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1: Bank transactions**: When multiple users transfer money simultaneously, the account balance must be protected by locks to prevent race conditions where two withdrawals read the same balance and both succeed, resulting in more money withdrawn than available.


**Example 2: Database connection pools**: A semaphore is used to limit the number of concurrent database connections. If the pool has 10 connections, a Semaphore(10) ensures only 10 threads can access the database simultaneously, with others waiting until a connection is released.


---

### How I would explain synchronization to others:

[Explain to someone who just finished Assignment 1 - use simple terms and analogies]

Imagine a shared notebook in a classroom where multiple students want to write at the same time. If two students write simultaneously, the entries get mixed up and become unreadable. A lock is like a rule that says only one student can hold the pen at a time  others must wait. A semaphore is like having a limited number of pens  if there are 3 pens, only 3 students can write at once. In our CPU scheduler, threads are the students and shared variables are the notebook. Without synchronization, the data gets corrupted just like the notebook would.


---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/Yousef-Alsaeed1/OS-Assignment3-Yousef-Alsaeed

**Number of commits**: 5

**Commit messages**: 
1. Set my student ID: 445050012
2. Task 1 (445050012): Added ReentrantLock for counter protection
3. Task 2 (445050012): Added ReentrantLock for execution log
4. Task 3 (445050012): Implemented semaphore for CPU control
5. fininshing the document

---

## Summary


**Total time spent on assignment**: Approximately 3 hours

**Key takeaways**: 
1. Always release locks in a finally block to prevent deadlocks
2. ArrayList and other non-thread-safe collections must be protected with locks in multithreaded programs
3. Semaphores are useful for controlling access to limited resources like a CPU


**Most challenging aspect**: Getting the nested try-catch-finally structure correct for the semaphore in the run() method.



**What I'm most proud of**: Successfully implementing all three synchronization mechanisms and understanding why each one is necessary.


---

**End of Documentation**
