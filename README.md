#proj62-linux-lock-priority-inheritance

曾语馨-1120232858，
Linux 锁优先级传递
一、题目分析
优先级翻转是指当一个高优先级任务通过锁等机制访问共享资源时, 该锁如果已被一低优先级任务占有, 将可能造成高优先级任务被许多具有较低优先级任务阻塞. 优先级翻转问题是影响 linux 实时性的障碍之一, 甚至引起很多稳定性问题.

该项目的目标:
实现 rwlock、rwsem、mutex 的优先级继承支持.
通过内核宏开关优先级继承相关特性

特征
优先级继承、传递
实时性


基于 Linux kernel 4.19 或 5.10 或社区主线实现
描述算法思路, 实现相关算法, 可以编译安装运行
提供与原生锁等在时延方面的测试数据
补丁发送到社区主线(可选)


License
GPL-2.0(https://opensource.org/licenses/GPL-2.0)

参考资料
Utilization inversion and proxy execution   https://lwn.net/Articles/820575
CONFIG_RT_MUTEX_FULL  https://rt.wiki.kernel.org/index.php/Main_Page
Linux Futex PI  https://www.kernel.org/doc/Documentation/pi-futex.txt

二、优先级翻转

1.操作系统的任务调度
操作系统有多个任务，任务之间谁可以得到执行，是通过任务调度来完成的
2.任务调度有多种方法（算法）
常见的有：
罗宾环调度算法：Round-robin scheduling algorithm
基于优先级的调度算法：Priority-controlled scheduling algorithm
3.任务调度的一种常见调度算法就是
根据优先级高低去调度，优先让高优先级的任务去执行的
核心逻辑可以总结为：
任务调度器，总是去激活某个，在所有任务中优先级是最高的，且处于就绪状态的，任务，即让其去执行
4.任务有多种状态：就绪，挂起，等等
当然，任何任务，都可能由于，需要某种资源，而该资源被别人（别的任务）占用，而无法继续运行下去
此时就变成：挂起 –> 等待其所需要的资源被释放
然后才可以继续变成，就绪，等待下次调度时，就可以继续执行了。
5.任务一般被称为：进程，或更小粒度的线程

具体来说：当高优先级任务正等待信号量（此信号量被一个低优先级任务拥有着）的时候，一个介于两个任务优先之间的中等优先级任务开始执行——这就会导致一个高优先级任务在等待一个低优先级任务，而低优先级任务却无法执行类似死锁的情形发生。

一个具体的例子：
假定一个进程中有三个线程Thread1(高）、Thread2（中）和Thread3（低），考虑下图的执行情况。

<img width="582" height="366" alt="1781698-20210203141734902-404926186" src="https://github.com/user-attachments/assets/2de2e55a-78a1-4bc7-8ce3-2a58c2b45544" />

优先级反转实例图示
T0时刻，Thread3运行，并获得同步资源SYNCH1；
T1时刻，Thread2开始运行，由于优先级高于Thread3，Thread3被抢占（未释放同步资源SYNCH1），Thread2被调度执行；
T2时刻，Thread1抢占Thread2；
T3时刻，Thread1需要同步资源SYNCH1，但SYNCH1被更低优先级的Thread3所拥有，Thread1被挂起等待该资源
而此时线程Thread2和Thread3都处于可运行状态，Thread2的优先级大于Thread3的优先级，Thread2被调度执行。最终的结果是高优先级的Thread1迟迟无法得到调度，而中优先级的Thread2却能抢到CPU资源。
上述现象中，优先级最高的Thread1要得到调度，不仅需要等Thread3释放同步资源（这个很正常），而且还需要等待另外一个毫不相关的中优先级线程Thread2执行完成（这个就不合理了），会导致调度的实时性就很差了。

什么是优先级继承
优先级继承就是为了解决优先级反转问题而提出的一种优化机制。其大致原理是让低优先级线程在获得同步资源的时候(如果有高优先级的线程也需要使用该同步资源时)，临时提升其优先级。以前其能更快的执行并释放同步资源。释放同步资源后再恢复其原来的优先级。

 

<img width="612" height="327" alt="1781698-20210203141821935-1581756944" src="https://github.com/user-attachments/assets/809d77aa-7822-4ca3-8f18-972411aa0a73" />

　　　　　　　　　　　　　　　　　　带有优先级继承调度过程
 

与上图相比，到了T3时刻，Thread1需要Thread3占用的同步资源SYNCH1，操作系统检测到这种情况后，就把 Thread3的优先级提高到Thread1的优先级。此时处于可运行状态的线程Thread2和Thread3中，Thread3的优先级大于Thread2的优先级，Thread3被调度执行。

Thread3执行到T4时刻，释放了同步资源SYNCH1，操作系统恢复了Thread3的优先级，Thread1获得了同步资源SYNCH1，重新进入可执行队列。处于可运行状态的线程Thread1和Thread2中，Thread1的优先级大于Thread2的优先级，所以Thread1被调度执行。

通过优先级继承机制，可以有效解决优先级反转问题，使优先级最高的Thread1获得执行的时机提前。

三、算法设计原理
该项目的目标:
实现 rwlock、rwsem、mutex 的优先级继承支持，通过内核宏开关优先级继承相关特性
