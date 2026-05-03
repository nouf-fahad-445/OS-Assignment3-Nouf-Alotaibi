# Assignment 3 - Complete Documentation

**Student Name**: [Nouf fahad Alotaibi]  
**Student ID**: [445052117]  
**Date Submitted**: [Submission Date]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [30 April, 10.30 pm]
**What I implemented**: 
 frok Repository then updated the student ID to my ID, and performed the initial commit.
**Challenges encountered**: 
None; the environment setup was smooth.
**How I solved it**: 
 Used the standard VS Code Git integration for cloning and pushing.
**Testing approach**: 
 Ran the initial base code to observe how race conditions cause inconsistent data in the logs.
**Time spent**: 
45 min

---

### Entry 2 - [1 may, 12:30 AM]
**What I implemented**: 
Task 1 - Applied individual ReentrantLocks for each of the three counters
**Challenges encountered**: 
 Researching the performance benefits of multiple locks versus one global lock
**How I solved it**: 
Chose a "fine-grained" approach because the counters don't depend on each other
**Testing approach**: 
Performed 8 consecutive runs to ensure the counter totals remained perfectly stable.
**Time spent**: 
1 hour

---

### Entry 3 - [1 may , 1:30 AM]
**What I implemented**: 
Task 2 - Protected the ArrayList (execution log) using a dedicated ReentrantLock.
**Challenges encountered**: 
Noticed the potential for deadlocks if the lock isn't released after an error.
**How I solved it**: 
Wrapped the locking logic in try-finally blocks to guarantee the lock is always released.
**Testing approach**: 
Ran high-concurrency tests to confirm no ConcurrentModificationException occurred.
**Time spent**: 
1:30 houre

---

### Entry 4 - [1 may , 2:35 pm]
**What I implemented**: 
 Task 3 – Integrated a binary Semaphore (1 permit) to manage access to the CPU resource.
**Challenges encountered**: 
applied consistently to both the run() and runToCompletion() methods.
**How I solved it**: 
Wrapped the execution logic in both methods with acquire() and ensured release() is called in the finally block to prevent resource leaks.
**Testing approach**: 
Switched to Semaphore(2) to verify that multiple threads could run simultaneously, then switched back to 1 to confirm strict mutual exclusion.
**Time spent**: 
2 hours

---

### Entry 5 - [4 may, 12 AM]
**What I implemented**: 
 Task 4 – Finalized the documentation (ASSIGNMENT_DOCUMENTATION.md), filmed the demo video, and pushed the final commits.

**Challenges encountered**: 
 Finding the best way to explain "lock granularity" so it’s easy for anyone to understand
**How I solved it**: 
 Created a visual diagram to show how separate locks work and detailed the logic in the Q4 section of the report
**Testing approach**: 
Conducted 5 final "stress tests"; all outputs and statistics remained perfectly identical across every run
**Time spent**: 
3 hours

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[Your answer here - 4-6 sentences with code examples]
• Context Switch Counter: Concurrent access to the non-atomic ++ operation leads to lost updates and inaccurate counts.
• Execution Log: Using a non-thread-safe ArrayList causes data corruption and crashes during simultaneous .add() calls.
• Incorrect Behavior: These race conditions result in inconsistent state, overwriting data, and ConcurrentModificationException.
• Root Cause: Lack of synchronization when multiple threads modify shared resources simultaneously.


---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

ReentrantLock: Used for Mutual Exclusion. I used it in SharedResources (e.g., contextSwitchLock) to ensure only one thread at a time updates counters or logs to prevent data corruption.
• Semaphore: Used for Capacity Control. I used cpuSemaphore(1) in Process.run() to simulate a single-core CPU, allowing only one process to execute its quantum at any given moment.

[Your answer here - explain your implementation choices]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Your answer here - reference try-finally blocks, lock ordering, etc.]
Definition: Deadlock is a situation where two or more threads are blocked forever, each waiting for a resource held by the other.
• Prevention Techniques:
1. Try-Finally Blocks: Always releasing locks/semaphores in a finally block ensures that resources are freed even if an exception occurs.
2. Fine-Grained Locking: Using separate locks for independent resources (like logLock vs contextSwitchLock) reduces the chance of circular wait.
• Implementation: In my code, I wrapped every .lock() and .acquire() call with a try-finally structure, placing .unlock() and .release() in the finally block to guarantee resource release and avoid hanging threads.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[Your answer here - explain coarse-grained vs fine-grained locking, independence of counters, concurrency implications. Show understanding of when to use each approach. 5-8 sentences expected.]
Design Choice: I chose a fine-grained locking approach by using separate locks for each counter (e.g., contextSwitchLock, completedProcessLock, and waitingTimeLock).
• Reasoning: Since the three counters are logically independent, a thread updating the context switch count does not need to block another thread that is adding waiting time.
• Trade-offs:
• Coarse-grained (one lock) is simpler to implement and prevents deadlocks easily but creates a "bottleneck" where threads wait unnecessarily.
• Fine-grained (separate locks) is more complex and increases the risk of deadlocks if not handled carefully, but it significantly improves performance.
• Concurrency: The fine-grained approach provides better concurrency because it allows multiple threads to update different shared resources simultaneously. This minimizes "lock contention," ensuring the CPU scheduler remains efficient even with a high number of processes.

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
contextSwitchCount
completedProcessCount
totalWaitingTime


