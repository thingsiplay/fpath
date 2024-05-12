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
path... | fpath [options] -- [path...]
```

By default stdin from a piped process is parsed as list of paths split by
newline. Optionally paths can be given as arguments too. All input paths are
output to stdout. Single and double quotes surrounding a path is automatically
detected and removed. Leading tilde character is translated into the users home
directory. With options all of these default processing can be changed.

The stdin can be ignored with option `-z`. And `--` tells the script to stop
processing options and accept anything that comes after the double dash as path
input files rather than options.

Let's see how this works with a series of simple examples:

### Basics

Output from `ls` will include quotes for paths with spaces:

```bash
$ cd ~
$ ls -1 Videos
Gaming
'My Friends'
'My Game Recording'
'My Tutorials'
YouTube
```

Pipe that into `fpath`, which removes those quotes automatically for
consistency:

```bash
$ cd ~
$ ls -1 Videos | fpath
Gaming
My Friends
My Game Recording
My Tutorials
YouTube
```

Enforce single quotes on all paths with `-q`:

```bash
$ cd ~
$ ls -1 Videos | fpath -q
'Gaming'
'My Friends'
'My Game Recording'
'My Tutorials'
'YouTube'
```

Because we are not in the same directory as "~/Videos", using the name of a
file or directory would not work. To build a correct path we can tell `fpath`
to add a base path in front of each relative path:

```bash
$ cd ~
$ ls -1 Videos | fpath -b Videos
Videos/Gaming
Videos/My Friends
Videos/My Game Recording
Videos/My Tutorials
Videos/YouTube
```

Add `-a` to turn every relative path into an absolute path, by adding the
current working directory. This operation is safe, as we build a correct
partial path in the previous example. Now we can expand it fully:

```bash
$ cd ~
$ ls -1 Videos | fpath -a -b Videos
/home/tuncay/Videos/Gaming
/home/tuncay/Videos/My Friends
/home/tuncay/Videos/My Game Recording
/home/tuncay/Videos/My Tutorials
/home/tuncay/Videos/YouTube
```

`find` is a better tool for scripting and combining output with other tools.

```bash
$ cd ~
$ find Videos/* -maxdepth 0
Videos/Gaming
Videos/My Friends
Videos/My Game Recording
Videos/My Tutorials
Videos/YouTube
```

Let's play with the path separator slash "/" and replace it by option `-/` to
any other text:

```bash
$ cd ~
$ find Videos/* -maxdepth 0 | fpath -/' -> '
Videos -> Gaming
Videos -> My Friends
Videos -> My Game Recording
Videos -> My Tutorials
Videos -> YouTube
```

Color of the path separator (meaning "/" by default) can changed with the style
option `-s` . For simplicity only one style can be specified here, but curved
brackets `{` and `}` are not required.

```bash
$ cd ~
$ find Videos/* -maxdepth 0 | fpath -s red
# see img/red.png
```

![screenshot: red style](img/red.png)

Have in mind that styles from `-s` will partially overwrite and collide with
styles specified in `-F` option, if not being carefully.

### Advanced

Using the `-F` option you get to control what and how it is being output.
Control sequences and variables allow fine grained formatting. These commands
need to be enclosed by curved brackets `{` and `}`, such as `{red}` or
`{dirname}` to be recognized. Mix and match them together with arbitrary text
to form the exact format you like.

To see a list of supported style and format commands, use the explain option
`-H` or `--explain`:

```bash
fpath --explain fmt
fpath --explain style
```

For the following examples, I have create a new directory with random files:

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0
./Arcade - A-Z Uncommon Arcade Games.lpl
./emojicherrypick
./new.png
./playlists.zip
```

Let's turn the tables and get to the meat. Show base name of file, with it's
directory part below, indicated by an arrow:

```bash
$ cd ~/Desktop/My" "Files/
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

Let's add some more information to the output, such as the file permissions and
file size (but with double colon and without arrow this time):

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F'{name}:\n\t{dir}\n\t{.mode}\n\t{.size}\n'
Arcade - A-Z Uncommon Arcade Games.lpl:
        /home/tuncay/Desktop/My Files
        -rw-r--r--
        6.7 KB

emojicherrypick:
        /home/tuncay/Desktop/My Files
        drwxr-xr-x
        4.0 KB

new.png:
        /home/tuncay/Desktop/My Files
        -rw-r--r--
        71.1 KB

playlists.zip:
        /home/tuncay/Desktop/My Files
        -rw-r--r--
        14.5 KB

```

Wanna have a format that is more familiar? Here we have added the mime type
with {mime} as well, which works by looking into file content with standard
Linux program `file`. It's called only once for all paths together, therefore
this will not slowdown the script with many paths, even thousands.

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{.size}\t{mime}     \t{name}'
-rw-r--r--      6.7 KB  us-ascii        Arcade - A-Z Uncommon Arcade Games.lpl
drwxr-xr-x      4.0 KB  binary          emojicherrypick
-rw-r--r--      71.1 KB binary          new.png
-rw-r--r--      14.5 KB binary          playlists.zip
```

In `-F` option we can also use the various color and effects commands, such as
`{yellow}` in example. Unlike in option `-s`, we have to put them in curly
braces. But it is important to understand that we have to reset the color or
entire style for the line, otherwise it will spill over to next line. Look at
the screenshot for the output, to understand the differences:

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{.size}\t{name}'
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{bold}{yellow}{.size}\t{name}'
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{bold}{yellow}{.size}\t{name}{default}'
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{bold}{yellow}{.size}{/color}\t{name}{default}'
# see img/bold_yellow.png
```

![screenshot: bold yellow style](img/bold_yellow.png)

Time and date can be displayed with `{.atime}` for last access, `{.mtime}` for
last modification and `{.ctime}` for last change in a human readable format.

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mtime}  \t{name}'
Sunday, October 23, 2022 02:07:30       Arcade - A-Z Uncommon Arcade Games.lpl
Wednesday, April 13, 2022 06:18:43      emojicherrypick
Wednesday, April 10, 2024 07:13:06      new.png
Saturday, May 04, 2024 04:40:03         playlists.zip
```

Time commands have a special variant allowing for custom formatting. Their
options are separated by double color, like this `{.mtime:options}`. The
options are actually format codes interpreted by
[strftime](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes):

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mtime:%Y/%m/%d}\t{name}'
2022/10/23      Arcade - A-Z Uncommon Arcade Games.lpl
2022/04/13      emojicherrypick
2024/04/10      new.png
2024/05/04      playlists.zip
```

I hope this helps in understanding what this tool can do and if it is for you.

Have a great rest of your day.
