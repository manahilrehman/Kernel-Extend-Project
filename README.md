# Kernel-Extend-Project


## Student Information

**Name:** Manahil Rehman
**Roll No:** BCS-F24-E09

**Project:** Operating System/ Computer Archeitecture Kernel Extension

**Base Kernel:** xv6-riscv

---

## Selected Modules

The following modules have been selected for my kernel extension project.

### Part 1: OS Modules — Multitasking

| # | Module | Subsystem | Syscalls | Test Program | Difficulty |
|---|--------|-----------|----------|--------------|------------|
| 1 | **Process Pause/Resume** | Process States | `pause_proc()`, `resume_proc()` | `pausetest` | Easy-medium |
| 2 | **Shared Memory** | Pages Mapped into Two Processes | `shmget()`, `shmat()`, `shmdt()` | `shmtest` | Medium |
| 3 | **File Descriptor Info** | Open-File Table | `fdinfo()` | `fdinfo` | Easy |
| 4 | **Kernel Log Buffer (dmesg)** | Kernel Logging | `klog()` | `dmesg` | Easy-medium |
| 5 | **User IDs and File Permissions** | Access Control | `setuid()`, `getuid()`, `chmod()` | `permtest` | Medium |

### Part 2: Architecture Modules — RISC-V

| # | Module | Subsystem | Syscalls | Test Program | Difficulty |
|---|--------|-----------|----------|--------------|------------|
| 6 | **Real-Time Clock** | Memory-Mapped I/O (QEMU RTC) | `gettime()` | `datetest` | Easy-medium |
| 7 | **Virtio Disk Statistics** | Disk Driver | `diskstat()` | `diskstat` | Easy |
| 8 | **Lock Statistics** | Atomics and Spinlocks | `lockstat()` | `lockstattest` | Medium |
| 9 | **CSR Dump** | Supervisor CSRs (`sstatus`, `satp`, `scause`, `sie`) | `csrdump()` | `csrdump` | Easy |
| 10 | **TLB Flush Counter** | `sfence.vma` and Address Translation | `tlbstat()` | `tlbstat` | Medium |

---

## Module Reservation

**These 10 modules are reserved for my kernel extension project.**

Implementation, testing, documentation, and architectural diagrams will be added in later stages.



## Detailed Specifications & Architecture Diagrams

---

### Module 1: Process Pause/Resume

#### Overview & Purpose
This module allows user space programs to temporarily pause and resume a target process execution using custom kernel system calls. It helps in controlling process scheduling and execution flow directly from user space.

#### Key Features & Functionality
- **Pause Process (`pause_proc(pid)`):** Changes the process state from running/runnable to a stopped/suspended state (`TASK_STOPPED`).
- **Resume Process (`resume_proc(pid)`):** Wakes up a suspended process and restores its state back to runnable (`TASK_RUNNING`).
- **Safety Checks:** Verifies process permissions and checks if the PID exists before modifying process control block (PCB) states.

#### Implementation Details
- **Subsystem:** Process States & Scheduler
- **Syscalls:** `pause_proc()`, `resume_proc()`
- **Test Program:** `pausetest`
- **Difficulty:** Easy-medium

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor User as User Program (pausetest)
    participant Syscall as System Call Handler
    participant Kernel as Kernel Scheduler / PCB
    participant Proc as Target Process

    alt Pause Process Flow
        User->>Syscall: Call pause_proc(PID)
        Syscall->>Kernel: Locate task_struct by PID
        Kernel->>Proc: Change State to TASK_STOPPED
        Kernel-->>User: Return Success (0)
    else Resume Process Flow
        User->>Syscall: Call resume_proc(PID)
        Syscall->>Kernel: Locate task_struct by PID
        Kernel->>Proc: Change State to TASK_RUNNING / Wake Up
        Kernel-->>User: Return Success (0)
    end


---

### Module 2: Shared Memory

#### Overview & Purpose
Provides a mechanism for two or more processes to share a common memory segment for high-speed inter-process communication (IPC).

