# Linux system administration

## Basics

### File system

#### Inodes
Contain metadata about files. Every inode contains access information about
files, their size, permissions etc. Identified by an **inode number**

#### Navigating

##### listing directory contents
* `ls` lists files in a directory
* `ls -a` show hidden files
* `ls -l` long form
* `ls -lh` long form and human readable size
* `ls -i` shows the inode number

##### where am I?
* `pwd` print working directory
* `pwd -L` will show symlink in file path
* `pwd -P` show actual file path

##### symlinks
Symlinks are aliases, references to another directory entry. Symlinks break if
the file they point to is deleted. Symlinks can span volumes
* `ln -s [where to link] [link name]`

##### hard links
Hard links reference the same inode as another directory entry. Hard links
cannot span volumes. If multiple hard links exist to some file, if you delete
one the other hard link will become the file.

##### changing directories
* `cd [dirname]` go to directory  
* `cd ~` go to home directory
* `cd ~[user]` go to home directory **for specified user**
* `cd ..` go up one directory
* `cd .` go to current working directory

#### Files and file manipulation
To linux, everything is a file.
A file consists of:
1. directory table entry
2. Inode table entry
3. data blocks
  * sequence of storage addresses
  * one memory block is the minimum size for a file
  * when you format a drive, you are able to set memory block size

