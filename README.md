# fpath

Reformat and stylize file path like text output.

- Author: Tuncay D.
- Source: [Github](https://github.com/thingsiplay/fpath)
- License: [MIT License](LICENSE)

![screenshot: mix blue and red style](img/blue_and_red.png)

## What is this?

Convenience script to style and reformat the output with path like formats.
Designed to combine listings from other programs output, such as from `find`
and even `ls`, or any other tool with similar capabilities. The script works
mostly on text data rather than the file system, but some special commands
are exception to this rule and will access the file system.

### But why?

Seems like a good and legit question. I'm a person who does a lot of things in
the terminal and scripts. Often I will just output all indexed files from
`balooctl6` command or search for files with very specific settings with `find`
command. Often it is hard to read what I am seeing on the screen, especially
with many files. Simply adding some coloring or format and filter output to
make it better readable is a useful thing to do, even if its only temporarily.
This script `fpath` was born, in a matter to simplify and speedup this process
when needed.

## Installation

This is a Python script. It is developed for Linux and has no dependencies,
other than Python itself. Just download and give it the executable bit.

```bash
git clone https://github.com/thingsiplay/fpath
cd fpath
chmod +x fpath
```

### Copy / Install

Following install operation is optional and is just an example how you could
copy the script to your system. Use a destination that makes sense for your
system (i.e. a directory where executable files are located).

```bash
\install -v fpath ~/.local/bin/
```

Note: The backslash at the beginning of `\install` tells Bash to ignore Bash
alias or functions and use the program directly.

## Usage

The usage is pretty simple:

```bash
stdout | fpath [options] -- [path...]
```

By default it will read from (newline character delimited) stdout of another
program and optionally from given arguments as file paths. And they are output
back after some processing. By default quotes (single and double) enclosing a
path are removed and the leading tilde is resolved into users home directory;
just as god intended. Let's see how this works in a simple series of commands.

Use `-h` option to display available options.

```bash
fpath -h
```

Let's assume the output of `ls` from the home directory. (Usually it is advised
against using `ls` output in pipes, but for demonstration purposes just let's
ignore any good advice here.)

### Basics

As you can see by default output is just their names without a path at all. And
those with spaces in the name are enclosed by quotes:

```bash
$ cd ~
$ ls -1 ~/Videos
Gaming
'My Friends'
'My Game Recording'
'My Tutorials'
YouTube
```

The default output by piping into `fpath` in this case would just remove those
quotes for consistency:

```bash
$ cd ~
$ ls -1 ~/Videos | fpath
Gaming
My Friends
My Game Recording
My Tutorials
YouTube
```

However single quotes can be enforced for all paths with option `-q`:

```bash
$ cd ~
$ ls -1 ~/Videos | fpath -q
'Gaming'
'My Friends'
'My Game Recording'
'My Tutorials'
'YouTube'
```

Option `-a` can add the current working directory to all relative paths to turn
them into an absolute path. This would result in something like
`/home/tuncay/Gaming` like paths. We can use option `-t` to convert the home
part back into a tilde, for visual clarity:

```bash
$ cd ~
$ ls -1 ~/Videos | fpath -a -t
~/Gaming
~/My Friends
~/My Game Recording
~/My Tutorials
~/YouTube
```

As said before, `ls` isn't actually the best tool for combining its output into
other programs. The better tool for this job would be to use `find` or even any
other tool dedicated for this kind of job. Here it produces entire paths with
all its directory parts and without quotes, a much better place to be in.

```bash
$ cd ~
$ find ~/Videos/* -maxdepth 0
/home/tuncay/Videos/Gaming
/home/tuncay/Videos/My Friends
/home/tuncay/Videos/My Game Recording
/home/tuncay/Videos/My Tutorials
/home/tuncay/Videos/YouTube
```

Let's play with the path separator slash "/" and replace it by option `-/` to
any other text:

```bash
$ cd ~
$ find ~/Videos/* -maxdepth 0 | fpath -/' -> '
 -> home -> tuncay -> Videos -> Gaming
 -> home -> tuncay -> Videos -> My Friends
 -> home -> tuncay -> Videos -> My Game Recording
 -> home -> tuncay -> Videos -> My Tutorials
 -> home -> tuncay -> Videos -> YouTube
```

Single style names can be used with `-s` option, to style the path separator
itself. For simplicity no curved brackets `{}` are required and we cannot
combine multiple styles here. Off course I cannot show you the colors that
easily without screenshots. This is how it would look like with red separators:

```bash
$ cd ~
$ find ~/Videos/* -maxdepth 0 | fpath -s red
# see img/red.png
```

![screenshot: red style](img/red.png)

