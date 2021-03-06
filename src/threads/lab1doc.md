# Project 1 Design Document

> Denison cs372  
> Fall 2018

## GROUP

> Fill in the name and email addresses of your group members.

- Ben Liepert <lieper_b1@denison.edu>
- Federico Read Grullon <readgr_r1@denison.edu>
- Emma Steinman <steinm_e1@denison.edu>

## PRELIMINARIES

> If you have any preliminary comments on your submission, notes for the
> Instructor, or extra credit, please give them here.

We chose to complete Exercise 1.3 - Advanced Scheduler instead of Exercise 1.2.2 - Priority Donation. We also pass tests with different seeds.

> Please cite any offline or online sources you consulted while
> preparing your submission, other than the Pintos documentation, course
> text, lecture notes, and course staff.

## PROJECT PARTS

> In the documentation that you provide in the following, you need to provide clear, organized, and coherent documentation, not just short, incomplete, or vague "answers to the questions".  The questions are provided to help you structure your thinking around the information you need to convey.

### A. ALARM CLOCK  

#### DATA STRUCTURES

> A1: Copy here the declaration of each new or changed `struct` or
> `struct` member, `global` or `static` variable, `typedef`, or
> enumeration.  Document the purpose of each in 25 words or less.

 * `(extern) struct list sleeping_list;`
     * timer.c and thread.h
     * keep track of the sleeping threads to make it more efficient when checking which threads to wake up in timer_interrupt()

*  additions to `struct thread`
    * thread.h
    * `int64_t wakeAt` is the value at which to wake the thread, -1 if not sleeping
    * `struct list_elem sleepingelem` is a struct to keep track of an element in sleepingList
    * `struct semaphore *sleepSema` is a semaphore to prevent race conditions when sleeping 
* `extern struct ready_list;`
   * list.c
   * declare as an extern struct to make it global



#### ALGORITHMS

> A2: Briefly describe the algorithmic flow for a call to `timer_sleep()`,
> including the effects of the timer interrupt handler.

`timer_sleep()` gets the current thread and sets it’s `wakeAt` value to the absolute time it should wake up. It then disables interrupts before pushing the thread onto the `sleepingList`. Then it calls `thread_block()` which blocks the thread and reenables interrupts. `timer_interrupt()`, which is called every tick, calls `threads_wake()` which iterates through each element currently in the sleeping list. If the thread’s `wakeAt` value is less than or equal to the total number of ticks, it is time to wake up the thread so the thread is removed from `sleepingList` and the thread is unblocked. If it’s priority is higher than the next thread to run, `thread_yield()` is called, running this thread instead. 

> A3: What steps are taken to minimize the amount of time spent in
> the timer interrupt handler?

By creating `sleepingList`, we minimize the time it takes to iterate through to find the threads that are sleeping, as opposed to iterating through all of the threads and checking their `wakeAt` values.

#### SYNCHRONIZATION

> Think about, in the general case, the possible concurrent threads of execution involved in the pieces of code for the Alarm Clock, and the synchronization needed to provide a correct solution to the problem.

> A4: How are race conditions avoided when multiple threads call
> `timer_sleep()` simultaneously?

By disabling interrupts momentarily (while the thread is inserted into `sleepingList` and blocked) even if other threads call `timer_sleep()` the running thread will be able to finish the critical section atomically. 

> A5: How are race conditions avoided when a timer interrupt occurs
> during a call to `timer_sleep()`?

To handle timer interrupts during timer_sleep() we disabled interrupts right before the thread is pushed onto sleepingList and calling sema_down() which will make sure the awoken thread is set to ready and possibly scheduled atomically. Otherwise, interrupts should not catastrophically ruin the execution of the thread. 

#### RATIONALE/JUSTIFICATION

> A6: Why did you choose this design?  In what ways is it superior to
> another design you considered?

After a few iterations, we finalized our design when it worked. Especially this early in the project we were trying to make our code as efficient as possible. At one point we were iterating over every element in ready_list and checking to see if the wakeAt value was correct, then we realize we could make a sleeping_list to eliminate having to iterate over every thread. We also moved our code from `thread_tick()` to a new function `thread_wake()` which is called in the interrupt handler, as opposed to searching every element in the list when it is just trying to increment `ticks`. We tried different ways of preventing race conditions, many of which did not work. If we had more time we would be able to maximize efficiency more but our solution worked how we wanted it to. 

### PRIORITY SCHEDULING

#### DATA STRUCTURES

> B1: Copy here the declaration of each new or changed `struct` or
> `struct` member, `global` or `static` variable, `typedef`, or
> enumeration.  Document the purpose of each in 25 words or less.

We did not declare any new `struct` or variables for this exercise. 

> B2: Explain the data structure used to track priority donation.  Provide a diagram showing nested donation, using Markdown include of a jpeg or png.

We chose to complete the Advanced Scheduler exercise instead of priority donation.


#### ALGORITHMS

> B3: How do you ensure that the highest priority thread waiting for
> a lock, semaphore, or condition variable wakes up first?

To ensure the thread, lock, condition variable, or semaphore with the highest priority is woken up first we ordered readyList by priority. Every time anything is inserted to the list, it uses list_insert_ordered() which inserts it so that the list is always sorted in decreasing order. Thus, the element at the front of the list always has the highest priority.

> B4: Describe the sequence of events when a call to `lock_acquire()`
> causes a priority donation.  How is nested donation handled?

We chose to complete the Advanced Scheduler exercise instead of priority donation.

