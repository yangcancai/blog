---
title: dead_lock
date: 2020-06-18 23:08:10
categories:
- c++ 
tags:
- dead_lock 
- gdb
- c++ 
- Valgrind 
---
 大家都知道写程序不小心很容易造成死锁，死锁的结果就是程序卡死了。如果在线上出现这种情况该如何处理尼。这篇文章会模拟死锁的情况使用工具进行问题定位，基本算是一篇入门的文章，各路大神可以忽略。
### linux c++死锁调试
#### AB-BA型的c++代码
```c++
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <iostream>
#include <unistd.h>
typedef struct thread_str{
    int rs;
}thread_str;
int g_tickets = 100;
int g_num = 100;
pthread_mutex_t g_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t g_mutex_num = PTHREAD_MUTEX_INITIALIZER;
void sell_num(int thread){
    pthread_mutex_lock(&g_mutex_num);
    if (g_num > 0)
        printf("thread%d sell num:%d\n", thread, g_num--);
    pthread_mutex_unlock(&g_mutex_num);
    return;
}
void* thread_proc1(void* arg)
{
    auto* pthread = static_cast<thread_str*>(arg);
    while (1)
    {
        pthread_mutex_lock(&g_mutex);
        usleep(10);
        sell_num(1);
        if (g_tickets > 0)
            printf("thread 1 sell tickets:%d\n", g_tickets--);
        else
        {
            pthread_mutex_unlock(&g_mutex);
            break;
        }
        pthread_mutex_unlock(&g_mutex);
    }
    pthread->rs = 1;
    pthread_exit(pthread);
}

void* thread_proc2(void* arg)
{
    auto* pthread = static_cast<thread_str*>(arg);
    while (1)
    {
        pthread_mutex_lock(&g_mutex_num);
        pthread_mutex_lock(&g_mutex);
        if (g_tickets > 0)
            printf("thread 2 sell tickets:%d\n", g_tickets--);
        else
        {
            pthread_mutex_unlock(&g_mutex);
            if (g_num > 0)
                printf("thread%d sell num:%d\n", 2, g_num--);
            pthread_mutex_unlock(&g_mutex_num);
            break;
        }
        pthread_mutex_unlock(&g_mutex);
        if (g_num > 0)
            printf("thread%d sell num:%d\n", 2, g_num--);
        pthread_mutex_unlock(&g_mutex_num);
    }
    pthread->rs = 2;
    pthread_exit(pthread);
}
int main(int argc, char*argv[])
{
    pthread_t tid1, tid2;
    void *ret1, *ret2;
    thread_str t1,t2;
    pthread_create(&tid1, NULL, thread_proc1, &t1);
    pthread_create(&tid2, NULL, thread_proc2, &t2);
    pthread_join(tid1, &ret1);
    pthread_join(tid2, &ret2);
    printf("ret1:%d\n", static_cast<thread_str*>(ret1)->rs);
    printf("ret2:%d\n", static_cast<thread_str*>(ret2)->rs);
    getchar();
    return 0;
}
```
#### 方法一gdb
##### 1、启动程序./build.sh
```bash
[cam@localhost demo_cpp]$ ./build.sh 
-- Configuring done
-- Generating done
-- Build files have been written to: /data/c++/demo_cpp/cmake-build-debug
[ 50%] Building CXX object CMakeFiles/debug_dead_lock.dir/debug_dead_lock.cpp.o
[100%] Linking CXX executable debug_dead_lock
[100%] Built target debug_dead_lock
thread 2 sell tickets:100
thread2 sell num:100
thread 2 sell tickets:99
thread2 sell num:99
thread 2 sell tickets:98
thread2 sell num:98
.....................
.....................
thread2 sell num:45
thread 2 sell tickets:44
thread2 sell num:44
thread 2 sell tickets:43
thread2 sell num:43
 ```
