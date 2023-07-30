# Topic 104: Devices, Linux Filesystems, Filesystem Hierarchy Standard

## 104.1 Create partitions and filesystems | Weight: 2

**Which technoligies use for partioning a bolck device?**  
Systems with BIOS use MBR (Master Boot Record)  
Systems with UEFI use GPT (GUID Partition table)

**What is udev?**  
udev is a device manager for the Linux kernel.  
Linux systems use udev to add block devices and their paritions to the /dev in the form of /dev/sdb1 (2nd disk (b) and first parition (1)).

**What is fdisk?**  
fdisk is the main command for viewing/changeing and creating partitions on MBR systems.

**List the partitons?**  
`fdisk -l`

**What is parted? What is it advantage over fdisk/gdisk?**  
partd is the GNU tools for editing partitions.  
It main advantages is the ability to resize the partitions.  
It is trickier.

**How format a partition?**  
You can format your partitions with mkfs command (and mkswap for swap). This is a front end to commands like mkfs.ext3 for ext3, mkfs.ext4 for ext4.

**What is blkid?**  
blkid is a command that can print information about block devices, such as their UUID, label, file system type, etc.

## 104.2 Maintain the integrity of filesystems | Weight: 2

**What is inode?**  
An inode is a data structure in a Unix-style file system that describes a file-system object such as a file or a directory. Each inode stores the attributes and disk block locations of the object's data, such as:

* File size
* File permissions
* File owner and group
* File type
* File creation, modification and access time
* Number of hard links to the file
* Location of the file data on the disk

An inode is identified by an integer number, often called an i-number or inode number. The inode number indexes a table of inodes on the file system, which is usually located near the beginning of the partition. From the inode number, the kernel's file system driver can access the inode contents, including the location of the file, thereby allowing access to the file.

You can find out the inode number of a file using the command ls -i. For example:

```bash
$ ls -i /etc/passwd
262145 /etc/passwd 
``` 

**Which command find out about the free and used space on file systems?**  
df  
The diskfree command is used to find out about the free and used space on file systems.

**Which commands shows the used space of directories and files?**  
du  
The diskusage command shows the used space of directories and files.

**How see what uses my users space?**  
`du /home -h --max-depth 1`

**How to fix a corrupted file?**  
fsck

## 104.3 Control mounting and unmounting of filesystems | Weight: 3

**How mount the /dev/sda3 on /media/mydisk?**  
The directory /media/mydisk should be there and then, we just run:  
`sudo mount -t ext4 /dev/sda3 /media/mydisk`

**How display UUID column in lsblk?**  
`lsblk -o +UUID`

**What does fstab do?**  
For automatic mounting, Linux uses the /etc/fstab file. Its like a table which shows what file system should be mounted where during the boot.

## 104.5 Manage file permissions and ownership | Weight: 3

**To check your user and group use:**  
 whoami  
 groups  
 id  
id shows both user and group information.  
These are stored in /etc/passwd and /etc/group files.

**Compelete the table shows some more information regarding the first part of the ls -l command**  
`ls -ltrh script.sh`  
`-rwxr-xr-x 1 jadi adm 34 Mar  5  2023 script.sh
`

| Position | Meaning |
| --- | --- |
| 1 | ??? |
| 2,3,4 | ??? |
| 5,6,7 | ??? |
| 8,9,10 | ??? |
| 11 | ??? |

answer:

| Position | Meaning |
| --- | --- |
| 1 | What this entry is. Dash (-) is for ordinary files, 'l' is for links & 'd' is for directory |
| 2,3,4 | read, write and execute access for the owner |
| 5,6,7 | read, write and execute access for the group members |
| 8,9,10 | read, write and execute access for other users |
| 11 | Indicated if any other access methods (such as SELinux) are applies to this file - not part of the LPIC 101 exam |

**How can we change the permission?**  
by using chmod
>using octal (base 8) codes  
>using short codes

| Symbolic | Octal |
| --- | --- |
| rwx | 7 |
| rw- | 6 |
| r-x | 5 |
| r-- | 4 |
| -wx | 3 |
| -w- | 2 |
| --x | 1 |
| --- | 0 |

There is also an easier method. In this method u means user, g means group and o means others. You can append +x to give execute permission, +r to give read permission and +w to give write permission. For example u+x will grant execute permission to user. If you want to remove a permission, use a - sign. For example g-r to prevent group members from reading the file.

```bash
chmod 751 myfile
chmod u-x myfile
```

**Give read permission of all files inside /tmp/ to any user.**

```bash
chmod -R o+r /tmp
```

**Changing owner and groups?**  
chown, chgrp


**umask?**  
This command tells the system what permissions (in addition to execute) should not be given to new files. In other words, imagine that all new files will have 666 octal permission (write + read for user, group & others), so a umask of 0002 (yes! 4 digits) will remove the 2 (write) for others (666-0002=664)

**If we do not have write access to /etc/passwd or /etc/shadow, how is it possible to change our password then?**  
For this Linux sets two special bits for each file; suid (set user id) and sgid (set group id). If these bits are set on a file, that file will be executed with the access of the owner (or group) of the file and not the user who is running it.

Copy
$ ls -ltrh /usr/bin/passwd
-rwsr-xr-x 1 root root 50K Jul 18  2014 /usr/bin/passwd
Kindly note the s in the executable bit for the user permissions and also for the group? That means when any user runs this program, it will be run with the owner of the file access level (which is root) instead of that user's id.

It is possible to set/unset the suid and sgid using chmod and +s or -s instead of x.

While we are on this, let me introduce you to the last special bit. It is called sticky bit and if set, makes the file resistant to deletion. If the sticky bit is set, ONLY the owner of the file will be able to delete it, even if others do have write access to it. This is useful for places like /tmp (everybody has write access on /tmp directory but we do not want others to delete our files).

