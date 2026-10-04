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



