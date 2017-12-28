## Wildcards

Wildcards let you select filenames based on patterns of characters. See the table in the book for a list of the available wildcards.

## cp - Copy Files and Directories

Most of this command is listed in the shortcuts section. These are just the useful options that are commonly used with it:

```
-a, --archive
```
  Copy the files and directories, and all of their attributes including ownership and permissions. Normally, copy takes on the default attributes of the user performing the copy

```
-i, --interactive
```
  Before overwriting an existing file, prompts the user for confirmation. By default, `cp` silently overwrites files.

```
-r, --recursive
```
  Recursively copies directories and their contents. This (or `-a`) is required when copying directories.

```
-u, --update
```
  When copying files from one directory to another, only copy files that either don't exist, or are newer than the existing corresponding files, in the destination directory.

```
-v, --verbose
```

## mv -- Move and Rename Files

The options are `-i`, `-u`, `-v` with the same specs. as in `cp`.

## rm -- Remove Files and Directories

`-i`, `-r`, `-v` with the same semantics as `cp`. There is an additional flag, `-f`/`--force` that ignores nonexistent files and does not prompt the user when doing the removal. It overrides `-i`. Note that to remove a directory, `-r` must be specified.

Note that files deleted with `rm` cannot be recovered -- there is no undelete command. Good tip is that when you're using wildcards with `rm`, test it out first with `ls` so that you can see the exact files that will be deleted.

## ln -- Create links

The `ln` command creates either hard or symbolic links. It is used as:

```
ln file link
```
to create a hard link and

```
ln -s item link
```
to create a symbolic link where "item" is either a file or a directory.

## Hard Links

Every file has a single hard link that gives it its name. When a hard link is created, an additional directory entry is created for that file. Hard links have two important limitations:
* It cannot reference a file outside its own file system, i.e. a file that is not on the same disk partition as the link itself.
* It may not reference a directory.

A hard link is indistinguishable from the file itself. When a hard link is deleted, the link is removed but the contents of the file continue to exist until all links to that file are deleted.

To understand hard links, think of files as consisting of two parts: the `name` part and the `data` part. Hard links create additional name parts that refer to the same data part. The system assigns a chain of disk blocks to an inode that is then associated with a name part. Each hard link therefore refers to a specific inode containing the file's contents.

So two hard-links refer to the same file if they share the same inode number, which can be obtained from `ls` via. the `-i` option.

Note that renaming files do not affect hard links, as hard links point to the specific inode and not the file name itself.

## Symbolic Links

These contain a "pointer" to the referenced file or directory. When you write to a symbolic link, you write to the file. But when you delete a symbolic link, the file will still exist. When the file is deleted, the symbolic link will continue to exist but it will point to nothing (dangling link). `ls` will display broken links in a distinguishing color, such as red, so you can visually detect them.

When creating sym-links, it is more desirable to use relative path names than absolute ones. This way, if a directory containing a sym link is ever renamed, the sym link will still reference the correct location.

Note that if the pointed-to file is renamed, the sym link will NOT be updated to point to the newly named file.