**Why they need protection**: 
These variables are shared across multiple threads representing different processes. Operations like count++ or total += time are not atomic; they involve a read-modify-write cycle. Without protection, a race condition occurs where multiple threads might read the same initial value and overwrite each other's updates, leading to "lost updates" and inaccurate simulation statistics.

**Synchronization mechanism used**: 
 contextSwitchLock, completedProcessLock, and waitingTimeLock
**Code snippet**:
```java
// Paste your implementation here
```
public static void incrementContextSwitch() {
    contextSwitchLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        contextSwitchLock.unlock();
    }
}

**Justification**: 
Fine-grained Locking: Implemented separate locks for independent counters to allow simultaneous updates and maximize concurrency.
Performance Optimization: Avoided "coarse-grained" locking to prevent threads from unnecessarily blocking each other while updating different stats.
Reliability: Used try-finally blocks to ensure locks are always released, effectively preventing deadlocks.
Data Integrity: This approach ensures accurate performance metrics without data corruption or execution delays.

---

### Critical Section #2: Execution Log

**What resource**: 
The executionLog object, which is an ArrayList<String> located in the SharedResources class.

**Why it needs protection**: 
he ArrayList class in Java is not thread-safe. When multiple process threads attempt to call .add() concurrently, they simultaneously modify the list's internal array and its size pointer. Without synchronization, this leads to a race condition that can cause ConcurrentModificationException, data corruption (where log entries are overwritten), or the program crashing with an ArrayIndexOutOfBoundsException.

**Synchronization mechanism used**: 
A dedicated ReentrantLock named logLock.

**Code snippet**:
```java
// Paste your implementation here
```
public static void logExecution(String message) {
    logLock.lock(); 
    try {
        executionLog.add(message); 
    } finally {
        logLock.unlock(); 
    }
}

**Justification**: 
Dedicated logLock: Implemented a specific ReentrantLock for the log to ensure mutual exclusion without blocking independent numerical counters.
 Fine-grained Strategy: This maintains high performance by preventing unnecessary bottlenecks, allowing logs and stats to update simultaneously.
 Deadlock Prevention: The try-finally block guarantees the lock is released even if the .add() operation fails, keeping the simulation running.
Data Safety: Ensures only one thread modifies the ArrayList at a time, preventing corruption and ConcurrentModificationException.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
The semaphore acts as a synchronization tool to control access to the CPU resource among competing process threads. It ensures that only a specific number of processes can "execute" (simulate work via Thread.sleep) at any given time, preventing multiple threads from printing their progress bars simultaneously and clashing in the console.
**Number of permits and why**: 
I used 1 permit (a binary semaphore).
 This mimics a single-core CPU environment where only one process can be actively running its time quantum at a time. Using more permits would allow concurrent execution, which contradicts the logic of a standard Round Robin scheduler.
**Where implemented**: 
implemented in the Process.run() method and the runToCompletion() method
**Code snippet**:
```java
// Paste your implementation here
```
public void run() {
    try {
        SharedResources.cpuSemaphore.acquire(); 
        try {
           //  execution code
        } finally {
            SharedResources.cpuSemaphore.release(); 
        }
    } catch (InterruptedException e) {
        System.out.println("Interrupted");
    }
}


