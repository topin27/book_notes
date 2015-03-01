--------------------------------------------------------------------------------

## 13. The Virtual Filesystem

### Filesystem Abstraction Layer


### Unix Filesystem

Historically, Unix has provided four basic filesystem-related abstractions: 
file, directory entries, inodes, and mount points.

A path example is "/home/wolfman/butter" -- the root directory /, the director-
ies "home" and "wolfman", and the file "butter" are all directory entries, 
called *dentries*. In Unix, directories are actually normal files the simply 
list the files contained therein.

Unix systems separate the concept of a file from any associated information 
about it, such as access permissions, size, owner, creation time, and so on. 
This information is sometimes called *file metadata* and is stored in a separ-
ate data structure from the file, called the *inode*.

All this information is tied together with the filesystems own control informat-
ion, which is stored in the *superblock*. Superblock includes information about 
both the individual files and the filesystem as a whole.

Even if a filesystem does not support distinct inodes, it must assemble the 
inode data structure in memory as if it did. Or if a filesystem treats directo-
ries as a special object, to the VFS they must represent directories as mere 
files.


### VFS Objects and Their Data Structures

The VFS is object-oriented. The four primary object types of the VFS are:
	* The *superblock* object, which represents a specific mounted 
	  filesystem.

	* The *inode* object, which represents a specific file.

	* The *dentry* object, which represents a directory entry, which is a 
	  single component of a path.

	* The *file* object, which represents an open file as associated with 
	  a process.

An operations object is contained within each of these primary objects. These 
objects describe the methods that the kernel invokes against the primary 
objects:
	* The `super_operations` object, which contains the methods that the 
	  kernel can invoke on a specific filesystem, such as `write_inode()` 
	  and `sync_fs()`.

	* The `inode_operations` object, which contains the methods that the 
	  kernel can invoke on a specific file, such as `create()` and `link()`.

	* The `dentry_operations` object, which contains the methods that the 
	  kernel can invoke on a specific directory entry, such as 
	  `d_compare()` and `d_delete()`.

	* The `file_operations` object, which contains the methods that a proc-
	  ess can invoke on an open file, such as `read()` and `write()`.
The operations objects are implemented as a structure of pointers to functions 
that operate on the parent object.

Each registered filesystem is represented by a `file_system_type` structure. 
Furthermore, each mount point is represented by the `vfsmount` structure. Fina-
lly, two per-process structures describe the filesystem and files associated 
with a process. They are, respectively, the `fs_struct` structure and the 
`file` structure.


### The Superblock Object

The superblock object is represetned by `struct super_block`, usually correspo-
nds to the *filesystem superblock* or the *filesystem control block*, which is 
stored in a special sector on disk. Filesystem that are not disk-based generate 
the superblock on-the-fly and store it in memory. When mounted, a filesystem 
invokes `alloc_super()`, reads its superblock off of the disk, and fills in its 
`struct super_block` object.


#### Superblock Operations

The most important item in the superblock object is `s_op`.


### The Inode Object

The inode object represents all the information needed by the kernel to manipu-
late a file or directory. For Unix-style filesystems, this information is 
simply read from the on-disk inode. The inode object is represented by `struct 
inode`.

An inode represents each file on a filesystem, but the inode object is constru-
cted in memory only as files are accessed.


#### Inode Operations


### The Dentry Object

The VFS treats directories as a type of file. Despite this useful unification, 
the VFS	often needs to perform directory-specific operations, the VFS employs 
the concept of a directory entry(dentry). A *dentry* is a specific component in 
a path.

Dentries might also include mount points. In the path "/mnt/cdrom/foo", the 
components "/", "mnt", "cdrom", and "foo" are all dentry objects. The VFS 
constructs dentry objects on-the-fly, as needed, when performing directory 
operations.

Dentry objects are represented by `struct dentry`. Unlike to previous two 
objects, the dentry object does not correspond to any sort of on-disk data 
structure. The VFS creates it on-the-fly from a string representation of a path 
name.


