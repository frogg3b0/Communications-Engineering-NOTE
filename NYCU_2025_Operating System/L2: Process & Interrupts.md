
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
這張投影片講的是 Linux 中 Process Creation（程序建立） 的過程，尤其是搭配 Copy-on-Write (CoW) 技術來提升效能的實作方式。

### Process Creation = `fork()` + `exec()`
* `fork()`:
   * Linux 中要產生一個新的程序（process）時，會透過 `fork()`。這個動作會建立一個**與父程序相同的子程序**
   * 採用這種機制是因為效率高、實作簡單
   * 雖然說是「複製父程序」，但有幾個重要資訊是「不同的」
      * `PID`（Process ID): 每個 process 都要有唯一的識別碼。
      * `Page Table`: 會建立新的，但指向同樣的實體頁面
      * `Page Frames`
* `exex()`: 常見用法為先用 `fork()` 建立子程序，再用 `exec()` 載入真正要執行的程式
* Challenge: 一口氣從 parent process 複製所有資源會非常耗時間，於是就有了 CoW 來解決上述問題

### Copy-on-Write (CoW)
* 一開始父子 process 共用一份記憶體（ read-only），節省空間與複製時間。
* 直到其中一方要修改記憶體時，才會真的分配新的 page frame 並複製資料（觸發 page fault）
* 直到真正需要寫入才複製，這也是「延遲複製」的精神
  
### 正常父行程 P1 的記憶體存取行為
<img width="2048" height="620" alt="image" src="https://github.com/user-attachments/assets/d7b7235c-3c4c-4c91-9c25-b04e7f46fe0a" />  

CPU 想要存取一個資料時
1. 需要先去查 page table，透過虛擬位址 VA ，把 VA 映射成對應的 實體位址 PA
2. 再透過 PA 去 RAM (page frame) 存取資料

### `fork()` + write → Copy Page Table
<img width="2048" height="584" alt="image" src="https://github.com/user-attachments/assets/c1d76550-841e-4b9c-8125-0386737e90de" />  

1. 當我們呼叫 `fork()` 時，系統會建立一個新的 process（P2），它會 複製 P1 的 page table，但實體資料還沒複製
2. 這時候兩個行程 P1 和 P2 都指向相同的 page frame ，並且設定為 read-only

### Copy-on-Write (CoW) 實際發生
<img width="2048" height="635" alt="image" src="https://github.com/user-attachments/assets/844bd39f-9181-4d25-ba8a-8e1718581b50" />  

1. 當 P2 想要**寫入記憶體**時，因為該頁是 read-only 的，就會觸發 page fault
2. 作業系統這時候才會真正複製一份 page frame給 P2，這就是 Copy-on-Write

### Linux Implementation: `copy_process()`
1. Allocate a new kernel stack (e.g., `thread_info`) and process descriptor (`task_struct`)
    * 每個 process 在 Linux 核心中都會被表示成一個 `task_struct` 結構
    * 此結構中包含了該 process 的所有必要資訊：狀態、排程資訊、記憶體位置、PID、開啟的檔案、signal handler 等
    * 為了管理 process 的 kernel stack，Linux 還會為每個 process 配一個 `thread_info` 結構，並與 `task_struct` 共用相同的 page
  
2. Copy or set fields in the process descriptor
    * Copy: 有些欄位是直接複製自父行程，例如 program counter, states
    * Set fields: 有些欄位需要新的設定，例如新的 Process ID (PID)
  
3. Set the state field to `TASK_UNINTERRUPTABLE`
    * 因為剛建立的 process 還沒完全初始化完成，不能被排程執行
    * 所以會先被設定為 `TASK_UNINTERRUPTABLE` 狀態，代表它目前不會被中斷或被 scheduler 切走。

4. Duplicate parent’s page table
    * Linux 使用 Copy-on-Write 技術來實作記憶體複製：**複製的是 page table而非內容**
    * 兩個 process 的虛擬記憶體初期會指向相同的 physical page（並設為 read-only），只有在 write 時才觸發 CoW

5. Duplicate (Process) or Share (Thread):

*** 

## Process vs Thread
### Thread
* 又稱為 **Lightweight Process**
* Executes within the same program in a **shared memory address space**, **shared open files**, and **shared page tables**
    * 所以，多個 thread 能夠同時操作共用的資料區（如 global 變數、heap），這就是為什麼 thread 之間需要額外處理 同步問題（race condition）
* Linux implements all threads as standard processes
    * Each thread have their own `task_struct`
       * Program Counter
       * Registers
       * Stack
       * State
    * 換句話說，在 Linux 裡，thread 只是共享資源的 process
