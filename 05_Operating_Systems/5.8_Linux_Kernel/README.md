# 1.8 Linux Kernel (Basics)

## Linux Architecture

### User Space vs Kernel Space
- **User Space**
  - Application programs
  - User libraries
  - Limited privileges
  - Protected memory
  
- **Kernel Space**
  - Core kernel code
  - Device drivers
  - Full hardware access
  - Privileged operations

### System Call Interface
- **Purpose**
  - User-to-kernel communication
  - Controlled access to resources
  
- **Mechanism**
  - Software interrupt (trap)
  - Mode switch (user → kernel)
  - Parameter passing
  
- **Common System Calls**
  - Process: fork(), exec(), exit()
  - File: open(), read(), write(), close()
  - Memory: mmap(), brk()
  - IPC: pipe(), socket()

## Major Kernel Subsystems

### Process Scheduler
- **CFS (Completely Fair Scheduler)**
  - Default Linux scheduler
  - Red-black tree structure
  - Virtual runtime tracking
  
- **Real-Time Scheduler**
  - SCHED_FIFO
  - SCHED_RR
  - Priority-based
  
- **Components**
  - Task struct (process descriptor)
  - Run queue
  - Load balancing

### Memory Manager
- **Virtual Memory Management**
  - Page tables
  - Memory mapping
  - Demand paging
  
- **Slab Allocator**
  - Kernel object caching
  - Memory pool management
  
- **Components**
  - Buddy system
  - Page cache
  - Swap management

### VFS (Virtual File System)
- **Abstraction Layer**
  - Unified file interface
  - Multiple file system support
  
- **Key Structures**
  - superblock
  - inode
  - dentry (directory entry)
  - file
  
- **Supported File Systems**
  - ext4, XFS, Btrfs
  - NFS, CIFS
  - proc, sys, dev

### Network Stack
- **Protocol Layers**
  - Socket layer
  - TCP/UDP layer
  - IP layer
  - Device driver layer
  
- **Components**
  - Socket buffer (sk_buff)
  - Routing table
  - Netfilter (iptables)
  
- **Features**
  - Zero-copy networking
  - TCP offload
  - Network namespaces

## Kernel Features

### Loadable Kernel Modules
- **Dynamic Loading**
  - Runtime module insertion
  - No reboot required
  
- **Module Operations**
  - insmod / modprobe
  - rmmod
  - lsmod

### Process Management
- **Task Struct**
  - Process control block
  - Scheduling info
  - Memory descriptors
  
- **Process States**
  - TASK_RUNNING
  - TASK_INTERRUPTIBLE
  - TASK_UNINTERRUPTIBLE
  - TASK_ZOMBIE

## Study Materials
- [ ] Linux kernel architecture diagram
- [ ] System call flow chart
- [ ] Kernel compilation and customization
- [ ] Module development basics

## Practice Problems
- [ ] Trace system call execution
- [ ] Write simple kernel module
- [ ] Analyze kernel source code
- [ ] Understand /proc filesystem