#### Dentry State

A valid dentry object can be in one of three state: used, unused, or negative.

A used dentry corresponds to a valid inode and indicates that there are one or 
more users of the object. 

An unused dentry corresponds to a valid inode, but the VFS is not currently 
using the dentry object. Because the dentry object still points to a valid 
object, the dentry is kept aroud -- cached -- in case it is needed again.

A negative dentry is not associated with a valid inode because either the inode 
was deleted or the path name was never correct to begin with.


#### Dentry Cache

The kernel caches dentry objects in the detry cache or, simply, the *dcache*.

The dentry cache consists of three parts:
	* Lists of **used** dentries linked off their associated inode.

	* A doubly linked **least recently used** list of unused and negative 
	  dentry objects.

	* A hash table and hashing function used to quickly resolve a given 
	  path into the associated dentry object.

The dcache also provides the front end to an inode cache, the *icache*. Inode 
objects that are associated with dentry objects are not freed because the 
dentry maintains a positive usage count over the inode.

Caching dentries and inodes is beneficial because file access exhibits both 
spatial and temporal locality. File access is spatial in that programs tend to 
access multiple files in the same directory.


#### Dentry Operations

The `dentry_operations` structure specifies the methods that the VFS invokes on 
directory entries on a given filesystem.


### The File Object

The file object is used to represent a file opened by a process. The file 
object is in-memory representation of an open file.

The file object merely represents a processs view of an open file. The object 
points back to the dentry(which in ture points back to the inode) that actually 
represents the open file.

The file object is represented by `struct file`. The file object does point to 
its associated dentry object via the `f_dentry` pointer. The dentry in turn 
points to the associates inode, which reflects whether the file itself is dirty.


#### File Operations


### Data Structures Associated with Filesystem

Because Linux supports so many different filesystems, the kernel must have a 
special structure for describing the capabilities and behavior of each 
filesystem. The `file_system_type` structure accomplishes this.

There is only one `file_system_type` per filesystem, regardless of how many 
instances of the filesystem are mounted on the system, or whether the filesys-
tem is even mounted at all. Things get more interesting when the filesystem is 
actually mounted, at which point the `vfsmount` structure is created. This 
structure represents a specific instance of a filesystem -- in other wors, a 
mount point.


### Data Structures Associated with a Process

Each process on the system has its own list of open files, root filesystem, 
current working directory, mount points, and so on. Three data structures tied 
together the VFS layer and the processes on the system: `files_struct`, 
`fs_struct`, and `namespace`.

All per-process information about open files and file descriptors is contained 
in `struct file_struct`.

`struct fs_struct` contains filesystem information related to a process and is 
pointed at by the `fs` field in the process descriptor.

Per-process namespaces were added to the 2.4 Linux kernel. They enable each 
process to have a unique view of the mounted filesystems on the system -- not 
just a unique root directory, but an entirely unique filesystem hierarchy.

These data structures are linked from each process descriptor. For most proces-
ses, the process descriptor points to unique `file_struct` and `fs_struct` str-
uctures. For process created with the clone flag "CLONE_FILES" or "CLONE_FS", 
however, these structures are shared. The `namespace` structure works the other 
way around. By default, all processes share the same namespace. Only when the 
"CLONE_NEWNS" flag is specified during `clone()` is the process given a unique 
copy of the `namespace` structure. Because most processes do **not** provide 
this flag, all the processes inherit their parents namespaces.

Threads usually specify "CLONE_FILES" and "CLONE_FS" and, thus, share a single 
`file_struct` and `fs_struct` among themselves. Normal processes, on the other 
hand, do not specify these flags and consequently have their own filesystems 
information and open files tables.


--------------------------------------------------------------------------------

## 14. The Block I/O Layer

### Anatomy of a Block Device

The smallest addressable unit on a block device is a *sector*. The sector size 
is a physical property of the device.

