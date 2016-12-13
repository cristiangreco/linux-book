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

The family of stat() syscalls retrieve information about a file, mostly drawn from the inode. They require execute permissions on all the directories on the path that lead to the filename, but not on the file itself.

The syscalls return a stat structure where the st_mode field is a bitmask containing file type (first 4 bits) and file permissions (bottom 12 bits).

The 9 permissions bits for user, group and other work as expected for regular files. Permissions for directories are interpreted a bit differently:

- read: the content can be listed
- write: files can be created and deleted from the directory (no need for perms on the file itself to delete it)
- execute: search perms, i.e. the files can be accessed (read inode content)

The easiest way to realize this is to think that a directory contains just a mapping of names to inodes numbers: permissions on directory allow to read, modify and follow those mappings.

Permissions checking when accessing a path is:

- if the process is privileged, access is always granted
- else if effective user-ID matches the file owner, access is granted based on file user permissions
- else if effective group-ID or any supplemental group-ID matches the file group, access is granted based on file group permissions; else access granted based on file other permissions.

Also, if the path contains a directory prefix, execution permissions on each directory is checked.

The first 3 permission bits are the set-user-ID, set-group-ID and sticky bit. The sticky bit acts for directories as a restricted-deletion-flag: an unprivileged process can unlink and rename files in a directory with sticky bit set only if it has write permission on the directory and owns the file or the directory (normally write permission on the directory would be sufficient). This makes it possible to create a directory that is shared by many users, who can each create and delete their own files in the directory but can’t delete files owned by other users (used for /tmp). The sticky bit states that files and directories within that directory may only be deleted or renamed by their owner (or root).

At file creation, the permissions specified in the mode parameter are modified by the umask, a process attribute that specifies which permission bits should always be turned off. This value is inherited and usually for the shell is initialized to 0022 to disable write permissions to group and other.

Timestamps are in the fields st_atime (last access), st_mtime (last modification) and st_ctime (last change of inode information). Various syscalls implicitly change one or more of those values, and they can also be changed by specific syscalls.

About file ownership, at creation time the user-ID is the effective user-ID of the process, while the group-ID has a different behavior depending also on the type of filesystem. They can be changed with the family of chown syscalls, but only a privileged process can change the user-ID of a file (and set the group-ID to any value), while an unprivileged process can change the group-ID of a file that owns (i.e. that matches its effective user-ID) to any of the group of which it is member. If owner or group are changed, the set-user-ID and set-group-ID bits are turned off.

Directories, hard links and soft links
======================================

A directory is a special file type, logically organized as a table of mappings from filenames to inode numbers.

Because the name of a file is not in its inode, multiple names (in the same or in different directories) referring to the same inode can be created. These are hard links. Each new link increments the hard link counter in the inode by one. The inode and data blocks are deallocated only when the hard link count reaches zero. Hard links suffer some fundamental limitations:

- hard links must reside on the same file system of the file they refer to (think of hard links as different names pointing to the same inode number: each inode number is, by definition, unique within a file system, but the same inode number will refer to a different file on another file system)
- hard link cannot point to a directory - circular links could be created otherwise (for example, a link to a parent directory)

The problem with circular links is that when traversing a directory hierarchy (think a "find") there is no way to detect you are looping without keeping track of inode numbers as you traverse. In general, every path must be canonicalized (i.e. resolving links), and having hard links to directories (parent) would generate more paths for the same file. Also consider that a filesystem is a DAG, hard links would let a sub-directory name be a link to a parent directory, creating a file system “cycle” or loop.

A symbolic link is a special file type whose data is the name of another file. The pathname to which the link refer may be absolute or relative. They are not included in the (hard link) count of the file to which they refer, so if the target file is removed, the link itself continues to exist and cannot be referenced (dangling - they can also be created dangling). They don’t suffer the limitation of hard links (cross filesystem boundaries and link to directories - which can be detected and skipped when traversing: softlinks are resolved first to a canonical path, that is a minimal path without the links). The owner, group, and permissions of the symlink inode itself are ignored and don’t matter. The only thing that matters in a symlink is the target name.