#### Key Features & Functionality
- **Memory Mapping (`shmget`, `shmat`):** Allocates shared physical memory pages and maps them into virtual address spaces of calling processes[cite: 1].
- **Detachment (`shmdt`):** Unmaps the shared memory segment from process address space[cite: 1].
- **Synchronization:** Prevents race conditions during simultaneous memory reads/writes.

#### Implementation Details
- **Subsystem:** Memory Management & IPC[cite: 1]
- **Syscalls:** `shmget()`, `shmat()`, `shmdt()`[cite: 1]
- **Test Program:** `shmtest`[cite: 1]
- **Difficulty:** Medium[cite: 1]

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor ProcA as Process A
    actor ProcB as Process B
    participant Kernel as Shared Memory Subsystem
    participant RAM as Physical RAM Pages

    ProcA->>Kernel: shmget() & shmat()[cite: 1]
    Kernel->>RAM: Allocate Memory Pages
    Kernel-->>ProcA: Map Virtual Address
    ProcB->>Kernel: shmat()[cite: 1]
    Kernel-->>ProcB: Map Same Physical Address
    ProcA->>RAM: Write Data
    ProcB->>RAM: Read Data directly


---

### Module 3: File Descriptor Info

#### Overview & Purpose
Allows querying details about open file descriptors for a given process to inspect open files, file offsets, and access modes.

#### Key Features & Functionality
- **Descriptor Inspection (`fdinfo`):** Retrieves file metadata (file path, mode, flags, offset) from the open-file table.
- **Process Auditing:** Helps track resource leaks and unclosed files in running programs.

#### Implementation Details
- **Subsystem:** Open-File Table & VFS
- **Syscalls:** `fdinfo()`
- **Test Program:** `fdinfo`
- **Difficulty:** Easy

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor User as User Program (fdinfo)
    participant Syscall as System Call Handler
    participant PCB as Process Table (files_struct)
    participant VFS as Virtual File System

    User->>Syscall: Call fdinfo(fd)
    Syscall->>PCB: Retrieve File Descriptor Entry
    PCB->>VFS: Fetch File Offset, Mode & Path
    VFS-->>Syscall: Return Metadata
    Syscall-->>User: Copy Info to User Space


---

### Module 4: Kernel Log Buffer (dmesg)

#### Overview & Purpose
Exposes a custom interface to retrieve system log messages stored in the kernel's internal ring buffer, allowing user applications to inspect diagnostic outputs directly.

#### Key Features & Functionality
- **Log Fetching (`klog`):** Reads kernel print output (dmesg logs) from kernel space to user space[cite: 1].
- **Non-Destructive Reading:** Retrieves logs without clearing or resetting the underlying system ring buffer[cite: 1].
- **Buffer Safety:** Validates user-space pointer addresses and prevents buffer overflow during string copying[cite: 1].

#### Implementation Details
- **Subsystem:** Kernel Logging[cite: 1]
- **Syscalls:** `klog()`[cite: 1]
- **Test Program:** `dmesg`[cite: 1]
- **Difficulty:** Easy-medium[cite: 1]

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor User as User Program (dmesg)
    participant Syscall as System Call Handler
    participant LogBuf as Kernel Log Subsystem
    participant Buffer as Kernel Ring Buffer

    User->>Syscall: Call klog(user_buf, size)
    Syscall->>LogBuf: Validate User Buffer Address
    LogBuf->>Buffer: Fetch Log Bytes from Ring Buffer
    Buffer-->>LogBuf: Return Log Text Data
    LogBuf-->>Syscall: Copy Data to User Space
    Syscall-->>User: Return Bytes Read Count


---

### Module 5: User IDs and File Permissions

#### Overview & Purpose
Implements access control checks and credentials verification for process owner IDs and file system permissions, ensuring user separation and process security.

#### Key Features & Functionality
- **User Identity Check (`getuid`, `setuid`):** Checks and modifies running process privileges based on effective user IDs[cite: 1].
- **Permission Override (`chmod`):** Validates and changes file permissions safely across the file system[cite: 1].
- **Access Control:** Enforces restriction rules to prevent unauthorized process elevation or file access[cite: 1].

