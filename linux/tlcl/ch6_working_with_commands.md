## What Exactly Are Comamnds?

Can be one of four things:

* An executable program, such as what's in `/usr/bin`
* Command built into the shell itself, called shell builtins. `cd`, for example, is a shell builtin.
* Shell function, i.e. miniature shell scripts incorporated into the environment.
* Alias, i.e. commands that we define ourselves that are built from other commands.

## Getting a Command's Documentation

You can use bash's `help` command to display documentation for each of the available shell built-ins. For example,
```
enis:tlcl enis.inan$ help cd
cd: cd [-L|-P] [dir]
Change the current directory to DIR.  The variable $HOME is the
default DIR.  The variable CDPATH defines the search path for
the directory containing DIR.  Alternative directory names in CDPATH
are separated by a colon (:).  A null directory name is the same as
the current directory, i.e. `.'.  If DIR begins with a slash (/), then
CDPATH is not used.  If the directory is not found, and the shell
option `cdable_vars' is set, then try the word as a variable name.
If that variable has a value, then cd to the value of that variable.
The -P option says to use the physical directory structure instead of
following symbolic links; the -L option forces symbolic links to be
followed.
```

Some notation. The square brackets (e.g. `[-L|-P]`, `[dir]`) indicate optional items. A vertical bar character indicates mutually exclusive items.

Some commands also have a `--help` option that supports their specific usage information.

You can also use `man` to display a command's manual or man page.

apropos can also be used to search the list of manpages for possible matches based on a search term. Can also use whatis.

`info` is an alternative to man pages.

Some docs may also reside in the `/usr/share/doc` directory.

## Creating Your Own Commands With alias

You can put more than one command on a line by separating them with the semicolon character:
```
command1; command2; command3 ...
```

When creating an alias, you can check if that alias is already a command using `type`. Alias takes in a `<name>=<script>` argument where `<name>` is aliased to whatever command/script is present in `<script>`.

Running `alias` with no arguments will show all of the available aliases on that system. Note that command-line defined aliases will vanish once the shell session ends.
