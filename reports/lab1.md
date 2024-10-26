# chaptr3

## 一、编程

题目：获取任务信息

思路：
1. 在内核控制块TaskControlBlock中添加了变量syscall_times:[u32; MAX_SYSCALL_NUM和task_start_time: usize，分别记录task使用系统调用的次数和task开始的时间；
2. 在TaskManager中实现task的系统调用次数和task开始时间的维护。
3. 在TASK_MANAGER初始化时，将所有task的syscall_times全部是置零。在将task的task_status设置为就绪状态时,同时将task的task_start_time设置为当前时间（ms）。
4. 在TaskManager增加了函数syscall_counter(&self, syscall_id: usize)，统计当前task的系统调用次数，使用调用的系统调用号作为参数。然后提供syscall_counter() pubilc调用接口。
5. 在trap_handler函数中，进入syscall()前调用syscall_counter()，统计系统调用次数。
6.  在TaskManager增加了函数get_syscall_times(&self) -> [u32; MAX_SYSCALL_NUM]，返回当前task的系统调用次数。然后提供get_syscall_times pubilc调用接口。
7.  在TaskManager增加了函数get_task_runtime(&self) -> usize，返回当前task的运行时间。然后提供get_task_runtime pubilc调用接口。
8.  在TaskManager增加了函数get_task_status(&self) -> TaskStatus，返回当前task的状态。然后提供get_task_status pubilc调用接口。
9. 在sys_task_info(ti: *mut TaskInfo) -> isize中调用get_task_status()、get_syscall_times()和get_task_runtime(),获取task信息。

## 二、简答