* Linux does not provide any special scheduling or data structures to represent threads
    * Linux kernel 並沒有特別區分「這是一個 thread」或「這是一個 process」
    * 對 scheduler來說，它只看到「一堆具有 task_struct 的任務」，不在乎它們是否共享資源
    * 所以，thread 是一種語意上的概念
* Creating Threads: `clone()` is identical to `fork()`
    * `fork()`: 建立一個完整新 process（所有資源都複製，等要寫入記憶體時再 Copy-on-Write）
    * `clone()`: 依據 flag 決定要共享哪些資源
        * `clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0)`
* Kernel Threads (kernel threads 沒有自己的記憶體空間)
    * Like normal processes (schedulable and preemptable): 和一般 process 一樣，kernel thread 會被 scheduler 排程與中斷；也可被 preempt（強制換 context）
    * But, in contrast to normal processes, **kernel threads do not have an address space**
    * Kernel Threads Operate only in kernel-space and do not context switch to user-space.

### 小結論: Linux 中的 Thread 與 Kernel Thread
#### Thread
* Linux 內部將其視為普通的 process，每個 thread 都有獨立的 task_struct
* 本質是共享資源的 process：透過 `clone()` 系統呼叫並指定特定 flags
* Linux 核心 並不特別標記或調度「thread」，對 scheduler 而言所有任務都是 task_struct，thread 只是語意上的概念

#### Kernel Thread
* 和普通 process 一樣，kernel thread 也是 task_struct，同樣可被排程與中斷（schedulable & preemptable）
* 最大不同點：kernel thread 沒有自己的 address space，只在 kernel-space 執行，不會切換到 user-space
* 常用於處理核心層面的背景任務（如 I/O 處理、驅動、排程器…）
* 不會執行使用者程式碼，無法呼叫 exec()

***