#### Implementation Details
- **Subsystem:** Access Control & Credentials[cite: 1]
- **Syscalls:** `setuid()`, `getuid()`, `chmod()`[cite: 1]
- **Test Program:** `permtest`[cite: 1]
- **Difficulty:** Medium[cite: 1]

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor User as User Program (permtest)
    participant Syscall as Syscall Handler
    participant Cred as Kernel Credential Subsystem
    participant VFS as Virtual File System

    User->>Syscall: Call setuid() / chmod()
    Syscall->>Cred: Validate Process Credentials & UID
    alt Authorized
        Cred->>VFS: Apply Permission / Identity Update
        VFS-->>Syscall: Return Success (0)
        Syscall-->>User: Operation Completed
    else Unauthorized
        Cred-->>Syscall: Permission Denied (-EPERM)
        Syscall-->>User: Return Error Code
    end


---

### Module 6: Real-Time Clock

#### Overview & Purpose
Provides a direct hardware interface with the Real-Time Clock (RTC) peripheral to fetch high-precision system time and timestamp data directly from hardware registers.

#### Key Features & Functionality
- **Hardware Time Fetch (`gettime`):** Accesses memory-mapped I/O (MMIO) registers of the physical/emulated hardware clock (QEMU RTC)[cite: 2].
- **Timestamp Formatting:** Converts raw hardware clock ticks into human-readable time structures (hours, minutes, seconds).
- **Precision Auditing:** Allows kernel-level event timing and performance benchmarking[cite: 2].

#### Implementation Details
- **Subsystem:** Memory-Mapped I/O (QEMU RTC)[cite: 2]
- **Syscalls:** `gettime()`[cite: 2]
- **Test Program:** `datetest`[cite: 2]
- **Difficulty:** Easy-medium[cite: 2]

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor User as User Program (datetest)
    participant Syscall as Syscall Handler
    participant Driver as Kernel RTC Driver
    participant Hardware as QEMU Hardware RTC

    User->>Syscall: Call gettime()
    Syscall->>Driver: Issue Read Request
    Driver->>Hardware: Read MMIO Clock Registers
    Hardware-->>Driver: Return Raw Hardware Ticks
    Driver->>Driver: Format Ticks into Date/Time
    Driver-->>Syscall: Copy Formatted Time Structure
    Syscall-->>User: Return Time Data


---

### Module 7: Virtio Disk Statistics

#### Overview & Purpose
Collects operational metrics and low-level I/O performance statistics from virtualized block storage drives (virtio-disk) to monitor disk performance and data traffic.

#### Key Features & Functionality
- **Disk Stats (`diskstat`):** Reads total read/write operation counts, processed sector counts, and queue depth status[cite: 2].
- **I/O Profiling:** Helps analyze disk throughput, latency trends, and hardware bottleneck issues[cite: 2].
- **Safe State Querying:** Directly queries kernel block layer structures without disrupting active disk I/O requests[cite: 2].

#### Implementation Details
- **Subsystem:** Disk Driver / Virtio Block Device[cite: 2]
- **Syscalls:** `diskstat()`[cite: 2]
- **Test Program:** `diskstat`[cite: 2]
- **Difficulty:** Easy[cite: 2]

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor User as User Program (diskstat)
    participant Syscall as Syscall Handler
    participant Driver as Virtio Disk Driver
    participant Disk as Virtio Block Hardware

    User->>Syscall: Call diskstat()
    Syscall->>Driver: Request Disk I/O Metrics
    Driver->>Disk: Inspect virtqueue & Sector Counters
    Disk-->>Driver: Return Read/Write Counts & Operations
    Driver-->>Syscall: Package Stats into Structure
    Syscall-->>User: Copy Stats to User Space Buffer


---

### Module 8: Lock Statistics

#### Overview & Purpose
Tracks kernel synchronization primitives (such as spinlocks and atomic operations) to monitor lock contention, hold times, and overall system synchronization health.