##### File name conventions
* no spaces. if you really need a space or some other disallowed character, you
can put a backslash `\` character in front of it.

##### Creating a file
* `touch [filename]`

##### Creating and editing with a text editor

**nano** works on buffers of text, which is the text being edited. You need to
save it to persist to disk.

Useful nano commands:

* `^R` inserts the contents of a file into the current buffer at cursor position
* `^W` search
* `^\` replace text. Option to go step by step or replace all occurrences
* `^K` cuts text. cut text is available in the cut buffer
* `^U` uncut. puts the text in the cut buffer where your cursor is
* `^G` list available commands

##### Moving and renaming files

###### `cp`
Copying files
* `cp [file] [destination]` copy a file to somewhere
* `cp -r [dir] [destination]` copy a directory to somewhere
* `cp -l` copy symbolic links

###### `mv`
Moving or renaming
* `mv [file/directory] [destination]`
* `mv -i [file/directory] [destination]` will prompt if you want to override
file at destination (the default behavior)

##### Viewing files

###### `cat`
`cat -vet [filename]` will display the contents of the file, including
specials characters

###### `more` and `less`
Display files page by page

##### Deleting files

###### `rm` and also `rmdir`
remove files and directories
* `rm [filename]`
* `rm -r [filename]` recursively remove
* `rm -rf [filename]` recursively remove and don't stop for errors (use with
  extreme caution)

#### Pipes and redirects

`stdin` (0), `stdout` (1) and `stderr` (2) are the standard files / streams. Any
running terminal program has these three files available.

##### Pipes
A pipe is file that connects two streams (also files!). pipes allow you to
channel the output from one command to another command. Use the `|` keyword for
this.

Example usage:

`ls -lh | less`

##### Redirects
Similar to pipe, but instead of connecting commands, it will connect the
standard streams to files

* `>` write to file
  * use `cat > file` to add text to a file without needing to use a text editor
* `>>` append to file
* `<` use file as input (for a command)
  * `cat < foo.txt`
* `<<` ???

* `1>` redirect from stdout
* `2>` redirect from stderr

#### Regular expressions

To enable regex in nano, you have to edit its configuration file `nanorc`. You
can look for something in a linux system with the `whereis` command.

    $ whereis nanorc

Will give you `/etc/nanorc`

Open the file with elevated permissions; look for and uncomment the lines that
match `regexp` and `casesensitive`. Save the file.

in nano, `CTRL-\` will show the regex search pane.

#### grep and sed

##### `grep` = Global Regular Expression Print
`grep` is the command line equivalent of searching in a text editor.
When you find a match with grep, the whole line that contains the match is
returned to `stdout`

One issue with `grep` is that it does not do extended regex out of the box and \
will fail silently if it doesn't understand the regex you're feeding it.

Also, you should enclose the regex pattern in quotes to make sure you're not
using characters that are reserved for bash.

To make sure `grep` understands extended regex syntax, run it with a flag.

    grep -E '[a-z]+\b' [filename]

without the flag, you would not have been able to use the `\b` shorthand.

Another useful flag for `grep` is `-v`, which will return you any lines but the
ones that matched.

    grep -c [pattern] [filename] # returns the number of occurrences
    grep -l [pattern] [filename] # returns the names of matches files (instead
      of the whole line containing the match)
    grep -n [pattern] [filename] # includes line numbers
    grep -r [pattern] [filename] # recurse into directories

##### `sed` = Stream Editor
`sed` is the command line equivalent of replace (in a text editor). This is how
you replace some text with `sed`. Imagine test.txt contains the word **hello**:

    sed s/hello/'is it me you looking for?'/ test.txt

Note that you need to quote a string if it contains spaces.

`sed` uses standard regex by default, which is not particularly useful if you
have multiple matches on a single line. that's why you want to enable extended
regex with the `-r` flag.

    sed -r s/\bhello\b/'is it me you looking for?'/ test.txt

The most used commands in `sed` are:

* `s` for substitution
* `c` to replace the entire matched line with the substitution text. e.g.:
`sed '/foo/ c\bar'`
* `a` to append the substitution text to the end of the line
* `y` to transform character by character.

`sed` pattern may contain any valid regex pattern, like captured groups etc.

    cat test.txt | sed -r 's/(\bh+\b)/found: \1/'

Overview of available flags to `sed`. The explanation in the video is a bit
unclear here, because these flags actually need to be appended to the search
pattern which is passed to `sed`.

    sed -L # ignores case
    sed -r # use extended regex (already described earlier)
    sed -n # suppress printing

#### `Whereis` and `$PATH`

`Whereis` tells you where a specific command lives and where its corresponding
manpage is located.

The reason for linux files/commands being in the folders they are in, is mainly
because of traditional/historical reasons.

If you need bash to look for commands in a custom location, you can tell it to
do so by introducing a search path. The search paths are stored in bash's
`$PATH` variable.

You can `echo $PATH` and you will see the currently configured search paths for
bash separated by a colon `:`

Where the base path is set, depends on your linux distribution flavor.

#### SUDO
You can not access a file if the permissions set on in don't allow you to. This
counts even if you own the file. `sudo` allows you to circumvent file
permissions and operate on any file, event if it's owned by the system. There
are two ways to elevate permissions.

* `sudo` + disabled root account (will prompt you for your password)
* `su` + root account (mind you that the password should be **very** strong)

`sudo` is an application. It's actually another command shell. It can execute
one command at a time and will log usage in `/var/log/auth.log`.

`su` stands for **switch user**. You can become any user account you have the
password to. By default, `su` will switch to the `root` account.

#### `man` pages
Manuals. Typically broken down into sections. Section levels are covered in
`man man`

#### `df` = display filesystem
A typical call to `df -h` will give you an overview of mounted devices and the
amount of available space in human readable format. e.g.:

    /dev/sda1       453G  8,4G  422G   2% /

The last column `/` tells you where the device is mounted. The fact that this
row is about `/dev/sda1` tells you that this is an actual physical drive
(device driver). Additionally, a bunch of other, virtual, drives are also
displayed by `df`

For instance, the `udev` program mounts to `/dev`, meaning that if you access
that directory, you're actually looking at a program pretending to be a file
system.

##### Mounting and unmounting
To mount a drive you need
* a mount point: A directory where the new filesystem can be connected.
* the device name for the new drive.
use `fdisk -l` to get a list of drives currently connected.

All the types of filesystems your OS supports, are listed in
`/proc/filesystems`. grep for the filesystem you expect the newly attached drive
to have. Note: It's not obvious enough how you should know what type of
filesystem you're dealing with. What is shown in the video seems most like an
assumption. in this case it's apparently vfat because that's what windows uses
since 1995

    sudo mount -t [filesystem type] [device name] [mount point]

To unmount:

    sudo umount [device name]

Note that you cannot unmount a drive while you're somewhere in it.

In linux, `udev` is the daemon that manages auto-mounting of devices. It watches
for filesystem events, like a usb drive being connected or a terminal being
switched on. It provides `/dev` to the system as a virtual file system.

Where `fdisk -l` might not list a floppy drive if there's no disk in in, the
`/dev` directory will certainly be aware of it. You can  adjust the
auto-mounting rules that `udev` respects.

#### Partitions and the partition table
A partition is an area of a volume. It's like a separate disk with its own file
system. The partition table lives in a separate part of a drive. You can access
it with `sudo parted -l`

##### UEFI = Unified Extensible Firmware Interface
UEFI is the modern replacement for BIOS. Most modern linux distribution will be
booting from UEFI nowadays.

UEFI based systems will use the GPT instead of MBR.

##### Partitioning
A volume you want to create new partitions on, will need to be unmounted before
you start partitioning with `sudo parted`. Take not of which disk you're about
to format. `parted` is likely to try the first volume it is able to get its
hands on, which might very well be you main HD. Pass the volume explicitly!

    sudo parted /your/volume

the commands that `parted` accepts,  can be viewed by entering **help**

* `h` or `help` shows available commands
* `p` or `print` shows available volumes
* `q` or `quit` exits the program
* `unit mib` uses 'real' units
* `mkpart` start creating a new partition
  * enter a name
  * enter file system type (gpt or msdos)
  * where to start? enter `1mib`
  * where to end? enter whatever the size is of the volume you're about to
  format. Respect the offset you entered under **start**. So, for a 1GB disk,
  enter **1025mib**

###### `mkfs`
When you create an ext4 partition, you need to manually create the filesystem
with `sudo mkfs.ext4 /your/volume[n]` (where `n` is the partition number as
shown by `parted -l`). This is only true if your version of parted doesn't know
ext4 already.

##### Logical Volume Management

Linux version of RAID?

* PV = Physical volumes = partitions or drives
* VG = Volume Groups = groups of partitions or drives
* LV = Logical Volumes = Volumes from pooled resources of VGs

You need to install `lvm2`

for later reference.

#### `fstab`
`fstab` makes sure that all partitions that make up your file system, are
identified and loaded at startup. Volumes should ideally have a uuid to identify
them.

    sudo blkid # gives you a list of mounted volumes and their uuid's

##### TODO: Mount options
`/etc/fstab` contains information about which drives to mount on startup.
You can set mount options for your drives in `fstab`.

Adding a volume to `fstab`, means to add a new line:

    /your/drive: UUID="your drives uuid" TYPE="type"

#### Processes and daemons

Anything that does something in linux, is a process. Processes are run by the
kernel. You can list processes with the `ps` command, which will give an
overview of active processes with their associated id's that correspond to
directories in `/proc/some-process-id`, in which a symlink exists to an
executable file (`exe`).

##### Anatomy of a process

* `PID` = process id
* `UID` = user id of process owner
* `EUID` = Effective user id (in case setuid was used)
* `GID` = group id
* `EGID` = Effective group id
* `PPID` = parent process id, in case this process is a fork. The process
reports its exit status to the parent process
* **Address space map** = allocated process memory
* **Status** = runnable, sleeping, stopped, zombie. Zombie means when a process
can't exit because no parent or other process is listening for its exit status.
* **Nice value** = lower value means this process is getting more cpu power,
which means less cpu for other processes.

##### Threads

Because it take quite some overhead to create a process, threads exist.

* Part of a larger process
* Run simultaneously
* Share a process and its memory

You can identify threads by calling `ps -eL` and checking which rows have the
same process id. The row next to the process id contains the thread ids.

#### Managing processes

Get a list of processes with `ps`

    ps aux # see all processes with information
    ps fax # get a tree view of processes

##### Signals

    1 SIGHUP # Hangup detected, terminate (resets a daemon)
    2 SIGINT # Interrupt by user pressing crtl-c
    3 SIGQUIT # Exit if possible, produces a core dump
    9 SIGKILL # Kernel terminates process. a last resort!
    15 SIGTERM # Exit and clean up if possible

You can send a signal to a process with `kill`

##### Daemons

Daemons are processes which often have

* init
* upstart
* systemD

as parent.

Daemons are not associated to a terminal, but are run in the background

Find a daemon and its config files with `whereis`.

###### `init`
Init takes care of running a daemon. It's the most oldschool daemon runner.

    sudo service [service name] start
    sudo service [service name] stop

Init scripts are generally located in `/etc/init.d`

Init has run levels:

    0 # Halt
    1 # Single user mode
    2 # Local multiuser with networking but no servers
    3 # Full multiuser with networking
    5 # Full multiuser with networking and Xwindows
    6 # Reboot

`upstart` and `systemD` are the new daemon runners. A big advantage with these
programs is that they can declare process dependencies in their startup scripts.

    systemctl start [process] # to use systemD
    systemctl enable [process] # configure a daemon to start when booting

To enable or disable daemons to run at startup in `init` and `upstart`, move
their init scripts out of the `/etc/init` or `/etc/init.d` directories.

#### Virtual memory

Virtual memory is an abstraction between disk memory and ram. Processes that are
inactive can swap out the memory they have in ram to a swap partition on the
hard drive.

You can inspect virtual memory with `vmstat`. `top` inspects memory continuously
for all running processes.

##### Swap files and partitions

###### partitions
You can create a partition on your system to contain the swap file. Refer to
the section about partitioning for a recap. When you have created the partition:

    sudo mkswap /dev/swap-partition # to create the swap file
    sudo swapon /dev/swap-partition # to enable.
    sudo swapoff /dev/swap-partition # to disable

To make sure the swap partition is used at startup, add a line to `/etc/fstab`

    /dev/swap-partition none swap sw 0 0

###### files

    # Create a file
    sudo fallocate -l 537m swapfile # for a 512mib swap file.
    # set permissions
    sudo chmod 600 swapfile # only
    # create the swap
    sudo swapon swapfile

##### Tuning virtual memory

Take a look in `/proc/sys/vm`. Here the system keeps track of memory
settings. To manipulate the settings, you need to add the setting to
`/etc/sysctl.conf`. E.g. to manipulate **swappiness** add the line

    vm.swappiness=value

and reboot the system.

#### File security
* Permission classes for files in linux are: **owner**, **group** and **other**
* Every file has a **mode** or **permission** field in its inode.
* **owner** and **groupid** are also stored in the inode.
* File ownership and group membership can be changed with `chown` and `chgrp`
* mode bits for each class are: **read**, **write** and **execute**
  * Execute for directories means that you can list its contents

* `chown` changes file ownership
  * really only useful with `sudo`
* `chgrp` changes file group
  * note that you can change a file group to any group you are a member of,
  without having to use `sudo`. If you want to change it to a group that you're
  not a member of, `sudo` is required
* `chmod` changes file mask/permissions

Use `-X` to mark directories as searchable/scannable.

##### `chmod`
You can use `chmod` in binary mode

    chmod 755 test.sh # Owner can do anything, user/group and world can read and execute

You can also use text based mode

    chmod a=rwx,go-w test.sh # does the same as above.

`a` is a shorthand for all classes, so you could also type out `ugo`.

##### `setuid` and `setguid`

Putting `setuid` on a file will run a process with the file's owner's privileges
You can verify that a file is going to execute with setuid by looking at the
permissions and checking for an `s` where you would expect to see an `x` for
execute.

To set up a file to execute like this, use `chmod` with an extra bit in front of
the 3 bits you already know.

    sudo chmod 4755 test.sh

Or, you could also use character mode.

    sudo chmod u+s,a+x test.sh

`setguid` does something similar for directories. by enabling the 'sticky bit'
with `chmod`, you can protect a file from being deleted even if its permission
would normally allow another user to do so.

TODO: clarify and research.

#### Access control lists

Give the administrator fine grained control over file access. Read or
manipulate it with `getfacl` and `setfacl`

This is only available in new versions of linux with system.

    setfacl -m "u:myuser:wrx" test.sh # sets rwx for myuser
    setfacl -x "u:myuser" test.sh # removes privileges for myuser
    setfacl -b test.sh # removes all acl privileges

#### Account anatomy

Building an account from scratch

What makes an account?

* an entry in `/etc/passwd`. With the actual password in `/etc/shadow`
* an entry in `/etc/group` / `/etc/gshadow`
* a home directory
* a password

```
sudo adduser username # adds a user account
sudo deluser username # removes a user account
```

#### Log files

Log files are created and maintained by `rsyslogd`. It does not clean up after
itself, which is why you want to set up `logrotate`. The output of log files is
usually found in `/var/log`.

To check the last lines of a log file, use `tail`

    tail -f my.log # follow log as it grows
    tail -20 my.log # read the last 20 lines. default is 10 lines

`dmesg` produces the most complete overview of kernel events.

#### How to handle boot problems

Steps to debug hardware:

1. plugged in?
2. keyboard + mouse + screen?
3. enough ram
4. fans running
5. BIOS/UEFI config
  * correct boot volume
  * Boot config matches motherboard

Files and filesystems. Mounting into another files system:

If the broken system disk is missing a partition, u can use `parted` in rescue
mode.

* `unit s` for sectors
* `rescue`
* key in the beginning and ending sectors

`fsck` will check a filesystem for errors. Note that you **need to unmount**
any filesystem you want to submit to a check, otherwise bad things will happen.

    umount /dev/filesystem
    fsck -ft type /dev/filesystem

CTRL-ALT-F2: opens terminal
CTRL-ALT-F7: closes terminal
