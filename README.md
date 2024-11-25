java c
A5: FUSE File Systems
Due Dec 3 by 11:59p.m.
Points 9
Available after Nov 12 at 12a.m.
Introduction
You will be implementing a version of the Very Simple File System (https://pages.cs.wisc.edu/~remzi/OSTEP/file-implementation.pdf) (VSFS) from the OSTEP text and lectures. We will be using FUSE to interact with your file system. FUSE allows you to implement a file system in user space by implementing the callback functions that the libfuse library will call. The Tutorial 6 Exercise (https://q.utoronto.ca/courses/354291/assignments/1405721) should give you some practice with using FUSE.
Your tasks include:
Implement the code to format an empty disk into a VSFS file system by completing mkfs.c (this will be compiled into the mkfs.vsfs executable). This part of the assignment does not need FUSE at all.
Implement the FUSE functions to list the root directory, and to create, remove, read, write, and resize files, as well as get status information about files, directories, or the overall file system, by completing vsfs.c.
We are providing a set of formatted VSFS disk images so that you can work on these two parts of the assignment independently.
Using FUSE
Refer to the Tutorial 6 Exercise (https://q.utoronto.ca/courses/354291/assignments/1405721) handout for instructions on getting started with FUSE.
If you would like to learn more about FUSE:
libfuse GitHub repository: https://github.com/libfuse/libfuse (https://github.com/libfuse/libfuse)
FUSE wiki: https://github.com/libfuse/libfuse/wiki (https://github.com/libfuse/libfuse/wiki) (https://github.com/libfuse/libfuse)
FUSE API header file for the version we're using:
https://github.com/libfuse/libfuse/blob/fuse_2_9_bugfix/include/fuse.h (https://github.com/libfuse/libfuse/blob/fuse_2_9_bugfix/include/fuse.h)
Additional Setup
Unlike the passthrough file system of the tutorial exercise, your VSFS file system will operate on a disk image file. A disk image (https://en.wikipedia.org/wiki/Disk_image) is simply an ordinary file that holds the content of a disk partition or storage device.
To allow you to test your file system operations independently of your file system format code (mkfs.vsfs), we have provided some simple VSFS-formatted disk images in the course pub directory on teach.cs at /u/csc369h/fall/pub/a5/images:
vsfs-empty.disk - Small, empty file system (64 inodes, 1 MB size). Contains just root directory with '.' and '..' entries.
vsfs-empty2.disk - Another small, empty file system (256 inodes, 1MB size). Contains just root directory with '.' and '..' entries.
vsfs-maxfs.disk - Maximum size VSFS file system (512 inodes, 128 MB size). Contains just root directory with '.' and '..' entries.
vsfs-1file.disk - Small file system (64 inodes, 1 MB size) containing a single small file (only 1 data block) in the root directory.
vsfs-3file.disk - Medium file system (128 inodes, 16 MB size) containing 3 files (small - only direct blocks, medium - some indirect blocks, and maximum VSFS file size).
vsfs-42file.disk - Medium file system (64 inodes, 16 MB size) containing 42 small files (root directory inode uses multiple direct blocks).
vsfs-many.disk - Small file system (256 inodes, 2 MB size) containing lots of small files (root directory inode uses indirect block pointer).
You will need to make your own copies of these disk images to use them, since you will need to be able to write to them.
You will also need to create your own empty disk images that you can format using your mkfs program. To do so, you will run the following commands:
truncate -s  
./mkfs.vsfs -i  
The truncate command will create the image file if it doesn't exist and will set its size; mkfs.vsfs will format it into your vsfs file system (after you have completed the implementation).
Once you have a formatted vsfs disk image (one of ours, or your own) the next step is to mount your file system. We assume that you will be using /tmp/userid as in the Tutorial 6 Exercise (https://q.utoronto.ca/courses/354291/assignments/1405721) as the mountpoint, and that you will want to keep it running in the foreground so that you can see your output as it runs:
./vsfs   -f
The image file is the disk image formatted by mkfs.vsfs. Not only does vsfs mount the disk image into the local file system, it also sets up callbacks and then calls fuse_main() so that FUSE can do its work. Both vsfs and mkfs.vsfs have additional options -- run them with -h to see their descriptions.
After the file system is mounted, you can access it using standard tools (ls, cp, rm, etc.). To unmount the file system, run:
fusermount -u 
Note that you should be able to unmount the file system after any sequence of operations, such that when it is mounted again, it has the same contents.
Consistency Checkers
The name fsck comes from the common tool (https://en.wikipedia.org/wiki/Fsck) for checking the consistency of file systems in Unix-like operating systems. We provide two executables on teach.cs servers for checking the consistency of images, in the /u/csc369h/fall/pub/a5/tools/ directory:
fsck.mkfs checks that your mkfs.vsfs implementation correctly formats the disk.
fsck.vsfs checks that your code that performs various file system operations (written in vsfs.c) has not corrupted the file system.
Simplifying Assumptions
For this assignment, we make a number of simplifying assumptions:
VSFS file systems are always small enough that they can be entirely mmap'd into the vsfs process's virtual address space.
The underlying operating system will handle all write-back of dirty pages to the vsfs disk image.
If the file system crashes, the disk image may be inconsistent. Your code should not crash, but it does not need to make any special effort to maintain crash consistency.
There is a flat namespace. All files are located in the root directory and there are no subdirectories. You do not need to implement mkdir or rmdir.
All paths are absolute (they all start with '/'). If you see a path that is not absolute, or that has more than one component, you can return an error.
Understanding the starter code
First read through all of the starter code to understand how it fits together, and which files contain helper functions that will be useful in your implementation.
mkfs.c - contains the program to format your disk image. You need to write part of this program. You will also find it helpful to read the code to see how we access parts of the file system after using mmap() to map the entire disk image into the process virtual address space.
vsfs.h - contains the data structure definitions and constants needed for the file system. You may add other definitions or constants that you find useful, but you should not change the file system metadata. That is, do not add or modify fields in the superblock, inode, or direntry structures and do not change the existing definitions.
vsfs.c - contains the program used to mount your file system. This includes the callback functions that will implement the underlying file system operations. Each function that you will implement is preceded by detailed comments and has a "TODO" in it. Please read this file carefully.
NOTE: It is very important to return the correct error codes (or 0 on success) from all the FUSE callback functions, according to the "Errors" section in the comment above the function. The FUSE library, the kernel, and the user-space tools used to access the file system all rely on these return codes for correctness of operation.
Note: You will see many lines like (void)fs;. Their purpose is to prevent the compiler from warning about unused variables. You should delete these lines as you make use of the variables.
fs_ctx.h and fs_ctx.c - The fs_ctx struct contains runtime state of your mounted file system. Any time you think you need a global variable, it should go in this struct instead. We have cached some useful global state in this structure already (e.g. pointers to superblock, bitmaps, and inode table), but you may find there is additional state that you want to add, instead of recomputing it on every operation.
map.h and map.c - contain the map_file() function used by vsfs and mkfs.vsfs to map the image file into memory and determine its size. You should not need to change anything here, or make any additional calls to the map_file() function beyond what is in the starter code.
options.h and options.c - contain the code to parse command line arguments for the vsfs program. You should not need to change anything here.
util.h - contains some handy functions:
is_powerof2(x) - returns true if x is a power of two.
is_aligned(x, alignment) - returns true if x is a multiple of alignment (which must be a power of 2).
align_up(x, alignment) - returns the next multiple of alignment that is greater than or equal to x.
div_round_up(x, y) - returns the integer ceiling of x divided by y.
bitmap.h and bitmap.c - contain code to initialize bitmaps, and to allocate or free items tracked by the bitmaps. You will use these to allocate and free inodes and data blocks, so make sure you read the functions and understand how to use them. You may notice that the bitmap_alloc function can be slow, since it always starts the search for a 0 bit from the start of the bitmap. You are free to improve on this if you wish, but you do not need to do so.
You are welcome to put some of the helper functions in separate files instead of keeping everything in vsfs.c. Make sure to update the Makefile to compile those files and add/commit/push them to your git repository.
In other words, you are allowed to create your own source and header files. You can also change/update any starter code file as you see fit.
Recommended progression of your work
You should tackle this project in stages so that you can be confident that each piece works before moving on to the next step. The creation of a new file system (mkfs.c) and operations on a formatted file system (vsfs.c) can be handled independently however, so you can do Steps A and B in either order.
Step A1: Write enough of mkfs.vsfs so that you can mount the file system and check the superblock. We have implemented vsfs_statfs() in vsfs.c so that you can mount the file system in your disk image and then run stat -f on the root directory to check that the superblock is initialized correctly by your mkfs.
Step A2: Complete the implementation of mkfs.vsfs. Use the provided fsck.mkfs tool to check the correctness of the file system as you proceed.
Step B1: Write vsfs_getattr(). You have probably seen from the tutorial exercise that FUSE calls getattr() a lot. Implementing this function is the key to the rest of the operations. You will want to write a helper function that takes a path and returns a pointer to the inode (or the inode number) for the last componen代 写A5: FUSE File SystemsR
代做程序编程语言t in the path. Remember that you only need to handle paths that are of the form. "/" or "/somefile" - all paths are absolute and there are no subdirectories in our vsfs file systems.
Step B2: Write vsfs_readdir() so that you can run ls -la on the root directory when the root directory entries fit within a single data block. You should be able to mount vsfs-empty.disk, vsfs-maxfs.disk, vsfs-1file.disk and vsfs-3file.disk and list their root directories on completion of this step.
Step B3: Add the ability to create and remove empty files by implementing vsfs_create() and vsfs_unlink(). On completion of this step, you should be able to mount vsfs-empty.disk and use 'touch /tmp/userid/anewfile' to create a new empty file. The new file should be visible and the mode and timestamps should be appropriate when you 'ls -l' on the root directory. You should also be able to delete the new file you created (e.g. 'rm /tmp/userid/anewfile').
Step B4: Add the ability to grow a file up to the limit of the inode's direct block pointers, or shrink a file to empty, using truncate. Implement vsfs_truncate(). This operation shares functionality with writing to a file (increasing the file size) or removing a file (freeing all the blocks allocated to the file), so think about how you can avoid duplicating code.
Step B5: Add the ability to write to, and read from, small files, first where the data is stored in a single data block, and then when the data can be stored using only the direct block pointers in the inode. Implement vsfs_write() and vsfs_read().
Step B6: Add the ability to remove small files (where the file data uses only the direct block pointers in the inode).
Step B7: Enhance your implementation of vsfs_readdir() to list larger directories, first using just the direct blocks in the directory inode, and then using the directory inode's indirect block to read all of the directory data blocks. You should be able to mount vsfs-42file.disk (direct blocks only) and vsfs-many.disk (direct and indirect blocks) and list the root directory on completion of this step.
Step B8: Enhance your implementations of vsfs_truncate(), vsfs_write(), vsfs_read(), and vsfs_unlink() to support large files, where the indirect block in the file's inode is used to locate some of the file's data blocks.
Tip: Comment your code well. It will help you keep track of what is implemented and your understanding of how things work. Refactor your code during development (not after) and keep your functions short and well-structured.
Tip: Check that there is enough space before making any changes to the file system. This will save you from having to roll back changes if you discover that an operation cannot be completed due to lack of space.
Tip: Remember to update fields in the superblock (e.g. free_inodes, free_blocks) as you operate on the file system.
Testing and debugging recommendations
You can use standard Unix tools to manipulate directories and files in a mounted vsfs image in order to test your implementation. System call tracing with strace can help understand what syscalls they invoke to access the file system. You can, in general, use the behaviour of the host file system (ext4) as a reference - your vsfs should have the same observable behaviour for operations that vsfs needs to support. You can also write your own C programs that invoke relevant syscalls directly.
You will find it useful to run vsfs under gdb:
gdb --args ./vsfs   -d
You can then run file system operations in a separate terminal window. You can set breakpoints at the start of your FUSE callback functions (e.g. break vsfs_getattr) to help you understand what callbacks are invoked when you execute a file system operation (e.g. ls), in what order, and with what arguments. The debugger is also helpful in investigating crashes (e.g., segfaults) and stepping through the execution of the callback functions so that you can check your the state of the filesystem as the operations execute. Off-by-one errors are common but can be catastrophic when they lead to accessing the wrong block of file system metadata.
You might also find it useful to view the binary contents of your vsfs image files using xxd. See man 1 xxd for documentation.
To avoid errors when mounting the file system, make sure that the mount point is not in use (e.g. by a previous vsfs mount that didn't finish cleanly). If fusermount fails to unmount because the mount point directory is "busy", you can use the lsof command (see man lsof) to identify the process that keeps it open.
One common error message you might see when running operations on the mounted file system is "transport endpoint is not connected". This error usually means that the file system is still mounted, but the vsfs program has terminated (e.g. crashed). In this case you need to manually unmount it with fusermount -u.
One of the most common errors you might see at the early stages of the implementation is ls -la reporting an "I/O error" and displaying "???" entries. This error usually means that your getattr() callback returns invalid data in the stat structure and/or an invalid return value.
To test reads at a given file offset, you can use the tail -c command (see man 1 tail). To test either reads or writes at a given file offset, you can write your own C programs that use pread() and pwrite().
Limits and details
The maximum number of inodes in the system is a parameter to mkfs.vsfs, the image size is also known to it, and the block size is VSFS_BLOCK_SIZE (4096 bytes - declared in vsfs.h). Many parameters of your file system can be computed from these three values.
We will not test your code on an image smaller than 64 KiB (16 blocks) with 4 inodes. You should be able to fit the root directory and a non-empty file in an image of this size and configuration. You shouldn't pre-allocate additional space for metadata (beyond the fixed metadata defined for VSFS, the space needed to store the inode table and the root directory) in your mkfs.vsfs implementation. Indirect blocks should only be allocated on demand, when a file or directory grows large enough to need it.
The maximum path component length is VSFS_NAME_MAX (252 bytes including the null terminator). This value is chosen to fit the directory entry structure into 256 bytes (see vsfs.h). Names stored in directory entries are null-terminated strings so that you can use standard C string functions on them.
The maximum full path length is _POSIX_PATH_MAX (4096 bytes including the null terminator). This allows you to use fixed-size buffers for operations like splitting a path into a directory name and a file name.
The maximum file size is dictated by the number of direct block pointers in a vsfs inode (VSFS_NUM_DIRECT) and the number of block pointers in an indirect block (VSFS_BLOCK_SIZE / sizeof(vsfs_blk_t)).
The number of directory entries is limited by the maximum number of directory entry data blocks (same as the limit on file blocks).
The number of blocks in your file system is limited by the number of bits in a single VSFS block, since we use only 1 block for the data bitmap.
You can assume that read and write operations are performed one block at a time. Each read() and write() call your file system receives will only cover a range within a single block. NOTE: this does not apply to truncate() - a single call needs to be able to extend or shrink a file by an arbitrary number of blocks.
The size of a directory (i.e., directory inode's i_size field) should always be a multiple of the block size. The size of the root directory should be VSFS_BLOCK_SIZE after formatting the disk into a vsfs file system. When that block fills up and another block is added to the directory, the size should become 2*VSFS_BLOCK_SIZE , and so on.
Set the .ino field to VSFS_INO_MAX for directory entries in a block that are not in use (either because the block was just allocated to the directory, or because a file was deleted).
The inode's i_blocks field is the number of data blocks allocated to the file or directory. This count does not include the indirect block, if one is being used.
Sample disk configurations that must work include:
64KiB size and 4 inodes
64KiB size and 16 inodes
1MiB size and 64 inodes
128MiB size and 512 inodes
We will not be testing your code under extreme circumstances so don't get carried away by thinking about corner cases. However, we do expect you to properly handle "out of space" conditions in your code. Any operation that cannot be completed because there are not enough free blocks or inodes must be cleanly aborted - no blocks or inodes can "leak" in the process. The simplest way to ensure this is to check that there is enough space to complete the operation before modifying any file system metadata. The formatting program (mkfs) must also check that the image file is large enough to accommodate the requested number of inodes.
Other implementation notes:
Although the "." and ".." directory entries can be manually listed by the vsfs_readdir() callback (as in the starter code), you should create actual entries for these when you initialize the root directory in mkfs().
The only timestamp you need to store for each file and directory is mtime (modification time) - you don't need to store atime and ctime. You can use the touch command to set the modification timestamp of a file or directory to the current time.
Any data and metadata blocks (other than the fixed metadata) should only be allocated on demand.
Read and write I/O should be performed by reading/writing the virtual memory where the disk image is mmap'd. It should NOT be performed byte-by-byte (which is very inefficient); use memcpy().
Your implementation shouldn't use any floating point arithmetic. See the helper functions in util.h - if you need other, similar, functions (like floor), they can also be easily implemented using integer arithmetic.
Documentation
It is recommended that you include a README.txt file that describes any aspects of your code that do not work well. Code that works well and implements a subset of the functionality will get a higher mark than code that attempts to implement more functionality but doesn't work.
What to submit
Add all the starter files from MarkUs to your A5 repository. Also add to your repository all additional source code files that you create as part of your implementation.
Your A5 repository must contain all the files necessary to compile and run mkfs.vsfs and vsfs. It may include a README file as described above. Do NOT add and commit virtual machine images, executables, .o or .d files, disk image files, or any other unnecessary files - you will lose marks if you do submit those. You are welcome to commit test code and other text files. You should use a .gitignore file to help ensure you only commit and push files you should.





         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
