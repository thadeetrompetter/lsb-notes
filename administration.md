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
