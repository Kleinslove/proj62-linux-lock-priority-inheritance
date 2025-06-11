# urb
曾语馨-1120232858，
Linux 锁优先级传递
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