##### 2、查看进程的pid ps -ef | grep debug
```bash
  [cam@localhost ~]$ ps -ef | grep debug
  cam      29815 29788  0 04:30 pts/0    00:00:00 ./debug_dead_lock
```
##### 3、查看当前进程所有的线程
```bash
[cam@localhost ~]$ pstree -p 29815
debug_dead_lock(29815)─┬─{debug_dead_loc}(29816)
                       └─{debug_dead_loc}(29817)
```
该进程包括一个main线程和2个子线程，跟代码表现是一样的
##### 4、启动gdb，attach到该进程，打印所有的线程栈
```bash
[cam@localhost ~]$ sudo gdb
GNU gdb (GDB) Red Hat Enterprise Linux 8.2-3.el6
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
(gdb) attach 29815
Attaching to process 29815
[New LWP 29816]
[New LWP 29817]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
0x00007f782442c2fd in pthread_join () from /lib64/libpthread.so.0
(gdb) thread apply all bt

Thread 3 (Thread 0x7f78230ee700 (LWP 29817)):
#0  0x00007f7824432334 in __lll_lock_wait () from /lib64/libpthread.so.0
#1  0x00007f782442d5d8 in _L_lock_854 () from /lib64/libpthread.so.0
#2  0x00007f782442d4a7 in pthread_mutex_lock () from /lib64/libpthread.so.0
#3  0x00000000004009c6 in thread_proc2 (arg=0x7ffc09b61b08) at /data/c++/demo_cpp/debug_dead_lock.cpp:52
#4  0x00007f782442baa1 in start_thread () from /lib64/libpthread.so.0
#5  0x00007f78239d8c4d in clone () from /lib64/libc.so.6

Thread 2 (Thread 0x7f78238ef700 (LWP 29816)):
#0  0x00007f7824432334 in __lll_lock_wait () from /lib64/libpthread.so.0
#1  0x00007f782442d5d8 in _L_lock_854 () from /lib64/libpthread.so.0
#2  0x00007f782442d4a7 in pthread_mutex_lock () from /lib64/libpthread.so.0
#3  0x00000000004008d7 in sell_num (thread=1) at /data/c++/demo_cpp/debug_dead_lock.cpp:19
#4  0x0000000000400945 in thread_proc1 (arg=0x7ffc09b61b0c) at /data/c++/demo_cpp/debug_dead_lock.cpp:32
#5  0x00007f782442baa1 in start_thread () from /lib64/libpthread.so.0
#6  0x00007f78239d8c4d in clone () from /lib64/libc.so.6

Thread 1 (Thread 0x7f7824854720 (LWP 29815)):
#0  0x00007f782442c2fd in pthread_join () from /lib64/libpthread.so.0
#1  0x0000000000400af4 in main (argc=1, argv=0x7ffc09b61c18) at /data/c++/demo_cpp/debug_dead_lock.cpp:80
(gdb) 
```
总结：Thread1和Thread2当前线程都执行到__lll_lock_wait这个函数，再查看对应debug_dead_lock.cpp:19和debug_dead_lock.cpp:32,
对应的代码都是等待锁，这里是两个锁，1线程等待2线程获取的锁，2线程等待1线程获取的锁，就造成了dead_lock
#### 方法二valgrind(helgrind)
##### 1、命令:valgrind --tool=helgrind cmake-build-debug/debug_dead_lock
```bash
[cam@localhost demo_cpp]$ valgrind --tool=helgrind cmake-build-debug/debug_dead_lock 
==29920== Helgrind, a thread error detector
==29920== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==29920== Using Valgrind-3.14.0 and LibVEX; rerun with -h for copyright info
==29920== Command: cmake-build-debug/debug_dead_lock
==29920== 
thread1 sell num:100
thread 1 sell tickets:100
==29920== ---Thread-Announcement------------------------------------------
==29920== 
==29920== Thread #3 was created
==29920==    at 0x58DEC0E: clone (in /lib64/libc-2.12.so)
==29920==    by 0x4E3F90F: do_clone.clone.0 (in /lib64/libpthread-2.12.so)
==29920==    by 0x4E3FE6C: pthread_create@@GLIBC_2.2.5 (in /lib64/libpthread-2.12.so)
==29920==    by 0x4C34320: pthread_create_WRK (hg_intercepts.c:427)
==29920==    by 0x4C344A5: pthread_create@* (hg_intercepts.c:460)
==29920==    by 0x400AE0: main (debug_dead_lock.cpp:79)
==29920== 
==29920== ----------------------------------------------------------------
==29920== 
==29920== Thread #3: lock order "0x6020C0 before 0x602100" violated
==29920== 
==29920== Observed (incorrect) order is: acquisition of lock at 0x602100
==29920==    at 0x4C320C5: mutex_lock_WRK (hg_intercepts.c:909)
==29920==    by 0x4C32150: pthread_mutex_lock (hg_intercepts.c:925)
==29920==    by 0x4009BB: thread_proc2(void*) (debug_dead_lock.cpp:51)
==29920==    by 0x4C3452E: mythread_wrapper (hg_intercepts.c:389)
==29920==    by 0x4E40AA0: start_thread (in /lib64/libpthread-2.12.so)
==29920==    by 0x58DEC4C: clone (in /lib64/libc-2.12.so)
==29920== 
==29920==  followed by a later acquisition of lock at 0x6020C0
==29920==    at 0x4C320C5: mutex_lock_WRK (hg_intercepts.c:909)
==29920==    by 0x4C32150: pthread_mutex_lock (hg_intercepts.c:925)
==29920==    by 0x4009C5: thread_proc2(void*) (debug_dead_lock.cpp:52)
==29920==    by 0x4C3452E: mythread_wrapper (hg_intercepts.c:389)
==29920==    by 0x4E40AA0: start_thread (in /lib64/libpthread-2.12.so)
==29920==    by 0x58DEC4C: clone (in /lib64/libc-2.12.so)
==29920== 
==29920== Required order was established by acquisition of lock at 0x6020C0
==29920==    at 0x4C320C5: mutex_lock_WRK (hg_intercepts.c:909)
==29920==    by 0x4C32150: pthread_mutex_lock (hg_intercepts.c:925)
==29920==    by 0x400930: thread_proc1(void*) (debug_dead_lock.cpp:30)
==29920==    by 0x4C3452E: mythread_wrapper (hg_intercepts.c:389)
==29920==    by 0x4E40AA0: start_thread (in /lib64/libpthread-2.12.so)
==29920==    by 0x58DEC4C: clone (in /lib64/libc-2.12.so)
==29920== 
==29920==  followed by a later acquisition of lock at 0x602100
==29920==    at 0x4C320C5: mutex_lock_WRK (hg_intercepts.c:909)
==29920==    by 0x4C32150: pthread_mutex_lock (hg_intercepts.c:925)
==29920==    by 0x4008D6: sell_num(int) (debug_dead_lock.cpp:19)
==29920==    by 0x400944: thread_proc1(void*) (debug_dead_lock.cpp:32)
==29920==    by 0x4C3452E: mythread_wrapper (hg_intercepts.c:389)
==29920==    by 0x4E40AA0: start_thread (in /lib64/libpthread-2.12.so)
==29920==    by 0x58DEC4C: clone (in /lib64/libc-2.12.so)
==29920== 
==29920==  Lock at 0x6020C0 was first observed
==29920==    at 0x4C320C5: mutex_lock_WRK (hg_intercepts.c:909)
==29920==    by 0x4C32150: pthread_mutex_lock (hg_intercepts.c:925)
==29920==    by 0x400930: thread_proc1(void*) (debug_dead_lock.cpp:30)
==29920==    by 0x4C3452E: mythread_wrapper (hg_intercepts.c:389)
==29920==    by 0x4E40AA0: start_thread (in /lib64/libpthread-2.12.so)
==29920==    by 0x58DEC4C: clone (in /lib64/libc-2.12.so)
==29920==  Address 0x6020c0 is 0 bytes inside data symbol "g_mutex"
==29920== 
==29920==  Lock at 0x602100 was first observed
==29920==    at 0x4C320C5: mutex_lock_WRK (hg_intercepts.c:909)
==29920==    by 0x4C32150: pthread_mutex_lock (hg_intercepts.c:925)
==29920==    by 0x4008D6: sell_num(int) (debug_dead_lock.cpp:19)
==29920==    by 0x400944: thread_proc1(void*) (debug_dead_lock.cpp:32)
==29920==    by 0x4C3452E: mythread_wrapper (hg_intercepts.c:389)
==29920==    by 0x4E40AA0: start_thread (in /lib64/libpthread-2.12.so)
==29920==    by 0x58DEC4C: clone (in /lib64/libc-2.12.so)
==29920==  Address 0x602100 is 0 bytes inside data symbol "g_mutex_num"
==29920== 
==29920== 
thread 2 sell tickets:99
thread2 sell num:99
thread 2 sell tickets:98
thread2 sell num:98
thread 2 sell tickets:97
thread2 sell num:97
thread 2 sell tickets:96
thread2 sell num:96
thread 2 sell tickets:95
thread2 sell num:95
thread 2 sell tickets:94
thread2 sell num:94
thread 2 sell tickets:93
thread2 sell num:93

^C==29920== 
==29920== Process terminating with default action of signal 2 (SIGINT)
==29920==    at 0x4E412FD: pthread_join (in /lib64/libpthread-2.12.so)
==29920==    by 0x4C325CA: pthread_join_WRK (hg_intercepts.c:553)
==29920==    by 0x4C326A5: pthread_join (hg_intercepts.c:572)
==29920==    by 0x400AF3: main (debug_dead_lock.cpp:80)
==29920== ----------------------------------------------------------------
==29920== 
==29920== Thread #3: Exiting thread still holds 1 lock
==29920==    at 0x4E47334: __lll_lock_wait (in /lib64/libpthread-2.12.so)
==29920==    by 0x4E425D7: _L_lock_854 (in /lib64/libpthread-2.12.so)
==29920==    by 0x4E424A5: pthread_mutex_lock (in /lib64/libpthread-2.12.so)
==29920==    by 0x4C32065: mutex_lock_WRK (hg_intercepts.c:902)
==29920==    by 0x4C32150: pthread_mutex_lock (hg_intercepts.c:925)
==29920==    by 0x4009C5: thread_proc2(void*) (debug_dead_lock.cpp:52)
==29920==    by 0x4C3452E: mythread_wrapper (hg_intercepts.c:389)
==29920==    by 0x4E40AA0: start_thread (in /lib64/libpthread-2.12.so)
==29920==    by 0x58DEC4C: clone (in /lib64/libc-2.12.so)
==29920== 
==29920== ---Thread-Announcement------------------------------------------
==29920== 
==29920== Thread #2 was created
==29920==    at 0x58DEC0E: clone (in /lib64/libc-2.12.so)
==29920==    by 0x4E3F90F: do_clone.clone.0 (in /lib64/libpthread-2.12.so)
==29920==    by 0x4E3FE6C: pthread_create@@GLIBC_2.2.5 (in /lib64/libpthread-2.12.so)
==29920==    by 0x4C34320: pthread_create_WRK (hg_intercepts.c:427)
==29920==    by 0x4C344A5: pthread_create@* (hg_intercepts.c:460)
==29920==    by 0x400AC3: main (debug_dead_lock.cpp:78)
==29920== 
==29920== ----------------------------------------------------------------
==29920== 
==29920== Thread #2: Exiting thread still holds 1 lock
==29920==    at 0x4E47334: __lll_lock_wait (in /lib64/libpthread-2.12.so)
==29920==    by 0x4E425D7: _L_lock_854 (in /lib64/libpthread-2.12.so)
==29920==    by 0x4E424A5: pthread_mutex_lock (in /lib64/libpthread-2.12.so)
==29920==    by 0x4C32065: mutex_lock_WRK (hg_intercepts.c:902)
==29920==    by 0x4C32150: pthread_mutex_lock (hg_intercepts.c:925)
==29920==    by 0x4008D6: sell_num(int) (debug_dead_lock.cpp:19)
==29920==    by 0x400944: thread_proc1(void*) (debug_dead_lock.cpp:32)
==29920==    by 0x4C3452E: mythread_wrapper (hg_intercepts.c:389)
==29920==    by 0x4E40AA0: start_thread (in /lib64/libpthread-2.12.so)
==29920==    by 0x58DEC4C: clone (in /lib64/libc-2.12.so)
==29920== 
==29920== 
==29920== For counts of detected and suppressed errors, rerun with: -v
==29920== Use --history-level=approx or =none to gain increased speed, at
==29920== the cost of reduced accuracy of conflicting-access information
==29920== ERROR SUMMARY: 9 errors from 3 contexts (suppressed: 76 from 22)
```
结论：通过Thread #2: Exiting thread still holds 1 lock,Thread #3: Exiting thread still holds 1 lock
可以知道这两个线程发生了AB-BA类型的死锁
### gdb和valgrind对比
 1、gdb可以对启动的进程进行分析，不需要停止目标进程,所以对线死锁分析比较方便
 2、valgrind对于本地代码分析会更方便
### 本实例代码
[github demo_cpp]( https://github.com/yangcancai/demo_cpp/tree/c4ad603a47ad23e42e1861151aa225b0936b3996)
