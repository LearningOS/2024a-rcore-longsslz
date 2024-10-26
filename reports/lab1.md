# chaptr3

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

> 无

2. 此外，我也参考了以下资料，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

> 无

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。

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

### 1、
 sbi版本: rustsbi v0.3.2

(1) ch2b_bad_address 
0x0 as *mut u8 是一个空指针，指向内存地址 0x0。write_volatile(0) 试图将值 0 写入该地址。在大多数操作系统中，内存地址 0x0 是受保护的，不允许访问。直接访问这个地址会触发一个页错误（Page Fault），即程序试图访问未映射或受保护的内存地址。操作系统检测到这个非法访问后，就会终止应用程序。

(2) ch2b_bad_instruction
在用户模式下无法直接执行 `sret` 指令。`sret` 是 RISC-V 架构中的一条特权指令，用于从**系统模式（Supervisor Mode）**返回到用户模式（User Mode）。因此，用户模式下的代码无法直接使用 sret，否则会引发非法指令异常。

(3) ch2b_bad_register
在用户模式下访问特权级寄存器 `sstatus` 会引发非法指令异常或陷入异常，因为`sstatus`寄存器是一个特权级寄存器，只有在较高的特权级（如 S 特权级（Supervisor） 或 M 特权级（Machine））下才允许访问。

### 2、
1. 进入 `__restore` 时，`a0` 通常代表当前任务的 `TrapContext` 的地址。`TrapContext` 保存了当任务被中断或发生异常时的寄存器状态。`a0` 指向的 `TrapContext` 用于恢复寄存器，从而在返回用户模式后，任务可以继续从中断点或异常点继续执行。

`__restore` 的两种使用情景
-  **用户模式任务返回用户态执行**
   - 当用户模式任务在执行过程中发生中断或异常，操作系统内核会捕获该事件并保存用户模式的上下文到 `TrapContext` 中。处理完事件后，需要恢复任务上下文并返回用户模式继续执行。
   - 此时，`__restore` 的任务是读取 `TrapContext` 中的保存状态（寄存器、栈指针、程序计数器等），将处理器切换回用户模式，以便任务继续在用户态运行。
- . **上下文切换**
   - 在多任务操作系统中，当一个任务的时间片结束或操作系统决定切换到另一个任务时，当前任务的上下文（即寄存器和状态）会被保存在 `TrapContext` 中。
   - 在这种情况下，`__restore` 会使用 `TrapContext` 中保存的信息恢复新任务的状态，切换到用户模式，使新任务从它的上次中断点继续执行。

3. 
- **`sstatus` (Supervisor Status Register)**
   - **用途**：`sstatus` 寄存器包含处理器的状态信息，包括权限级别、全局中断使能等。
   - **进入用户态的意义**：在恢复用户态时，`sstatus` 设置可以确保处理器运行在用户权限级别。特别是，`sstatus` 中的 `SPP` 位（以前的特权级别）需要设置为用户模式，以确保返回用户态时处理器权限正确。此外，`SPIE` 位（Supervisor Previous Interrupt Enable）也会影响返回用户态后的中断状态。
   
-  **`sepc` (Supervisor Exception Program Counter)**
   - **用途**：`sepc` 存储返回用户态时的程序计数器地址。这个地址对应用户态程序触发异常或系统调用前的指令地址。
   - **进入用户态的意义**：当 `sret` 指令执行时，`sepc` 的值将被加载到 `pc` 中，作为恢复的用户程序的指令地址。这允许用户态程序从异常或中断发生前的位置继续执行。

-  **`sscratch` (Supervisor Scratch Register)**
   - **用途**：`sscratch` 用于存储异常处理的用户栈指针（user stack pointer），以便在用户态和内核态之间切换栈。
   - **进入用户态的意义**：在恢复用户态之前将用户态栈指针恢复到 `sscratch`，可以在后续发生异常或中断时快速切换栈。`sscratch` 的值在发生用户态异常时会被加载到 `sp` 中，用于指向用户栈位置。

3. 
- `x2` (sp, Stack Pointer)：
	•	x2 被跳过是因为它会在切换过程中被动态调整，不需要作为一般寄存器来保存。
	•	在异常或中断发生时，sp 的值已经在 sscratch 中保存并在 __restore 的最后几行重新加载回栈指针，因此不需要手动保存 x2。这样做可以避免重复操作，并确保栈指针与当前的执行模式（内核态或用户态）一致。
- `x4` (tp, Thread Pointer)：
	•	x4 通常用于存储线程指针，在很多应用中可以指向线程本地存储。由于系统设计上，用户应用在一般情况下不会使用 tp 寄存器，因此在这里跳过了它的保存和恢复。

4.
- `sp` 指向用户态栈：在返回用户态后，栈指针 `sp` 可以正确引用用户态的栈。
- `sscratch` 保存内核态栈指针：方便在下次发生陷入或异常时可以迅速切换到内核栈，确保异常处理过程能够安全地在内核栈上执行。

5. 状态切换发生在最后一条指令 `sret`，这是 RISC-V 架构中的“特权指令”之一，用于从系统模式（如内核态）返回用户模式。

6. 
- `sp` 指向内核态栈：进入内核态后 `sp` 使用内核栈。
- `sscratch` 保存用户态栈指针：记录用户态栈指针，从而便于异常处理完成后正确恢复用户态栈和状态。

7. 从 U 态进入 S 态的切换发生在 ecall 指令。