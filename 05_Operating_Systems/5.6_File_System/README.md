# 1.6 File System

## File System Structure

### Disk Layout
- **Boot Block**
  - Bootstrap loader
  - System initialization
  
- **Super Block**
  - File system metadata
  - Block size, inode count
  - Free block information
  
- **Inode Table**
  - File metadata storage
  - Inode allocation
  
- **Data Blocks**
  - Actual file content
  - Directory data

### Inode (Index Node)
- **Metadata**
  - File size
  - Owner (UID, GID)
  - Permissions
  - Timestamps (access, modify, change)
  - Link count
  
- **Data Pointers**
  - Direct pointers
  - Indirect pointers
  - Double/Triple indirect

### Directory
- **Directory Structure**
  - Name to inode mapping
  - Directory entries
  
- **Directory Types**
  - Single-level directory
  - Two-level directory
  - Tree-structured directory
  - Acyclic graph directory

## File Allocation Methods

### Contiguous Allocation
- **Advantages**
  - Simple implementation
  - Fast sequential access
  
- **Disadvantages**
  - External fragmentation
  - File growth limitation

### Linked Allocation
- **Pointer-Based**
  - Each block points to next
  - No external fragmentation
  
- **FAT (File Allocation Table)**
  - Centralized linking table
  - Random access improvement

### Indexed Allocation
- **Index Block**
  - Pointers to data blocks
  - Random access support
  
- **Inode Structure**
  - Multi-level indexing
  - Supports large files

## File Caching

### Disk Cache
- **Buffer Cache**
  - Recently accessed blocks
  - Read/write optimization
  
- **Page Cache**
  - Memory-mapped files
  - Unified caching

### Write Policies
- **Write-Through**
  - Immediate disk write
  - Data consistency
  - Lower performance
  
- **Write-Back**
  - Delayed disk write
  - Better performance
  - Risk of data loss
  - Periodic flush

## File System Operations
- [ ] File creation and deletion
- [ ] Directory operations
- [ ] File reading and writing
- [ ] Seeking and positioning

## Study Materials
- [ ] File system layout diagrams
- [ ] Inode structure examples
- [ ] Allocation method comparisons
- [ ] Caching strategy analysis

## Practice Problems
- [ ] Calculate file block locations
- [ ] Implement simple file system
- [ ] Analyze file system performance
- [ ] Design caching strategies
