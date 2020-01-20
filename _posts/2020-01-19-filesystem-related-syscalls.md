---
layout: post
title: Filesystem related system calls
categories: Filesystem
tags: [filesystem, system call, linux]
---

Filesystem is an important subsystem of the linux kernel. I want to share this post to clarify and summarize some important filesystem concepts.  Filesystem related system calls is the interface that kernel exposed to programmers to hide the complexity of VFS and the underneath file system.  By understanding those system calls can help us know more details about what's under the hood.

File system calls can be categorized as several classes as follows.

## File operations & Directory operations

File operations include calls such as **OPEN**, **CLOSE**, **CREAT**, **RENAME** are easy to link with the corresponded glibc calls.  Directory operations include calls such as **MKDIR**, **RMDIR**, **GETCWD**, **CHDIR**, **CHROOT**, **GETDENTS**.

It is worth to talk more about the **GETDENTS** (more familiar calls are glibc **opendir**, **closedir**, **readdir**) call to better understand the VFS in linux kernel.  For VFS, there are several important structures as **File**, **Dentry** and **Inode**.  We know that only **Inode** has on-disk structure that is structured by different the physical file systems.  **Inode** structure itself can give the whole picture of the files and directories in the filesystem as follows.

Each inode corresponds to a file or directory. inode don't contain variable length filename, only contains metadata and the *physical block number* of the *data block* so it can be fixed length structure.  In the *data block*, it contains either file contents for files or relative filename and corresponded inode number for directories.  You can think of inode as a big linked list structure on disk.  If you understand this, you will naturally come up with this question, why we need separate file and directory operations?  Since we can just expose a simple set of file operations to both files and directories. (For directories, we can still use read to get the data block to get the filename and inode numbers to traverse the directory tree.  For each file, we can traverse the directory tree to finally get the inode number and further get the data block).  However, this will impose significant performance overhead.

Consider you want to read file */usr/sbin/time*, you need to get the inode number of root i.e. */* from **superblock** first, and then read the data block to get the inode number of *user*.  So on so forth, until you finally get the data block of */usr/sbin/time*.  This includes multiple disk I/Os to read inodes and data blocks corresponding to the file.  Naturally, caching will come to your mind to solve this issue.  What you need to be cached is a mapping between file name (either file or directory) and inode number.  This is how linux actually do in the VFS by employing **dentry** (directory entry) and corresponded **dentry cache**.  Detailed **dentry** structure can be referenced [here](http://books.gigatux.nl/mirror/kerneldevelopment/0672327201/ch12lev1sec7.html).  Noted that dentry didn't keep a mapping of full path name and inode, but use the basename and parent dentry address to do the hashing for cache lookup.  Given a full path, you still need several hash lookups to finally find the inode for that path (if the dentry is in cache).  Probably this is designed to save some memory. (time to trade off some space).  Besides dentry cache, VFS also employ **inode cache**, this is easier to understand as a in-memory cache for on-disk structure.  Another important component for VFS is **page cache**, this is a different story.

## Link operations

Such as **LINK**, **SYMLINK**, **UNLINK**, **READLINK** are used for link operations.  This is related to the linux link concept.  We don't discuss in detail here.

## File attributes

File attributes are metadata (such as permission and dates information) for a file.  Calls such as **STAT**, **CHMOD**, **CHOWN**, **UTIMES**, **SETXATTR**, **GETXATTR** are ways to read and modify basic and extend file attributes.  Underneath what it does is basically read/write the inode data (block backed).

## File descriptor manipulations

File descriptor manipulations calls like **IOCTL**, **DUP**, **FLOCK** have there special use. For example **IOCTL** are used for some device driver communication.

## Read/Write/Seek/Sync

These are most important calls that actually access the data blocks of a file (or directory). **READ/PREAD**, **WRITE/PWRITE**, **LSEEK** are easy to link with glibc calls. **FSYNC**, **MSYNC**, **SYNC** are more file system specific calls, sometimes you don't trust them.

## Asynchronous I/O

Linux aio is linux's way to do **real** asynchronous I/O compared to user space aio using thread (posix aio).  Related calls are **IO_SETUP**, **IO_DESTROY**, **IO_SUBMIT**, **IO_CANCEL**.

## I/O Multiplexing

**SELECT**, **POLL**, **EPOLL** are essentially blocking I/O. Because all of them will block the process until there are I/O events or timeout.  The benefits of I/O multiplexing compared to multi-threads, multi-processes are lowing the system overhead (no need for system resources for threads/processes).  

### references

[http://linasm.sourceforge.net/docs/syscalls/filesystem.php](http://linasm.sourceforge.net/docs/syscalls/filesystem.php)

[https://bean-li.github.io/vfs-inode-dentry/](https://bean-li.github.io/vfs-inode-dentry/)