Software has different goals and therefore imposes its own smallest logically 
addressable unit, which is the *block*. Filesystem can be accessed only in 
multiple of a block. Although the physical device is addressable at the sector 
level, the kernel performs all disk operations in terms of blocks. 
Furthermore, the kernel requires that a block be no larger than the page size. 
Therefore, block sizes are a power-of-two multiple of the sector size and are 
not greater than the page size.

Sectors, the smallest addressable unit to the device. Blocks, the smallest 
addressable unit to the filesystem.


### Buffers and Buffer Heads

When a block is stored in memory -- say, after a read or pending a write -- 
it is stored in a *buffer*. Each buffer is associated with exactly one block. 
A single page can hold one or more blocks in memory. Because the kernel 
requires some associated control information to accompany the data, each 
buffer is associated with a descriptor, which is called a *buffer head*, and 
is of type `struct buffer_head`.

Before the 2.6 kernel, the buffer head was a much more important data 
structure: it was the unit of I/O in the kernel. Not only did the buffer head 
describe the disk-block-to-physical-page mapping, but it also acted as the 
container used for all block I/O.This had two primary problems. First, the 
buffer head was a large and unwieldy data structure. The second issue with 
buffer heads is that they describle only a single buffer.


### The `bio` Structure

The basic container for block I/O within the kernel is the bio structure. 
This structure represents block I/O operations that are in flight as a list of 
*segments*. A segment is a chunk of a buffer that is contiguous in memory. By 
allowing the buffers to be described in chunks, the bio structure provides the 
capability for the kernel to perform block I/O operations of even a single 
buffer from multiple locations in memory. Vector I/O such as this is called 
`scatter-gather I/O`.


#### I/O Vectors

The `bi_io_vec` field points to an array of `bio_vec` structures. These 
structures are used as lists of individual segments in this specific block I/O 
operation.


#### The Old Versus the New

The bio structure represents an I/O operation, which may include one or more 
pages in memory. On the other hand, the `buffer_head` structure represents a 
single buffer, which describles a single block on the disk. Because buffer 
heads are tied to a single disk block in a single page, buffer heads result in 
the unnecessary dividing of requests into block-sized chunks, only to later 
reassemble them.  Because the bio structure is lightweight, it can describe 
discontiguous blocks and does not unnecessarily split I/O operations. Other 
benefits, as well:
	* The bio structure can easily represent high memory, because struct 
	  bio deals with only physical pages and not direct pointers.

	* The bio structure can represent both normal page I/O and direct I/O.

	* The bio structure makes it easy to perform scatter-gather block I/O 
	  operations, with the data involved in the operation originating 
	  from multiple physical pages.

	* The bio structure is much more lightweight than a buffer head 
	  because it contains only the minimum information needed to represent 
	  a block I/O operation and not unnecessary infomation related to the 
	  buffer itself.

The concept of buffer heads is still required, buffer heads function as 
descriptors, mapping disk blocks to pages.
 

### Request Queues

Block devices maintain *request queues* to store their pending blcok I/O 
requests. The request queue is represented by the `request_queue` structure. 
Each item in the queues request list is a single request, of type `struct 
request`. Each request can be composed of more than one bio structure because 
individual requests can operate on multiple consecutive disk blocks.


### I/O Schedulers

Simply sending out requests to the block devices in the order that the kernel 
issues them, as soon as it issues them, results in poor performance. Therefore, 
the kernel does not issue block I/O requests to the disk in the order they are 
received or as soon as they are received. Instead, it performs operations 
called *merging* and *sorting* to greatly improve the performance of the system 
as a whole.  The subsystem is called the *I/O scheduler*.


#### The Job of an I/O Scheduler

An I/O scheduler works by managing a block devices request queue. It decides 
the order of requests in the queue and at what time each request is dispatched 
to the block device. It manages the request queue with the goal of reducing 
seeks.


--------------------------------------------------------------------------------

## 16. The Page Cache and  Page Writeback

