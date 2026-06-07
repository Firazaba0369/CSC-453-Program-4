# CSC-453-Program-4

Question 1: Symlinks vs. Hard Links
Explain the difference between a symbolic link and a hard link. Why can a symbolic link cause an infinite loop during directory traversal, but a hard link to a directory cannot? (Hint: think about what the operating system allows you to create.)

Answer 1: 
    A hard link is a direct directory entry pointing to an inode, while a symbolic link is a file that stores a path to another file or directory. The operating system forbids creating hard links to directories, so a directory tree can never loop back on itself through hard links. A symlink however can point to any path including an ancestor directory, which causes a traversal to loop infinitely if not detected.


Question 2: Cycle Detection
Your cycle detection uses (st_dev, st_ino) pairs to identify directories. Why are both fields necessary? Give a concrete example of a scenario where checking only st_ino would incorrectly detect a cycle (or miss one).

Answer 2:
    Both fields are necessary because inode numbers are only unique within a single filesystem, not across all filesystems. For example, two different mounted filesystems could both have an inode numbered 42, so checking only st_ino would falsely detect a cycle when crossing a mount point even though the two inodes are completely unrelated. Using both st_dev and st_ino together uniquely identifies a file across the entire system.


Question 3: Filesystem Boundaries
When you run bfind / -xdev -name "*.conf", the traversal skips directories like /proc and /sys. Explain how your implementation detects that these directories are on a different filesystem. What field in struct stat changes when you cross a mount point, and why?

Answer 3:
    When a filesystem is mounted at a directory like /proc or /sys, the kernel assigns it a unique device ID stored in st_dev. Any entry inside that mounted filesystem will have a different st_dev than entries on the root filesystem, so comparing st_dev against the starting path's value is enough to detect a boundary crossing. When bfind sees st_dev change, it knows not to descend further under the xdev flag.


Question 4:
Linux presents a single unified directory tree (starting at /) even though it is composed of many different filesystems. Briefly explain the role of the VFS (Virtual File System) layer in making this possible. Why does the VFS need to exist — why can’t user programs just talk directly to each filesystem driver?

Answer 4: 
    The Virtual File System is a kernel abstraction layer that provides a single unified interface over many different underlying filesystem drivers such as ext4, tmpfs, and procfs. Without it, every user program would need to contain custom logic for each filesystem type, which is impractical and insecure. VFS intercepts calls like open, read, and stat and routes them to the correct driver, making the differences invisible to user space.