Have in mind that styles from `-s` will partially overwrite and collide with
styles specified in `-F` option, if not being carefully. For more advanced
usage and an explanation, see below [Advanced](#advanced) topic.

One more thing: Let's start by a different folder.

```bash
$ cd ~/Desktop/My Files/
$ find ./* -maxdepth 0
./Arcade - A-Z Uncommon Arcade Games.lpl
./emojicherrypick
./new.png
./playlists.zip
```

Without options `fpath` would remove the `./`, as having no directory part in
the path is synonymous with the current directory:

```bash
$ cd ~/Desktop/My Files/
$ find ./* -maxdepth 0 | fpath
Arcade - A-Z Uncommon Arcade Games.lpl
emojicherrypick
new.png
playlists.zip
```

We can use the `-a` option again to turn the path into an absolute path, with
its parts from current working directory.

```bash
$ cd ~/Desktop/My Files/
$ find ./* -maxdepth 0 | fpath -a
/home/tuncay/Desktop/My Files/Arcade - A-Z Uncommon Arcade Games.lpl
/home/tuncay/Desktop/My Files/emojicherrypick
/home/tuncay/Desktop/My Files/new.png
/home/tuncay/Desktop/My Files/playlists.zip
```

### Advanced

Using the `-F` option you get to control what and how it is being output.
Control sequences and variables allow fine grained formatting. These commands
need to be enclosed by curved brackets `{}`, such as `{red}` or `{dirname}` to
be recognized. Mix and match them together with arbitrary text to form the
exact format you like.

To see a list of supported style and format commands, use the explain option
`-H` or `--explain`:

```bash
fpath --explain fmt
fpath --explain style
```

Let's turn the tables and get to the meat. Show the name and directory of path
on their own lines, with an additional newline at the end for clarity (note
in the example output below the tabs are replaced by 8 spaces, but in the
terminal actual tabs are produced and output):

```bash
$ cd ~/Desktop/My Files/
$ find ./* -maxdepth 0 | fpath -a -F'{name}\n\t-> {dir}\n'
Arcade - A-Z Uncommon Arcade Games.lpl
        -> /home/tuncay/Desktop/My Files

emojicherrypick
        -> /home/tuncay/Desktop/My Files

new.png
        -> /home/tuncay/Desktop/My Files

playlists.zip
        -> /home/tuncay/Desktop/My Files

```

Let's add some more information to the output, such as the file permissions
and file size (but without the arrow from previous example):

```bash
$ cd ~/Desktop/My Files/
$ find ./* -maxdepth 0 | fpath -a -F'{name}\n\t{dir}\n\t{.mode}\n\t{.size}\n'
Arcade - A-Z Uncommon Arcade Games.lpl
        /home/tuncay/Desktop/My Files
        -rw-r--r--
        6.7 KB

emojicherrypick
        /home/tuncay/Desktop/My Files
        drwxr-xr-x
        4.0 KB

new.png
        /home/tuncay/Desktop/My Files
        -rw-r--r--
        71.1 KB

playlists.zip
        /home/tuncay/Desktop/My Files
        -rw-r--r--
        14.5 KB
```

Or let's take the same commands and scramble the output structure entirely:

```bash
$ cd ~/Desktop/My Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mode} {.size} {name}  ->  {dir}'
-rw-r--r-- 6.7 KB Arcade - A-Z Uncommon Arcade Games.lpl  ->  /home/tuncay/Desktop/My Files
drwxr-xr-x 4.0 KB emojicherrypick  ->  /home/tuncay/Desktop/My Files
-rw-r--r-- 71.1 KB new.png  ->  /home/tuncay/Desktop/My Files
-rw-r--r-- 14.5 KB playlists.zip  ->  /home/tuncay/Desktop/My Files
```

We can also utilize the style commands to change color or effects. However we
need to reset the settings at the end of the line, if we don't want it to spill
over to the next lines. To illustrate the effects and difference, I'll show you
a single screenshot with the output of multiple commands in one go:

```bash
$ cd ~/Desktop/My Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mode} {.size} {name}  ->  {dir}'
$ find ./* -maxdepth 0 | fpath -a -F'{.mode} {.size} {bold}{yellow}{name}  ->  {dir}'
$ find ./* -maxdepth 0 | fpath -a -F'{.mode} {.size} {bold}{yellow}{name}  ->  {dir}{default}'
$ find ./* -maxdepth 0 | fpath -a -F'{.mode} {.size} {bold}{yellow}{name}{/color}  ->  {dir}{default}'
# see img/bold_yellow.png
```

![screenshot: bold yellow style](img/bold_yellow.png)

To display the time and date of last modification, use `{.mtime}`, or last change
`{.ctime}` or for last access `{.atime}` in a human readable format:

```bash
$ cd ~/Desktop/My Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{.size}\t{.mtime} \t{name}'
-rw-r--r--      6.7 KB  Sunday, October 23, 2022 02:07:30       Arcade - A-Z Uncommon Arcade Games.lpl
drwxr-xr-x      4.0 KB  Wednesday, April 13, 2022 06:18:43      emojicherrypick
-rw-r--r--      71.1 KB Wednesday, April 10, 2024 07:13:06      new.png
-rw-r--r--      14.5 KB Saturday, May 04, 2024 04:40:03         playlists.zip
```

The time commands have also a special variant that allows custom format on
their own, via
[strftime](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes)
format codes. These additional arguments are separated by double colon ":" like
in `{.mtime:arguments}`:

```bash
$ cd ~/Desktop/My Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{.size}\t{.mtime:%Y-%m}\t{name}'
-rw-r--r--      6.7 KB  2022-10 Arcade - A-Z Uncommon Arcade Games.lpl
drwxr-xr-x      4.0 KB  2022-04 emojicherrypick
-rw-r--r--      71.1 KB 2024-04 new.png
-rw-r--r--      14.5 KB 2024-05 playlists.zip
```

I hope this helps in understanding what this tool can do and if it is for you.

Have a great rest of your day.