**Effect on program behavior**: 
The semaphore forces threads to execute serially rather than in parallel. Without it, all process threads would start at once, printing overlapping progress bars and causing a race condition in the simulation logic. With the semaphore, the output remains clean and synchronized, correctly reflecting the "one-at-a-time" nature of CPU scheduling.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
I executed the SchedulerSimulationSync program using Student ID 445052117. I monitored the transition of 14 processes through the Ready Queue and verified the final synchronization statistics.
```

**Results**: 
(Show that running multiple times produces consistent, correct results)  
 Total Context Switches: 33.  
Total Completed Processes: 14 matches process count 
Average Waiting Time: identical each run  
Execution Log Entries: 66

**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)
Without synchronization, the non-deterministic nature of thread scheduling could lead to several issues:
Data Integrity: Prevents lost updates in contextSwitchCount and corruption in totalWaitingTime, ensuring accurate performance metrics.
Reliability: Eliminates non-deterministic behavior and crashes, providing a consistent and stable simulation outcome.

**Conclusion**: 
he implementation of fine-grained locks and a binary semaphore proved successful. The "Process Summary Table" confirms that every single process was accounted for, and the "Execution Log Summary" shows a complete record of all 66 scheduling events, proving the thread safety of the system

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException
Checking for ConcurrentModificationException and ensuring the program manages 14 concurrent threads without crashing.

**Testing procedure**: 
I monitored the console output during the execution of 14 processes, specifically looking for runtime exceptions while the scheduler was rapidly updating the executionLog and printing progress bars to the terminal
**Results**: 
Zero exceptions occurred, and the final log was complete with no missing or corrupted entries.
 The program reached the "ALL PROCESSES COMPLETED" without interruption
**What this proves**: 
 cpuSemaphore: Successfully enforced a "one-at-a-time" rule, preventing overlapping progress bars in the console.  
logLock: Effectively protected the ArrayList, making it thread-safe despite high-concurrency access.

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)
 Verifying correct final values 


**Expected values**: 
 Expected values: completedProcessCount should equal 14

**Actual values**: 
 All values matched perfectly; 14 processes completed

**Analysis**: 
This confirms that synchronization does not alter the logical behavior of the scheduler—it only ensures that the "Read-Modify-Write" cycle for counters and logs remains accurate under high concurrency.

---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]
Temporarily modified the code from Semaphore(1) to Semaphore(2).
**Purpose**: 
To observe the behavioral effect of allowing two processes to access the "CPU" simultaneously.
**Results**: 
With 2 permits, execution overlapped, resulting in interleaved progress bars in the console output. However, final counters remained 100% accurate because they were still protected by their respective ReentrantLocks
**What I learned**: 
 Semaphores are highly flexible tools that control the degree of concurrency (how many threads can enter a section), while Mutexes/Locks ensure Mutual Exclusion (protecting data integrity regardless of the number of threads).

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[6-8 sentences about key concepts, challenges, insights]
Working on this assignment really showed me how critical synchronization is for building systems that actually work. I realized it’s not just about stopping the program from crashing, but about keeping the data accurate when multiple things are happening at once. One big thing I noticed was how much "Fine-grained locking" helps; by using different locks for the logs and the statistics, the program ran much smoother because threads weren't constantly blocking each other for no reason.
The hardest part was definitely getting the try-finally blocks right to make sure locks always get released, which I learned is the only way to prevent the whole system from freezing up in a "Deadlock". Also, playing around with the Binary Semaphore gave me a much better idea of how a real OS manages a single CPU among many competing processes

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
 Banking Systems (Fund Withdrawals)
Synchronization prevents a "race condition" when two transactions try to access the same account simultaneously. It ensures the balance is updated sequentially so that a user cannot withdraw more money than is actually available
**Example 2**: 
E-commerce Inventory (Ticket/Product Sales)
Synchronization ensures that the last available item is not sold to multiple people at the exact same time. It "locks" the stock count for the first buyer, preventing overselling and ensuring data consistency across the databas

---

### How I would explain synchronization to others:

[Explain to someone who just finished Assignment 1 - use simple terms and analogies]
Synchronization is like a traffic light for your code; it ensures that multiple threads don't crash into each other when using the same data. It forces threads to wait their turn, making sure information stays accurate and the program runs smoothly without errors.

---

## Part 6: GitHub Repository Information

**Repository URL**: 

**Number of commits**: 

**Commit messages**: 
1. 
2. 
3. 
4. 

---

## Summary

**Total time spent on assignment**: 8 hours 

**Key takeaways**: 
1. Data Integrity: I learned that synchronization is vital for keeping shared data (like logs and counters) accurate when multiple threads are running
2. Using ReentrantLock taught me how to ensure only one thread modifies a resource at a time, preventing "Lost Updates
3. I discovered that "Fine-grained locking" (using separate locks) is better for performance than one big lock because it reduces thread blocking

**Most challenging aspect**: 
The hardest part was deciding on the "lock granularity" and making sure using separate locks was safe. I proved it worked by never nesting the locks, which prevented any deadlocks from happening.

**What I'm most proud of**: 
The final program works perfectly every time without any crashes or errors. It’s easy to see in the code exactly why each lock and semaphore is used to keep the threads organized.

---

**End of Documentation**
