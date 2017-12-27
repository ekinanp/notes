Linux organizes files in a hierarchical directory structure. This means that files are organized in a tree-like pattern of directories that may contain files or other directories.

Root is the first directory in the file system.

Linux has a single file system tree. Storage devices are attached (mounted) at various points on the tree.

The home directory contains files belonging to a specific user. Regular users can only write files in their home directory.

Absolute pathnames begin with the root directory. Relative pathnames begin from the working directory, and typically start with ".", representing the current working directory, and "..", representing the parent directory. If a pathname is not specified to something, then "./" is implicitly assumed.

Some facts about filenames:
  (1) Filenames beginning with a period character are hidden. Examples of hidden files include configuration and settings files.

  (2) Filenames and commands in Linux are case sensitive.

  (3) Linux does not have a concept of "file extension", so files can be named in any way that is preferred.

  (4) Use underscore characters to represent spaces between words.
