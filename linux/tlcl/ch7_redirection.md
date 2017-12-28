## Standard Input, Output, and Error

Programs like `ls` send their results to a special file called `standard output` (`stdout`) and their status messages to another file called `standard error` (`stderr`). By default, both standard output and standard error are linked to the screen and not saved into a disk file.

Many programs also take input from `standard input` (`stdin`) which is, by default, attached to the keyboard.

I/O redirection lets us change where output goes and where input comes from so that input does not have to come from the keyboard, and output does not have to go to the screen.

## Redirecting Standard Output

To redirect `stdout` to another file, we use the `>` redirection operator followed by the name of the file. For example,
```
ls -l /usr/bin > ls-output.txt
```

sends the `stdout` of the `ls` command to the file `ls-output.txt`.

Note that typing in
```
> ls-output.txt
```

will create a new, empty file (or truncate an existing one). You can use the `>>` re-direction operator to append stuff to a file:

```
ls -l /usr/bin >> ls-output.txt
```

Note if the file does not exist, it will be created.

## Redirecting Standard Error

To redirect standard error, you must refer to its file descriptor. For reference, we have:
* 0 => `stdin`
* 1 => `stdout`
* 2 => `stderror`

Thus to redirect standard error, we can do `2> <filename>`. For example,
```
ls -l /bin/usr 2> ls-error.txt
```

## Redirecting Standard Output and Standard Error To One File

To capture all of the output of a command to a single file, we must redirect both standard output and standard error at the same time. There are two ways to do this. First, the traditional way:
```
ls -l /bin/usr > ls-output.txt 2>&1
```

With this method, we do two redirections. First, we redirect `stdout` to the file `ls-output.txt`. Next, we redirect `stderr` to `stdout`. The order is important. If we inverted the redirection, then `stderr` would still get directed to the screen.

With recent versions of bash, we can do
```
ls -l /bin/usr &> ls-output.txt
```

where `&>` indicates that both `stdout` and `stderr` are redirected to the file `ls-output.txt`.

## Disposing Of Unwanted Output

When we don't want the output of a command, i.e. we just want to throw things away, we can redirect output to a special file called `/dev/null`. This is a bit bucket that accepts input and doesn't do anything with it. For example, if we want to suppress error messages from a command we can do:
```
ls -l /bin/usr 2> /dev/null
```

## Redirecting Standard Input

You can use the `< |filename|` redirection operator to change the source of `stdin` to be from a file.  For example,
```
cat < lazy_dog.txt
```

## Pipelines

The pipe operator `|` lets you send the standard output of one command to the standard input of another, via the syntax
```
command1 | command2
```

## Filters

Filters are commands that take input, change it, and then output it again.