Sticky bit is identified by t and will be shown on the last bit of a ls -l:

Copy
$ ls -dl /tmp
drwxrwxrwt 13 root root 77824 Feb  8 21:27 /tmp
As you can see the sticky bit is set and although all users have write access in this directory, they wont be able to delete each others files.

Lets review how you can set these access modes:

| access mode | octal | symbolic |
| --- | --- | --- |
| suid | 4000 | u+s |
| sgid | 2000 | g+s |
| sticky | 1000 | +t |

guid on a directory will force any new file in that directory to have the guid of the directory itself.

## 104.6 Create and change hard and symbolic links | weight: 2

**To remove links, you can use the?**  
 rm or unlink command.

**Hard link and soft link**  
Hard links and soft links are terms used in Linux operating systems to point to files. They have some similarities and differences that you should know.

Similarities:

* Both hard links and soft links are created with the command ln.
* Both hard links and soft links can be used to access the content of the original file.

Differences:

* Hard links refer to the data itself and share the same inode number as the original file. Soft links or symbolic links point to the path to the data and have a different inode number
* Hard links act like a mirror copy of the original file and can't be used to link directories or cross file systems. Soft links are an actual link to the original file and if the original file is deleted, the soft link has no value
* Hard links are created with the command ln (original file path) (new file path). Soft links are created with the command ln -s (original file path) (new file path)
* Hard links are faster than soft links as they access the data directly. Soft links are slower as they have to follow the path to the data

## 104.7 Find system files and place files in the correct location | weight: 2

**FHS?**

| directory | usage |
| --- | --- |
| / | ??? |
| /bin |??? |
| /boot | ??? |
| /dev | ??? |
| /etc |??? |
| /lib | ??? |
| /media | ??? |
| /mnt | ??? |
| /opt | ??? |
| /sbin | ??? |
| /srv | ??? |
| /tmp | ??? |
| /usr | ??? |
| /var | ??? |
| /home | ??? |
| /lib | ??? |
| /root | ??? |

Filesystem Hierarchy Standard (FHS) is a document describing the Linux/Unix file hierarchy. It is very useful to know these because it lets you easily find what you are looking for as a system admin.

| directory | usage |
| --- | --- |
| / | Primary hierarchy root and root directory of the entire file system hierarchy |
| /bin | Essential command binaries |
| /boot | Static files of the boot loader |
| /dev | Device files |
| /etc | Host-specific system configuration |
| /lib | Essential shared libraries and kernel modules |
| /media | Mount point for removable media |
| /mnt | Mount point for mounting a filesystem temporarily |
| /opt | Add-on application software packages |
| /sbin | Essential system binaries |
| /srv | Data for services provided by this system |
| /tmp | Temporary files |
| /usr | Secondary hierarchy |
| /var | Variable data |
| /home | User home directories (optional) |
| /lib | Alternate format essential shared libraries (optional) |
| /root | Home directory for the root user (optional) |

**updatedb?**  
The updatedb command will update the database used by the locate command.

**type?**  
The type built-in command returns the location that the shell will use in order to run the given command.  
Three categories it returns are

* alias
* shell built-in
* external command

```bash
$ type ls
ls is aliased to 'ls --color=auto'
$
$ type cd
cd is a shell builtin
$
$ type find
find is /usr/bin/find
```

**fstab?**  
For permanent storage devices, Linux maintains the /etc/fstab file to indicate which drive devices should be mounted to the virtual directory at boot time.  
The proper order is the device (UUID or partition), followed by the directory to mount that device, followed by its type and options, and then the dump and fsck settings.

```bash
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/disk/by-uuid/dbf2754f-1bb1-415e-aaf8-365d012b9042 /boot ext4 defaults 0 1
```

**What is the most important features of the parted?**  
One of the selling features of the parted program is that it allows you to modify existing partition sizes, so you can easily shrink or grow partitions on the drive.

**gdisk?**  
The gdisk command is a text-mode program for creating and manipulating GUID partition tables (GPTs).  
| Command | Description |
|---------|-------------|
| b | Back up GPT data to a file |
| c | Change a partition’s name |
| d | Delete a partition |
| i | Show detailed information on a partition |
| l | List known partition types |
| n | Add a new partition |
| o | Create a new empty GUID partition table (GPT) |
| p | Print the partition table |
| q | Quit without saving changes |
| r | Recovery and transformation options (experts only) |
| s | Sort partitions |
| t | Change a partition’s type code |
| v | Verify disk |
| w | Write table to disk and exit |
| x | Extra functionality (experts only) |
| ? | Print this menu |

**dumpe2fs?**  
Display block and superblock group information

* The -b option prints known bad blocks. 
* The -f option is used to force the display of information

**fsck?**  
If corruption occurs on the filesystem, you’ll need the fsck program.

* -a : Automatically repair the file system without any questions (not recommended).
* -r : Interactively repair the file system (ask for confirmation before making any changes).
* -n : Do not make any changes to the file system (dry run).
* -p : Automatically repair the file system if possible (do not prompt unless there are serious errors).
* -y : Assume yes to all questions (not recommended).
* -f : Force checking even if the file system seems clean.
* -v : Verbose mode (display more details)

**debugfs?**  
Manually view and modify the filesystem structure, such as undeleting a file or extracting a corrupted file

**mke2fs?**  
The mke2fs command is a Linux utility that creates an ext2, ext3, or ext4 filesystem, usually in a disk partition. It creates the file system structure, such as the superblock and inode tables, which are required for the proper functioning of the file system.

**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
**Question?**  
