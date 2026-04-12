# 1.1 Process & Thread

## Process Concepts

### process ID (PID)

Unique Identifier: Every running program instance gets a distinct PID.
Lifecycle: PIDs are assigned upon process creation and released upon termination, meaning they can be reused.
System Initialization: The first process created by the kernel (e.g., init or systemd in Linux) typically gets PID 1.
Purpose: Primarily used for process management, debugging, resource allocation, and troubleshooting by operating systems like Windows, Linux, and macOS.

#### How to Find PIDs

Windows:
    Task Manager: Go to the "Details" tab to see the "PID" column.
    Command Prompt: Use the tasklist command.
    PowerShell: Use the Get-Process cmdlet.

Linux/Unix:
    ps: Use ps aux | grep <process_name> to find a specific PID.
    top: Displays real-time process activity, including PIDs.
    pgrep: Type pgrep <process_name> for a quick PID lookup. 

#### Process identification

-- wikipedia
In computing, the process identifier (a.k.a. process ID or PID) is a number used by most operating system kernels—such as those of Unix, macOS and Windows—to uniquely identify an active process. This number may be used as a parameter in various function calls, allowing processes to be manipulated, such as adjusting the process's priority or killing it altogether.

#### Unix like

In Unix-like operating systems, new processes are created by the fork() system call.
The PID is returned to the parent process, enabling it to refer to the child in further function calls. The parent may, for example, wait for the child to terminate with the waitpid() function, or terminate the process with kill().

There are two tasks with specially distinguished process IDs

PID 0 is used for swapper or sched, which is part of the kernel and is a process that runs on a CPU core whenever that CPU core has nothing else to do.

이름 swapper는 예전 유닉스에서 “스왑(디스크로 메모리 내쫓기)”과 엮여 있던 역사적 이름이고, 지금은 “아무 일 없을 때 도는 커널 쪽 작업”을 가리키는 말로 쓰이는 경우가 많습니다.

sched는 “스케줄러가 고른 다음에 실제로 CPU에 올라가는 그 엔티티”를 느슨하게 부르는 식의 표현으로 쓰이기도 합니다(문맥마다 조금 다름).

그 CPU 코어에서 실행 가능한(runnable) 태스크가 없으면, 스케줄러는 결국 idle을 골라서 돌립니다.
idle 안에서는 보통 전력 절감용 명령(예: x86의 halt 계열)을 쓰거나, 설정에 따라 짧게 busy-wait하는 식으로 동작합니다.

Linux also calls the threads of this process idle tasks. In some APIs, PID 0 is also used as a special value that always refers to the calling thread, process, or process group.

Process ID 1 is usually the init process primarily responsible for starting and shutting down the system.

Originally, process ID 1 was not specifically reserved for init by any technical measures: it simply had this ID as a natural consequence of being the first process invoked by the kernel.

More recent Unix systems typically have additional kernel components visible as 'processes', in which case PID 1 is actively reserved for the init process to maintain consistency with older systems.

Process IDs, in the first place, are usually allocated on a sequential basis, beginning at 0 and rising to a maximum value which varies from system to system.

Once this limit is reached, allocation restarts at 300 and again increases. In macOS and HP-UX, allocation restarts at 100.

However, for this and subsequent passes any PIDs still assigned to processes are skipped.

Some consider this to be a potential security vulnerability in that it allows information about the system to be extracted or messages to be covertly passed between processes

As such, implementations that are particularly concerned about security may choose a different method of PID assignment.

On some systems, like MPE/iX, the lowest available PID is used, sometimes in an effort to minimize the number of process information kernel pages in memory.


#### Process management

It manages the entire lifecycle of a program in execution, using Process Control Blocks (PCBs) to track states (new, ready, running, waiting, terminated)
and handles synchronization and context switching for smooth multitasking

and responsible for managing the lifecycle of processes.

A process is essentially a program in execution, and it includes the program code, its current activity represented by the value of the program counter, the contents of the processor’s registers, and the process’s variables. 
Effective process management ensures that the system’s resources are allocated efficiently and that processes are executed without interference.
This involves creating, scheduling, and terminating processes, as well as handling synchronization and communication between them.

##### Components of Process Management

1. Process Creation

Processes are created by other processes using system calls. This parent-child relationship leads to the formation of a process hierarchy. In Unix-like systems, the fork system call is used to create a new process by duplicating the current process.

The child process created by fork is an exact copy of the parent process, except for a few distinctions such as the unique process identifier (PID).

After a fork, the exec family of system calls (execl, execv, etc.) is often used to replace the child’s memory space with a new program, allowing the execution of different code.

2. Process Scheduling
