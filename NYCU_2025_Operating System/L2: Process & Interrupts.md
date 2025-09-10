# L2: Process & Interrupts

## What is a Process

### Process
* An active program and related resources, such as
   * Processor state
   * Memory address space

### Process Descriptor
* Contains all information about a process
* 名稱用法:
   * In Linux: Process Descriptor ,`struct task_struct`
   * General name: Process Control Block
* `task_struct`包含哪些資料
   * `pid_t pid`: Process ID，唯一識別值
   * `long state`: 程序狀態（running, sleeping, zombie 等）

 ### Task Lists
 * A list of **process descriptors** in a circular doubly linked lists
    * Circular doubly linked list（循環雙向鏈結串列）
       * 雙向：方便往前/往後找其他 process
       * 循環：不需額外處理 list 的結尾
      
<img width="343" height="281" alt="image" src="https://github.com/user-attachments/assets/d2265893-ee81-43e1-b4e1-82d40ad57cf8" />    
* process: 都有一份 task_struct 代表自己
* task_struct: 是一種儲存 process 所有狀態的資料結構，又稱為 Process Descriptor 、 Process Control Block
* task list: 由一堆 `task_struct` 組成，是一個雙向循環串列

***

## How Kernel Accesses a Process Descriptor
* Kernel Stack: Linux kernel use this **per-thread memory aera** to store (每個 thread 在 kernel space 都有一塊 kernel stack)
    * Intermediate information: 執行期間的「中間資訊」，例如函式呼叫的 return address、參數、區ˋ育變數
    * Metadata block(‵thread_info`): 這個結構存放 **指向 process descriptor,task_struct**的指標
* `thread_info`:
    * stored at the bottom of the kernel stack (Easier to Access)
    * Size of kernel stack: static 2 ~ 3 pages
    * Dereferences the task member of `thread_info` to **get the pointer of the `task_struct`**
 
上面白話來說就是:  
1. 每個 thread 在 kernel space 都有一塊 kernel stack
2. stack 的最底部放了一個小結構叫 `thread_info`
3. `thread_info` 裡有個欄位 task，它是一個指向 `task_struct` 的 pointer
4. 所以只要知道目前執行 thread 的 **kernel stack**，就能進一步找到他的 **`thread_info`**，也就得知該 ``task_struct` 的 pointer
因此就能讓 Kernel access the Process Descriptor

<img width="1248" height="544" alt="image" src="https://github.com/user-attachments/assets/4567ea02-8646-4e74-a0b8-1151bec57a4f" />

***

## Process State
<img width="335" height="281" alt="image" src="https://github.com/user-attachments/assets/81370a39-adb6-4d57-9ac5-68188ca53ce6" />  

* The **state field** of the process descriptor **describes the current condition of the process** 
    * `Task_Running`: Runnable, either **running** or **ready but not running**
    * `Task_Interruptible`: Sleeping (i.e., blocked), waiting for some condition
    * `Task_Uninterruptible`: Sleeping, does not wake up even receivesa signal
    * `__Task_Stopped`: Execution has stopped. (SIGCONT to resume)
    * `__Task_Traced`: Processes traced by a Debugger

### Process State Diagram
<img width="1129" height="916" alt="image" src="https://github.com/user-attachments/assets/d10ff4ad-aaeb-4f82-8078-1b2862d1e571" />    

1. `fork()` 產生新的 process
2. `TASK_RUNNING` (ready but not running)
    * 表示這個 process 雖然準備好執行，但目前還沒被分配到 CPU
    * 在這個狀態中，它會被放在一個「run queue」中，等待 scheduler 來挑選
3. 被排程器挑選執行 → `TASK_RUNNING` (running)
4. 搶佔（Preemption）→ 回到 ready queue
5. 自願睡眠 → `TASK_INTERRUPTIBLE` / `TASK_UNINTERRUPTIBLE`
    * 執行中的 process 如果需要等待某個事件（例如 I/O、lock、訊號等），就會呼叫 sleep 相關函式，此時它會進入「等待」狀態
6. 事件發生，喚醒 → 回到 `TASK_RUNNING` (ready)
7. 終止 → `do_exit()` → 結束生命週期

***

## The Process Family Tree

* The first process: All processes are descendants of the **init process (PID 1)**
* How to accesses one’s parent or children: Use the **parent or the children pointer** provided by the process descriptor 
* How to access other processes: By traversing through the **`task_list`**

<img width="353" height="286" alt="image" src="https://github.com/user-attachments/assets/045a7bb0-6ce2-41e5-b1de-a47d79bf87a2" />

### 白話文版本
1. 所有 processes 的始祖是 `init process`（PID 1）
2. 如何找到自己的 parent / children:   
每個 process 都有一個資料結構叫 task_struct（即 Process Descriptor）。裡面會記錄與其他 processes 的關係
3. 如何存取其他 process？  
所有的 process 都會被加入到一個雙向環狀串列：`task_list`。可以巡覽 `task_struct` 中的 tasks 欄位，來遍歷整個系統的所有 process。

***
## Process Creation
