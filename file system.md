File System
===========

A file system is an organized collection of files and directories. 

The kernel provides system calls to store and retrieve data from files, as well as methods to manipulate and navigate the hierarchical structure of a file system.

The storage space on a file system is allocated in logical blocks (a multiple of contiguous physical blocks on the device).

A file system is contained in a disk partition. A disk device may contain one or more partitions. The exact implementation details vary between each file system type but, in general, they are organized with the following parts: 

- boot block: contains information needed for boot (all file systems contain a boot block, but only one boot block is needed by the operating system)
- superblock: contains information about the filesystem itself, including the size of the inode table, the size of logical blocks, the size of the filesystem
- inode table: the main data structure, see below
- data blocks

Inode table
===========

Each file or directory in the file system has a unique entry in the inode table.
 
An inode contains, in general, the following information:

- file type (whether a regular file, a directory, a symlink, ...)
- ownership information (user ID and group ID)
- access permissions (read-write-execute permissions for user-group-others)
- timestamps (last access, last modification and last status change - creation time is not saved)
- number of hard links to the file
- size in bytes
- number of blocks
- pointers to data blocks

Note that the name of the file is not in the inode (in fact, the file name is just an entry in the parent directory).

File attributes
===============

Directories, hard links and soft links
======================================

A directory is a special file type, logically organized as a table of mappings from filenames to inode numbers.

Because the name of a file is not in its inode, multiple names (in the same or in different directories) referring to the same inode can be created. These are hard links. Each new link increments the hard link counter in the inode by one. The inode and data blocks are deallocated only when the hard link count reaches zero. Hard links suffer some fundamental limitations:

- hard links must reside on the same file system of the file they refer to (think of hard links as different names pointing to the same inode number: each inode number is, by definition, unique within a file system, but the same inode number will refer to a different file on another file system)
- hard link cannot point to a directory - circular links could be created otherwise (for example, a link to a parent directory)

The problem with circular links is that when traversing a directory hierarchy (think a "find") there is no way to detect you are looping without keeping track of inode numbers as you traverse. In general, every path must be canonicalized (i.e. resolving links), and having hard links to directories (parent) would generate more paths for the same file. Also consider that a filesystem is a DAG, hard links would let a sub-directory name be a link to a parent directory, creating a file system “cycle” or loop.

A symbolic link is a special file type whose data is the name of another file. The pathname to which the link refer may be absolute or relative. They are not included in the (hard link) count of the file to which they refer, so if the target file is removed, the link itself continues to exist and cannot be referenced (dangling - they can also be created dangling). They don’t suffer the limitation of hard links (cross filesystem boundaries and link to directories - which can be detected and skipped when traversing: softlinks are resolved first to a canonical path, that is a minimal path without the links). The owner, group, and permissions of the symlink inode itself are ignored and don’t matter. The only thing that matters in a symlink is the target name.