#### Key Features & Functionality
- **Lock Contention Tracking (`lockstat`):** Records lock acquisition counts, wait times, lock hold durations, and spin retry loops[cite: 2].
- **Deadlock Diagnostics:** Helps identify synchronization bottlenecks, thread starvation, and potential deadlock scenarios in multi-threaded execution[cite: 2].
- **Low-Overhead Metrics:** Captures lock state metrics with minimal performance overhead on kernel execution[cite: 2].

#### Implementation Details
- **Subsystem:** Atomics and Spinlocks[cite: 2]
- **Syscalls:** `lockstat()`[cite: 2]
- **Test Program:** `lockstattest`[cite: 2]
- **Difficulty:** Medium[cite: 2]

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor User as User Program (lockstattest)
    participant Syscall as Syscall Handler
    participant Profiler as Kernel Lock Profiler
    participant LockSub as Spinlock / Atomic Subsystem

    User->>Syscall: Call lockstat()
    Syscall->>Profiler: Request Lock Metrics
    Profiler->>LockSub: Query Spinlock Counters & Wait Times
    LockSub-->>Profiler: Return Contention Statistics
    Profiler-->>Syscall: Package Data into User Structure
    Syscall-->>User: Copy Lock Stats to User Buffer


---

### Module 9: CSR Dump

#### Overview & Purpose
Reads and exposes supervisor-level Control and Status Registers (CSRs) from the RISC-V processor architecture for hardware-level debugging, CPU state inspection, and system analysis[cite: 2].

#### Key Features & Functionality
- **Register Dump (`csrdump`):** Extracts raw values of key supervisor registers including `sstatus`, `satp`, `scause`, and `sie`[cite: 2].
- **Processor State Inspection:** Exposes low-level CPU interrupt flags, page table base addresses, and execution modes directly to user space.
- **Hardware Diagnostics:** Aids in debugging kernel crashes, trap handling behavior, and virtual memory configuration issues.

#### Implementation Details
- **Subsystem:** Supervisor CSRs (RISC-V)[cite: 2]
- **Syscalls:** `csrdump()`[cite: 2]
- **Test Program:** `csrdump`[cite: 2]
- **Difficulty:** Easy[cite: 2]

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor User as User Program (csrdump)
    participant Syscall as System Call Handler
    participant CPU as RISC-V CPU Core Registers

    User->>Syscall: Call csrdump()
    Syscall->>CPU: Execute inline assembly (csrr instructions)
    CPU-->>Syscall: Read sstatus, satp, scause & sie
    Syscall-->>User: Copy Register Values to User Buffer


---

### Module 10: TLB Flush Counter

#### Overview & Purpose
Monitors virtual memory address translation performance by tracking and counting Translation Lookaside Buffer (TLB) flush operations across execution cycles[cite: 2].

#### Key Features & Functionality
- **Flush Counter (`tlbstat`):** Tracks the total occurrences of memory address translation resets triggered by the `sfence.vma` instruction[cite: 2].
- **MMU Performance Tracking:** Measures the impact of page-table context switches and address translation invalidations on system performance[cite: 2].
- **Memory Diagnostic:** Assists in profiling page-table swapping latency and memory management efficiency[cite: 2].

#### Implementation Details
- **Subsystem:** Address Translation & MMU[cite: 2]
- **Syscalls:** `tlbstat()`[cite: 2]
- **Test Program:** `tlbstat`[cite: 2]
- **Difficulty:** Medium[cite: 2]

#### Module Architecture & Execution Flow
```mermaid
sequenceDiagram
    autonumber
    actor User as User Program (tlbstat)
    participant Syscall as System Call Handler
    participant MMU as Kernel MMU Subsystem
    participant TLB as TLB Hardware Controller

    User->>Syscall: Call tlbstat()
    Syscall->>MMU: Query sfence.vma Execution Counters
    MMU->>TLB: Fetch Accumulated TLB Flush Count
    TLB-->>MMU: Return Counter Data
    MMU-->>Syscall: Package Statistics
    Syscall-->>User: Copy Flush Count to User Space
