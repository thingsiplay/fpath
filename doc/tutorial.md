# Tutorial

[fpath](https://github.com/thingsiplay/fpath)

This is a short step by step series of simple commands to get familiar with
basic and advanced operations. It is recommended to read the
[README](../README.md) first, with information about default processing of
input data.

## Let's get started

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

## Advanced

Using the `-F` option you get to control what and how it is being output.
Control sequences and variables allow fine grained formatting. These commands
need to be enclosed by curved brackets `{` and `}`, such as `{red}` or
`{dirname}` to be recognized. Mix and match them together with arbitrary text
to form the exact format you like.

For the following examples, I have create a new directory with random files:

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0
./Arcade - A-Z Uncommon Arcade Games.lpl
./emojicherrypick
./new.png
./playlists.zip
```

### Multi line output

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

### Conditional output

Display any custom text conditionally, if its a directory `{.isdir:TEXT}`, a
regular file `{.isreg:TEXT}` or a symbolic link `{.islink:TEXT}`. The `TEXT`
portion is only written, if the condition is met.

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F'{name}\n\t-> {.isdir:directory}\n'
Arcade - A-Z Uncommon Arcade Games.lpl
        ->

emojicherrypick
        -> directory

new.png
        ->

playlists.zip
        ->

```

### Nest other variables in conditions

We can even nest other variables inside the `TEXT` (however this is not
extensively tested yet, so you should be careful):

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F'{name}\n\t-> {.isdir:"{path}/"}\n'
Arcade - A-Z Uncommon Arcade Games.lpl
        ->

emojicherrypick
        -> "/home/tuncay/Desktop/My Files/emojicherrypick/"

new.png
        ->

playlists.zip
        ->

```

### True directory path separator

Notice that with the above code `{path}/`, the last added slash is just an
arbitrary character and not a true path separator. Therefore it would not get
the special treatment with the coloring option `-s red` or changing its form
with option `-/' -> '` in example. There is a special `{.isdir}` variant
without arguments like in `{.isdir:TEXT}` . There is no text to select, as it
outputs a separator instead that inherits all the abilities from a path. Try
it out with and without `-s red` option:

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -s red -F'{name}\n\t-> {.isdir:"{path}{.isdir}"}\n'
Arcade - A-Z Uncommon Arcade Games.lpl
        ->

emojicherrypick
        -> "/home/tuncay/Desktop/My Files/emojicherrypick/"

new.png
        ->

playlists.zip
        ->

```

### File size and permissions

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

### ls like output with MIME of file

Wanna have a format that is more familiar? Here we have added the mime type
with `{mime}` as well, which works by looking into file content with standard
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

### Colors with style

In `-F` option we can also use the various color and effects commands, such as
`{yellow}` in example. Unlike in option `-s`, we have to put them in curly
braces. But it is important to understand that we have to reset the color or
entire style for the line, otherwise it will spill over to next line. Look at
the screenshot for the output, to understand the differences:

Note: An empty `{}` is a shortcut for `{default}`.

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{.size}\t{name}'
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{bold}{yellow}{.size}\t{name}'
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{bold}{yellow}{.size}\t{name}{default}'
$ find ./* -maxdepth 0 | fpath -a -F'{.mode}\t{bold}{yellow}{.size}{/color}\t{name}{}'
# see img/bold_yellow.png
```

![screenshot: bold yellow style](img/bold_yellow.png)

### Wanna a date?

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

### Custom date format

Time commands have a special variant allowing for custom formatting. Their
`OPTIONS` are separated by double color, like this `{.mtime:OPTIONS}`. The
`OPTIONS` are actually format codes interpreted by
[strftime](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes):

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F'{.mtime:%Y/%m/%d}\t{name}'
2022/10/23      Arcade - A-Z Uncommon Arcade Games.lpl
2022/04/13      emojicherrypick
2024/04/10      new.png
2024/05/04      playlists.zip
```

### Slice any parts

There are so many more commands and functionality. Such as slices in form of
`{START:END}` even supporting negative numbers, to get any part of the path,
such as `{-1:}` to access file name or `{3:5}` to get fourth and fifth element.
No, this is not at typo. Counting starts by 0 and the end index is where to
stop, without including it. Just test and play around with it to understand.

```bash
$ cd ~/Desktop/My" "Files/
$ find ./* -maxdepth 0 | fpath -a -F '{path}\n\t{2:4}\n\t{-3:}\n'
/home/tuncay/Desktop/My Files/Arcade - A-Z Uncommon Arcade Games.lpl
tuncay/Desktop
Desktop/My Files/Arcade - A-Z Uncommon Arcade Games.lpl

/home/tuncay/Desktop/My Files/emojicherrypick
tuncay/Desktop
Desktop/My Files/emojicherrypick

/home/tuncay/Desktop/My Files/new.png
tuncay/Desktop
Desktop/My Files/new.png

/home/tuncay/Desktop/My Files/playlists.zip
tuncay/Desktop
Desktop/My Files/playlists.zip
```

### Filter through running any program

With a recent update a new command is available to run arbitrary programs. Any
`DATA` enclosed between `{!PROG:ARGS}DATA{/!}` will be taken as stdin stream to
the program. And it's output will replace `DATA`:

```bash
$ cd ~/Desktop/My" "Files/
$ time find ./* -maxdepth 0 | fpath -F '{!grep:-i -E "^\w+$"}{name}{/!}'

emojicherrypick


```

If nothing is specified in `DATA`, such as `{!PROG:ARGS}{/!}` , then program is
executed without stdin stream, but its output will still be printed out in that
place. Notice in this example we need surrounding quotes for the arguments of
the program:

```bash
$ cd ~/Desktop/My" "Files/
$ time find ./* -maxdepth 0 | fpath -F '{!md5sum:"{path}"}{/!}'
6079b16fceede79ed1c9b797ba07ad37  Arcade - A-Z Uncommon Arcade Games.lpl

8bf704f44b5231caee4a26d4f4a61645  new.png
6c814dd890934b71f9538240db103795  playlists.zip
```

However this approach running programs with `{!PROG` command is very slow. In
fact, it will create a new shell process for every single line. So be careful.

I hope this helps in understanding what this tool can do and if it is for you.
