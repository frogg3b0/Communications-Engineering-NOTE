# L1 Operating Systems a Recap

## Computer Organization
### Introduction: Three main basic components
* Computing devices
* Memory device (e.g., DRAM)
* I/O device (e.g., HDD, SDD, or network device)

### Von-Neumann Architecture
[參考資料](https://zh.wikipedia.org/zh-tw/%E5%86%AF%C2%B7%E8%AF%BA%E4%BC%8A%E6%9B%BC%E7%BB%93%E6%9E%84)
1. CPU feature an **instructure** frome memory
2. Then, CPU docode and excutes the instriction
3. CPU feature next data (Automatically Increment PC)

### Challenges without Operating Systems
* CPU will be solely occupied by one process -> Process Scheduler
* CPU shall wait for I/O devices. (1 ns vs 100 us) -> Page Fault Handler & Context Switch
* CPU cannot understand ”file” in I/O devices -> Device Driver & File System

<img width="475" height="365" alt="image" src="https://github.com/user-attachments/assets/3e54274e-2156-4d24-a496-52dd8e61abfc" />

***
## Role of Operating System
### Goal
* A soltware layer stands between **applications** and **computer's hardware**
* The role of an OS is to **coordinate and manage hardware devices**

### OS's Roles
* Transparency: Users do not need to worry about hardware managements or contentions
* Resource Allocator: Manage CPU time, memory space and I/O resources
* Control Program: Prevent errors and improper use of computer

### Well-Known Components
* Process Scheduler: Fairly allocate CPU resources to application
* Memory Manager: Allocate memory, locate pages, reclaim memory
* Block I/O Layer: Manage I/O requests
* File System: Locate file contents, partiton device, control permission

<img width="586" height="363" alt="image" src="https://github.com/user-attachments/assets/7a99e26a-86f0-41dc-87c9-c6d63a2e24b8" />

### Well-Known Components
* Process Scheduler: Fairly allocate CPU resources to application
* Memory Manager: Allocate memory, locate pages, reclaim memory
* Block I/O Layer: Manage I/O requests
* File System: Locate file contents, partition devices, control permission

***

## What is Linux Kernel
### Linux Kernel
* 定義: A process manager consist of a branch of C and Assembly code
* 功能: **Process manager**
* 它不是普通的 user-level 程式，而是存在於系統的 Kernel Space，用來管理所有應用程式（user-level process）的資源與執行狀態。

### Is Linux Kernel a Process?
* NO, NOT exactly
    * 一般的程式（例如你寫的 Python 程式）是作業系統中的「process」
    * 但 Linux Kernel 並不是單純的一個 process ，它其實存在於所有 process 的 address space 中一段共通的區域

### 虛擬位址空間 (Virtual Address Space) 
*  每個 Process 都有自己的虛擬位址空間 (Virtual Address Space)
    * 作業系統會幫每一個 process 分配「虛擬記憶體空間」
    * 這樣做的目的是為了：安全性（彼此不能亂存）、隔離性（互不干擾）、簡化程式設計

### Page Frame
* 定義: 是實體記憶體（RAM）中的一個單位，大小固定，例如 4KB
* 每個 process 的虛擬記憶體會被 paging，之後再一頁一頁地對應到某個 page frame

到這邊為止，我們知道了 os 會幫 process 分配一塊 VAS，而這個 VAS 會被 paging，之後再一頁一頁的存入某個 page frame。基於這個概念可以有以下的延伸  

<img width="908" height="435" alt="image" src="https://github.com/user-attachments/assets/672007f3-1d4d-418e-adc5-5840dd1e47f6" />  

* 當 RAM 不夠時，作業系統可以把某些「目前沒在用」的記憶體頁（Page）換到磁碟（Storage）上，等到之後要用再搬回來
* Process A 的狀況:
    * Process A 的虛擬記憶體透過 Page Table 對應到實體記憶體的 Page Frame。
    * 它的每一頁都已經被放進 RAM，運作正常
* Process B 的狀況:
    * 它的 Page Table 中這一頁的對應位置是「Non-Present」的狀態，因此該 page 只好被 swap 到 Storage
    * 作業系統會用像 pte_tb_swap_entry() 這類的資料結構，記住那一頁其實現在在哪個儲存裝置的哪個位置

### 為什麼說「虛擬記憶體讓程式設計更容易」
* Process B 的程式並不知道它的資料目前已經不在 RAM ，它依然使用原本的虛擬位址，寫法不變
* 一旦它想要存取那一頁，作業系統就會：
    1. 停下來
    2. 把那頁從 Storage swap 回 RAM
    3. 更新 Page Table
    4. 再繼續執行

### 「虛擬記憶體（Virtual Memory）」如何對應到實體記憶體（RAM）

<img width="882" height="409" alt="image" src="https://github.com/user-attachments/assets/a02d8315-14fb-4242-aa7d-ef20a6bbec16" />    

#### 左邊的結構：虛擬位址空間如何透過 Page Table 對應到實體 RAM
* Process A/B: 不同的程式（每個程式有自己的虛擬記憶體空間）
* Page: 虛擬記憶體中切割出來的最小單位（例如每頁 4KB）
* Page Table: 該程式自己的表，用來記錄「每一個 Page 映射到哪個 RAM 的 Page Frame」，每個程式有自己獨立的 page table，虛擬記憶體頁面不會互相干擾
* Page Frame: RAM 中的實體頁面（每格 4KB，方便對齊、管理與交換）
* RAM: 也就是「Physical Address Space」，真實存在的主記憶體空間

#### 右邊的 Main Memory (單一個 Process 的虛擬記憶體空間)
* User Address Space
    * User Program：程式的可執行碼
    * Process Data：變數、堆疊、heap
* Kernel Address Space
    * 系統在該 process 的 VAS 保留的空間，用來儲存 OS Kernal 的程式碼與資料
    * 雖然 Kernel 的程式碼存在於所有程式的虛擬記憶體裡，但只有進入 Kernel Mode 才能存取那部分

### When Do CPUs Run Kernel
* CPUs runs the kernel only when **some special events are triggered**. Otherwise, it runs user programs or stays idle  
* Special Events
    * Hardware interrupts (Alerting signal)
        * Timer interrupt
        * I/O Interrupt (e.g., Keyboard, Hard Disks)
    * Software-generated interrupts (Traps or Exceptions)
        * Specific requests from user processes (System Call, such as Read/Write)
        * Some errors (e.g., invalid memory access or page faults)
<img width="1156" height="270" alt="image" src="https://github.com/user-attachments/assets/93e3cf5e-426d-461b-9d42-6ec8727e226f" />\

*** 

## 小結
### I. Each process shall have their own virtual address space
* 每個程式有自己的「虛擬位址空間」
* 不會直接看到其他程式的資料，防止互相干擾或資料竄改

### II. Kernel runs as part of a user process
* 每個 Process 的 Virtual Address Space 裡面除了自己的資料與程式碼區，還會有一塊保留區域
* 這塊保留區域指向的是同一個物理位址區域，也就是 **OS Kernel**
* 所以每個 Process 的 Kernel 區其實一模一樣，意思是 OS Kernel 會共用實體記憶體

### III. When Do CPUs Run Kernel
* Hardware Interrupts（硬體中斷）
    * Timer interrupt：例如 OS 定時中斷目前 process ，強制排程 context switch
    * I/O interrupt：例如鍵盤按下、硬碟完成讀取
* Software-Generated Interrupts（軟體中斷）
    * System Calls: 程式自己要求 OS 幫它做事情（例如讀檔案），就會發 trap
    * Exception: 像是「分頁錯誤 page fault」、「除以0」、「非法記憶體存取」
 
***
## System Trend – Rethinking OS
<img width="528" height="340" alt="image" src="https://github.com/user-attachments/assets/c9d9f411-c75d-435e-b9db-cccf0f0f86a4" />    

1. The device trend blurs the boundary between memory and storage
    * 傳統觀念下，DRAM 是主記憶體，SSD/HDD 是儲存
    * 現在有了 NVM（Non-Volatile Memory，非揮發性記憶體，如 Intel Optane、Z-NAND），速度逼近 DRAM，但容量又像 SSD
    * 此時會很難界定這到底是 memory or storage
2. Storage response time catches up with context switch overhead (1~10 us)
    * 現代 NVM 存取僅需幾微秒，跟 OS context switch 時間一樣快
3. The overhead of running file system becomes critical (e.g., page cache)
    * 現在的儲存設備越來越快（NVM、SSD）
    * 但在使用檔案時，OS 並不是直接存取硬碟，而是透過 **file system** 來做管理，並透過 page cache 將檔案內容暫存在記憶體中
    * 當儲存本身夠快時，反而是 file system 會變成效能瓶頸，因此需要重新思考 OS 的定位
4. Huge memory demands increases deployment cost. (Right page size)
    * 現代應用需要大量記憶體，會讓部署成本上升，因此如何選擇正確的「分頁大小（page size）」變得很重要
    * 當代系統（如 AI、雲端、資料中心）常常需要「幾十 GB～幾百 GB」記憶體，但 OS 的記憶體管理是以 page 為單位（通常一頁 = 4KB）
    * 如果每個程式的虛擬記憶體都要映射到一堆 4KB 的 page，那 page table 會非常大、管理起來耗費很多 CPU 資源

<img width="421" height="312" alt="image" src="https://github.com/user-attachments/assets/bd950f41-6af7-4b81-8007-4be0c5c6edec" />   

### User space
* Applications: 他們不能直接操作硬體，只能透過函式庫與系統呼叫 kernel
* C Library: 許多 C/C++ 程式都會經由這裡間接觸 OS 的 System Calls

### System Call
* 當應用程式需要存取硬碟、網路、記憶體等敏感資源時，必須經由 system call
* 會導致 CPU 進入 kernel mode

### Kernel Space
* Core Kernel: 最底層的核心功能，處理 context switching、中斷處理、系統時鐘管理、scheduler、虛擬記憶體與頁表管理
* Device Drivers: 負責與具體的硬體裝置溝通，把複雜的硬體控制邏輯封裝成統一 API 給 kernel 內部模組
* Hardware

***

## What topics will we cover
### 1. Process & Interrupts
### 2. Main Memory Management
### 3. I/O Devices & Device Driver
### 4. File System
### 5. Close to Applications: Graph Systems, Deep Recommendation Systems, Key-Value Databases, Virtualization Concepts
