Commands look like this:
  command -options arguments

where options modify the command's behavior, and the arguments are the items that the command acts upon. Examples of options are short options (single character preceded by a dash), or long options (consisting of a word preceded by two dashes). Many commands allow multiple short options to be strung together, e.g. in `ls`, the options `-l` and `-t` can be specified together as `-lt`.

## ls -l

```
enis:tlcl enis.inan$ ls -l
total 32
-rw-r--r--  1 enis.inan  staff   437 Dec 27 12:22 ch2_what_is_the_shell
-rw-r--r--  1 enis.inan  staff  1158 Dec 27 12:41 ch3_navigation
-rw-r--r--  1 enis.inan  staff   501 Dec 27 12:48 ch4_exploring_the_system
-rw-r--r--  1 enis.inan  staff   894 Dec 27 12:44 shortcuts
```

where the columns are:

(1) File permissions. The leading dash means a regular file, while `d` indicates a directory. The next three characters are the permissions for the file's owner; the next three are for the group; while the final three are for everyone else.

(2) File's number of hard links.

(3) File owner

(4) File group

(5) Size of the file (in bytes)

(6) Date and time of the file's last modification

(7) Name of the file

## file

The `file` command determines the file's type. It can be invoked this way:
```
file <filename>
```

Here's an example of its output:
```
enis:tlcl enis.inan$ file shortcuts.md
shortcuts.md: ASCII text
```

Note in Linux, "everything is a file", including devices.

## less

`less` is a command to view text files. Commands for viewing a file under less are:

Page Up or b       -- Scroll back one page
Page Down or space -- Scroll forward one page
Up Arrow           -- Scroll up one line
Down Arrow         -- Scroll down one line
G                  -- Move to the end of the text file
1G or g            -- Move to the beginning of the text file
/characters        -- Search forward to the next occurrence of characters
n                  -- Search for the next occurrence of the previous search
h                  -- Display help screen
q                  -- Quit less

## Guided tour

This section lists the interesting directories for Linux machines, as well as some of the more interesting files in them. Here's the interesting bits for me:

- `/media` contains mount points for removable media such as USB drives, CD-ROMS, etc., that are mounted dynamically at insertion. /mnt contains mount points for removable devices that have been mounted manually

- `/opt` directory installs "optional" software, i.e. those from commercial software products.

- `/proc` is a virtual filesystem maintained by the Linux kernel containing peepholes into the kernel itself. It gives a picture of how the kernel views your computer.

- `/sbin` contains "system" binaries, i.e. programs that perform vital system tasks, usually reserved for the superuser. `/bin` contains binaries (programs) that must be present for the system to boot and run.

- `/tmp` stores temporary, transient files created by various programs.

- `/usr` contains all the program and support files used by regular users.

- `/usr/local` contains programs that are not included with the Linux distribution, but are intended for system-wide use. Programs included from source code are in `/usr/local/bin`. `/usr/bin` contains stuff included in the Linux distribution. Note it's easier to remember this as `/usr` stores included programs, `/usr/local` are for installed programs (at a high-level). Then `/usr/lib` stores shared libraries for programs in `/usr/bin`, while `/usr/local/lib` stores shared libraries for programs in `/usr/local/bin`.

- `/var` contains things that are likely to be changed (e.g. databases, spool files, user mail, etc.). As another example, `/var/log` contains log files, i.e. records of system activity. The most useful is `/var/log/messages`. Note that everything else listed so far, save for `/tmp` and `/home`, are relatively static.

## Symbolic Links

Files can be referenced by multiple names in Unix-like systems; these are referred to as symbolic links. In `ls -l`, an entry with a symbolic link might look something like:
```
lrwxrwxrwx 1 root root 11 2007-08-11 07:34 libc.so.6 -> libc-2.6.so
```

where `l` shows that the file has a symbolic link. Here, `libc.so.6` is the symbolic link referencing `libc-2.6.so`

Symbolic links are useful. One common use-case is to link versions of a specific piece of software, e.g. (`ruby` for `ruby-2.4.1`) so that when a newer version of the software comes out, the only change that needs to be done is to update the symbolic link to point to the new version (by deleting the current symbolic link, and creating a new one).`

## Hard links

Hard link is a second type of link that also lets files have multiple names, but in a different way.