Accessing data from memory rather than the disk is much faster, and accessing 
data from processors L1 or L2 cache is faster still.

Access to a prticular piece of data tends to be clustered in time is called 
`temporal locality`.


### Approaches to Caching

#### Write Caching

Caches can implement one of three different strategies:
	* no-write: the cache simply does not cache **write** operations.
	* write-through: write operations immediately go through the cache to 
	  the disk.
	* write-back: processes perform write operations directly into the 
	  page cache, the backing store is not immediately or directly updated.

A write-back is generally considered superior to a write-through strategy 
because by deferring the writes to disk, they can be coalesced and performed in 
bulk at a later time.


#### Cache Eviction

* Least Recently Used(LRU).

* The Two-List strategy: Instead of maintaining one list, the LRU list, Linux 
  keeps two lists: the active list and the inactive list. Page are placed on 
  the active list only when they are accessd while already residing on the 
  inactive list.


### The Linux Page Cache

#### The `address_space` Object

The `address_space` structure is defined in <linux/fs.h>.


#### `address_space` Operations

The `a_ops` field points to the address space operations table, in the same 
manner as the VFS objects and their operations tables.  These function pointers 
point at the functions that implement page I/O for cached object. Each backing 
store describes how it interacts with the page cache via its own 
`address_space_operations`.

Steps involved in each, starting with a page read operation. First, the Linux 
kernel attempts to find the request data in the page cache.  The 
`find_get_page()` method is used to perform this check:
	page = find_get_page(mapping, index);
Here, `mapping` is the given `address_space` and `index` is the desired offset 
into the file, in pages. If the page does not exist in the cache, 
`find_get_page()` return NULL and a new page is allocated and added to the page 
cache.

Write operations on specific files are more complicated. Fist, the page cache 
is searched for the desired page. If it is not in the cache, an entry is 
allocated and added. Next, the kernel sets up the write request and the data is 
copied from user-space into a kernel buffer. Finally, the data is written to 
disk.


#### Radix Tree


#### The Old Page Hash Table

Prior to the 2.6 kernel, the page cache was not searched via the radix tree. 
Instead, a global hash was maintained over all the pages in the system. The 
global hash had four primary problems:
	* A single global lock protected the hash. Lock contention was quite 
	  high on even moderately sized machines, and performance suffered as a 
	  result.

	* The hash was lager than necessary because it contained all the pages 
	  in the page cache, whereas only pages pertaining to the current file 
	  were relevant.

	* Performance when the hash lookup failed was slower than desired, 
	  particularly because it was necessary to walk the chains off of a 
	  given hash value.

	* The hash consumed more memory than other possible solutions.


### The Buffer Cache

A buffer is the in-memory representation of a single physical disk block.

In earlier kernels, there were two separate disk caches: the page cache and the 
buffer cache. The former cached pages; the latter cached buffers. Today, we 
have one disk cache: the page cache. The kernel still needs to use buffers, 
however, to represent disk blocks in memory. Conveniently, the buffers 
describle the mapping of a block onto a page, which is in the page cache.


### The Flusher Threads

Dirty page writeback occurs in 3 situations:
	* When free memory shrinks below a specified threshold.

	* When dirty data grows older than a specific threshold.

	* When a user process invokes the `sync()` and `fsync()`.

The system administrator can set these values(`dirty_background_ratio, 
dirty_expire_interval, dirty_writeback_interval..`) in `/proc/sys/vm`.


#### Laptop Mode

Laptop mode is a special page writeback strategy intended to optimize battery 
life by minimizing hard disk activity and enabling hard drives to remain spun 
down as long as possible. It is configurable via `/proc/sys/vm/laptop_mode`.

Laptop mode makes a single change to page writeback behavior. In addition to 
performing writeback of dirty pages when they grow too old, the flusher threads 
also piggyback off any other physical disk I/O, flushing all dirty buffers to 
disk.


#### History: bdflush, kupdated, and pdflush


#### Avoiding Congestion with Multiple Threads