## Process Termination（程序終止）
Termination occurs when **the process or C compiler call `exit()`**
* Linux 核心會進一步呼叫 `do_exit()` 來進行實際的資源釋放與後續清理
* 步驟說明（對應 `do_exit()`）
  1. Sets the PF_EXITING flag in the flags member of the task_struct
     * 當一個 process 要終止時， Linux kernel 會把它的 process 狀態做標記，以讓其他 kernel 元件知道這個 process 正在退出中，不能再進行某些操作
  2. `exit_mm()`: 釋放與此 process 關聯的 memory descriptor,讓記憶體能回收並被其他 process 使用
  3. `exit_sem()`: 將此 process 從 IPC semaphore 等候佇列中移除，防止 zombie process 阻塞 semaphore queue
  4. `exit_files()` and `exit_fs()`
     *  `exit_files()`: 釋放 file descriptor table，並將每個 file 的引用計數減 1
     *  `exit_fs(): 釋放與 file system 相關的資訊
  5. `exit_notify()`: 向 parent process 發送 `SIGCHLD`，將此 process 的 子程序重新指定父程序。
     * 目的是維持 parent-child 關係的正確性，允許 parent 呼叫 wait() 來清理資源。
  6. `schedule()`: 呼叫排程器，切換至另一個可執行的 process
  7. After **the parent retrieves the information**
     * the process’s kernel stack (including `task_struct` and `thread_info`) will be deallocated.

***
## Process Scheduling

### 如何達成 Multitasking
#### Single Processor (multiprogramming)
* 雖然只有一個 CPU，但系統快速切換不同的 process 來執行，造成「看起來像是同時在跑多個程式」
* 意思是在只有一個處理器的系統中，要實現 multitasking ，就需要靠 Time Sharing 的方式來模擬出「好像同時執行多個程式」的效果。

#### Multiple Processors
* 每個處理器可以同時跑不同的 process，是真正的硬體層並行（真正的 parallelism）

### 好的排程器應該滿足的三個目標
* Fairness（公平性): 每個 process 都應該有機會使用 CPU，不該有某個 process 一直獨佔 CPU
* Response Time（回應時間）: 應該要快速回應
* Throughput（吞吐量）: 整體系統每秒能完成多少工作（例如完成幾個 process），為系統效率的指標之一

### 根據「誰決定讓出 CPU」來分類
1. Cooperative Scheduling（協同式排程）
   * Process **主動釋放 CPU 控制權**
   * 缺點: 如果一個程式不釋放，整個系統會卡住
2. Preemptive Scheduling（搶佔式排程）
   *  Involuntarily suspend a running process (context switch)
   *  作業系統強制中斷正在執行的 process，而非 process 主動釋放
   *  具體作法是透過「context switch（上下文切換）」來轉交 CPU 給其他 process
   *  Linux 採用此機制
3. 設計 Preemptive Scheduling 的核心概念
   * Timeslice: The time a process runs before it is preempted, prevent some processes monopolizing processors

### Scheduling policy challenge
1. How to decide the timeslice
2. Which process shall be run next

***

## Considerations for Designing Policies (設計排成器的考量因素)
### Process Priority
* nice value
    * larger value represents being micer to other process
    * 解釋: nice 越小（甚至是負值）→ 越不 nice → 優先權越高 → 獲得更長的 time slice
* real-time value
    * All real-time processes have **higher priority** than normal processes
    * 解釋: 實時任務（如音訊播放、控制系統）不能被隨意打斷，因此排程器會保證它們的優先執行權

***

### Timeslice
#### Timeslice 定義
* 每個 process 在被排程上 CPU 時，最多可以連續運行的時間
* 若執行超過該時間，就會被 preempt（搶走 CPU）並 context switch
* Longer timeslice → Fewer context switches → Better Resource Utilization

#### Timeslice 的挑戰
#### Challenge 1:
* Triggers context switch frequently if all processes are **low priority**
* 解釋: 若所有 process 的 nice 值都很大（代表 priority 很低），他們的 time slice 會很小，導致頻繁 context switch

#### Challenge 2:
* **Timeslice difference is huge** for processes 
* 解釋: 意味著有些 process 一執行就是幾百毫秒，有些卻只有 5ms，造成Resource unbalance

#### CFS can solve these challenges, but O(1) cannot

***

### Process Characteristics (程序特性)
1. I/O-Bound Process
    * Spends much of time **submitting and waiting on I/O requests**
    * Example: Graphical User Interface (GUI)
    * Policy: Run more frequently and interactive, **shorter timeslice**
2. Processor-Bound Process
    * Spends much of time **executing code**
    * Example: ssh-keygen, MATLAB
    * Policy: Run less frequently, longer timeslice

### Maintenance Overhead
這部分在探討「排程器內部如何維持與選擇下一個 process」  
* How to find the next process
   1. Queue
   2. Tree
   3. Bit-map
* Runqueue 的設計選擇
   1. Global runqueue：全系統只有一個排程佇列，需加 lock 保護
   2. Per-processor runqueue：每個 CPU core 自己有一組 runqueue，減少 lock contention → 提升效能與 CPU 使用率

***

## Linux Process Scheduler
這段是在介紹 Linux 作業系統不同時期的 Process Scheduler 設計演進  

### O(N) Scheduler（before Kernel 2.4）
* All processors **share a global runqueue** (意思是所有 ready 的 process 都會放進這個 queue，並根據優先權排序)
* 缺點 1. Shall lock the runqueue when a process runs out of its timeslice
* 缺點 2. Processes shall be inserted to an **expired queue**
* Lead to scale poorly when there are tons of processes

### O(1) Scheduler（Kernel 2.5）
* 為了解決上面的 scalability 問題，Linux 在 kernel 2.5 中引入了 O(1) scheduler
* O(1) scheduler 的設計目標：快速找到目前最應該執行的 process（也就是擁有最高優先權的那一個）
* 在 O(1) scheduler 中，每一個 priority 值（0～139） 都有對應的一個佇列（queue）
* 也就是一個 priority 對應一個 queue，queue 裡面放的是擁有這個 priority 的 process。
#### bitmap 優先權索引
1. Check bitmap to get the target priority value (leftmost)
   * 查詢 Bitmap，找出最左邊 bit = 1 的位置，所以這步是「**選出目前最高優先級的非空 queue**」
3. Use the priority value to get (dequeue) the next process (FIFO) in the active array
   * 拿到那個 priority 值之後，就知道要去哪一個 queue，我們就「dequeue」：**把排第一的 process 拿出來執行**
3. Recalculate priority and enqueue to the expired bitarray
   * 當一個 process 用完它的 time slice 後，OS 會依據它的使用狀況（執行時間、priority等）重新計算它的 priority
缺點: Low interactive performance (Low priority processes)

<img width="2349" height="386" alt="image" src="https://github.com/user-attachments/assets/9dc68871-5ca2-40e1-aaab-374613b512a9" />  
* 左圖代表我們的  bitmap ，會在這邊找到優先度最高的 queue
* 右圖是各個 queue 裡面蘊含的 process，我們會從中拿出第一個 process 來執行 (dequeue)

### O(log(N) Scheduler (Kernel 2.6)
* Name: Completely Fair Scheduler（CFS）
* Goal: more interactive(像 GUI 需求) and **more Fair than O(1)**
* Concept 1: each process would **receive 1/n** of the processor’s time (fair share)
* Concept 2: Schedule processes for infinitely small durations (意思是如果要做到理想化的多工，應該是每個 process 都能「無限小時間」地不斷被輪流執行)

#### CFS 的運作邏輯