> B5: Describe the sequence of events when `lock_release()` is called
> on a lock that a higher-priority thread is waiting for.

We chose to complete the Advanced Scheduler exercise instead of priority donation.

#### SYNCHRONIZATION

> B6: Describe a potential race in `thread_set_priority()` and explain
> how your implementation avoids it.  Can you use a lock to avoid
> this race?

A potential race condition in thread_set_priority is if an interrupt comes after thread_set_priority() is called but before it actually stores the new priority value. In this case, the current thread’s priority should be higher but the interrupt blocks the thread before the priority is actually changed, so when the interrupt returns, another thread might be called instead of the original running thread because the priority was not saved. We disable interrupts before changing the thread’s priority and disable them right before checking whether or not to yield to the next thread to run. This way, the thread’s new priority is set atomically. can you use locks?????

#### RATIONALE

> B7: Why did you choose this design?  In what ways is it superior to
> another design you considered?

We kept track of priority and chose to always insert ordered when putting threads in the ready list, thus the first element will always have the highest priority and we do not have to iterate over the list each time to find the highest priority. We chose to complete the Advanced Scheduler exercise rather than Priority Donation. 

### ADVANCED SCHEDULER

#### DATA STRUCTURES

> C1: Copy here the declaration of each new or changed `struct` or
> `struct` member, `global` or `static` variable, `typedef`, or
> enumeration.  Document the purpose of each in 25 words or less.

* `fixed_point_t load_avg`
    * in `timer.c`
    * we used this to keep track of the load average for use in calculations related to advanced scheduling
* changed `struct thread`
    * `int niceVal` holds the niceness value of the thread, related to advanced scheduling 
    * `fixed_point_t recent_cpu` we use this to keep track of recent cpu usage, used in calculating priority and load average
* use `typedef struct` (but we did not write)
    * in `fixed-point.h`
    * `int f` keeps track of the fixed point number without wasting space 
#### ALGORITHMS

> C2: Suppose threads A, B, and C have "nice" values 0, 1, and 2, respectively.  Each
> has a recent_cpu value of 0.  Fill in the table below showing the
> scheduling decision (thread-to-run) and the priority and recent_cpu values for each
> thread after each given number of timer ticks:


ticks |  cpu A | cpu B | cpu C | pri A | pri B | pri C  | thread-to-run
----- | -- | -- | -- | -- | -- | --  | ------
0|0|0|0|63|61|59
4| 0 | 1 |  2| 63 |60.75  |  58.5|A
8| 0 | 1 |  2| 63 | 60.75 | 58.5 |A
12| 0| 1.047619 | 2.0952 | 63 | 60.7380775 |  58.47619 |A
16| 0| 1.049886 |  2.09977|  63|  60.737529| 58.47505  |A
20| 0 | 1.04999457 |  2.09998938| 63 | 60.737501 | 58.4750027  |A
24| 0 | 1.0869107 | 2.173782153 | 63 | 60.72827723 | 58.45655446  |A
28| 0 | 1.089946018 | 2.179888787 | 63 | 60.7275135 | 58.4550278  |A
32| 0 | 1.090197202 | 2.180394135 | 63 | 60.7274507 |  58.45490147 |A
36| 0 | 1.140145726 | 2.280291417 | 63 | 60.71496357 |  58.42992715 |A

> C3: Did any ambiguities in the scheduler specification make values
> in the table uncertain?  If so, what rule did you use to resolve
> them?  Does this match the behavior of your scheduler?

There were a few ambiguities that made values in the table uncertain.
  * How often to recalculate load_avg?
    * If the load_avg is not reset often enough, no values change. Here we decided to recalculate every 12 ticks to better demonstrate how niceness affects priority and thus recent_cpu. 
    * In our scheduler we recalculate load_avg every 100 ticks, because there are more threads and more happening per tick 
  * How does it decide which thread to run?
    * Here, we used highest priority to decide which thread runs next because we do not know the order of arrival or time to completion or any other metric other than priority.
    * This is not good in this simulation because it leads to starvation as thread A has a niceness of 0 and it's priority never changes as the priorities of both thread B and thread C slowly decrease, thus thread B and thread C do not get to run.
    * Our scheduler also uses highest priority but with more threads and more ticks the values will vary more and have less starvation.
> C4: How is the way you divided the cost of scheduling between code
> inside and outside interrupt context likely to affect performance?

When we were deciding what to include inside interrupt context vs outside interrupt context, we agreed that safety was better than efficiency for our project. However, in professional coding it is better to maximize efficiency rather than focus only on safety. Our code might be less efficient thus running slightly slower than a program that balanced code between inside and outside interrupt context. 

#### RATIONALE

> C5: Briefly critique your design, pointing out advantages and
> disadvantages in your design choices.  If you were to have extra
> time to work on this part of the project, how might you choose to
> refine or improve your design?

Besides making sure it passes all of the tests, we would have focused more on the design and efficiency of the program. Advantages of our design was the fact that we were passing tests intuitively, but disadvantages were the lack of efficiency and overarching design. If we had more time we would have made sure the code worked to our standards. 

> C6: The assignment explains arithmetic for fixed-point math in
> detail, but it leaves it open to you to implement it.  Why did you
> decide to implement it the way you did?  If you created an
> abstraction layer for fixed-point math, that is, an abstract data
> type and/or a set of functions or macros to manipulate fixed-point
> numbers, why did you do so?  If not, why not?

We chose to use the .h file given to us to implement fixed-point math. If we had more time we could have written our own but we used it to save time and focus more on the scheduling part of the exercise. 
