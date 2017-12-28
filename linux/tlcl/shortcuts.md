## TERMINAL:
Up key => Previous command
Down key => Next command

Copy text into terminal (mouse) => Highlight the text by holding down the left mouse button, drag the mouse over it (or double click on a word)

ctrl-d => EOF

## FILTERS:

All of these commands will read from `stdin` by default if no filename is provided.

```
sort [filename]
```
  Prints the lines of the given file in sorted order.

```
uniq [filename]
```
  Removes duplicate lines from the file. You can see the list of duplicates with the `-d` option.

```
wc [filename]
```
  Counts the # of lines, words, bytes in files. You can use specific options to, for example, specify that only one stat needs to be displayed (e.g. the `-l` option will print out the number of lines).

```
grep pattern [filename]
```
  Finds lines that match a specified pattern in files. Some handy options are `-i` for ignoring case, `-v` to print lines that do not match the specified pattern.

```
head [filename]
```
  Prints the first 10 lines of the file (by default), can adjust with the `-n` option.

```
tail [filename]
```
  Prints the last 10 lines of the file (by default), can adjust with the `-n` option. You can also use the `-f` option to have tail watch the file in realtime, printing out any new lines that have been added to it.

```
tee (<file1>)+
```
  Reads standard input and copies it to both the passed-in filenames, and also to `stdout`. Useful as an intermediate step in a pipeline where you want to `record` the results of an earlier bit of computation and then pass it along.

## COMMANDS:
```
date
```
  Displays the current time and date

```
cal
```
  Displays a calendar of the current month

```
df
```
  Shows current amount of free space on the disk

```
free
```
  Shows the amount of free memory

```
exit
```
  Ends the current terminal session

```
pwd
```
  Displays the current working directory

```
ls
```
  Displays the list of files and subdirectories in the current working directory

```
ls <arg1> <arg2> ... <argn>
```
  Displays the list of files and subdirectories in the specified arguments

```
cd
```
  Change the working directory to the home directory

```
cd -
```
  Changes the working directory to the previous working directory

```
cd ~<user-name>
```
  Changes the working directory to the home directory of `<user-name>`

```
file <filename>
```
  Prints a brief description of the file's contents.

```
less <filename>
```
  View the file as scrollable text

```
zless <gzip path>
```
   Same as less, but for a gzipped text file.

```
mkdir <dir1> <dir2> ...
```
   Creates directories with the paths specified by `<dir1>`, `<dir2>` .... 

```
cp <item1> <item2>
```
   Copies single file or directory `<item1>` to file or directory `<item2>`

```
cp <item>... directory
```
   Copies multiple items (either files or directories) into a directory

```
mv <item1> <item2>
```
   Moves single file or directory `<item1>` to file or directory `<item2>`

```
mv <item>... directory
```
   Moves multiple items (either files or directories) into a directory

```
rm item...
```
   Removes multiple items (either files or directories)

```
ln file link
```

Creates a link to the file `file` with link `link`.

```
type <command>
```

Indicates how a command name is interpreted, i.e., resolves it to one of the four types outlined in Ch6.

```
which <command>
```

Display which executable program will be executed. Note that this does not work for aliases or builtins.

```
help <built-in>
```

Shows documentation for a given shell built-in

```
man <command>
```

Display a command's manual page

```
apropos <search-term>
```

Display a list of appropriate commands that match the given search term, searched within the available man pages.

```
info <command>
```

Display a command's info entry

```
whatis <command>
```

Display a very brief description of a command

```
alias <name>=<script>
```

Create an alias `<name>` whereby the shell will execute the passed-in `<script>` whenever `<name>` is invoked. 

```
unalias <name>
```

Removes an alias

```
cat [file...]
```

Reads one or more files and copies them to standard output
