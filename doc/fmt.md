# Fmt

[fpath](https://github.com/thingsiplay/fpath)

An incomplete list of available fmt commands usable within `-F` option. See
`--explain fmt` for a more exhaustive list. Note that `-F` also supports
[styles](styles.md) if enclosed in curly braces like `{red}` .

```bash
fpath --explain fmt

fpath --explain fmt_compact

fpath -F {FMT}
```

## Arguments

A double colon `:` in the command name specifies the following part to be
arguments to the command. `{.isdir:TEXT}` will set the command name to `.isdir`
and it's argument to `TEXT` . It depends on the command how to interpret the
arguments.

Example:

```bash
fpath -F '{.isdir:TEXT}'

fpath -F '{.atime:OPTIONS}'
```

## File path by id

File names and path related variables.

The "r" variants will resolve any symbolic link to their target.

```
{path}              - entire known path
{uri}               - convert path to file URI
{dir}               - directory path without the file name part
{dirname}           - name of the last directory part without a path
{name}              - name of the last element, either a directory or file
{stem}              - same as {name}, but without file extension
{ext}               - file extension after last dot
{exts}              - multiple file extensions, if there are multiple dots

{rpath}
{ruri}
{rdir}
...
```

Example:

```bash
fpath -F {path}

fpath -F '{rname} in {rdirname}'
```

## File path parts by index

Output only a specific part of the path by specifying start and end index. Each
part delimited by "/" of the path is one individual element.

```
{-START:END}        - slice from any to any index, including negative numbers,
                      empty value means the rest of the path
```

Example:

```bash
fpath -F {3:5}

fpath -F {-2:}
```

## File type information

The Linux command `file` is executed in a sub shell to retrieve information
about the files on the file system. This command collects the paths and runs a
single process once with all files together in one go. Therefore this is as
fast as it can get; but still considerably slower than other commands.

The "r" variants will resolve any symbolic link to their target.

NOTE: Known limitation with the `{center}`, `{right}`, `{left}` and `{fill}`
commands, which do not affect these commands. That's because this `file`
process is run at last after the formatting.

```
{file}              - long type information with description
{type}              - short type name
{mime}              - name of the associated mime type

{rfile}
{rtype}
{rmime}
```

Example:

```bash
fpath -F {type}
```

## File ownership

Simply the user or group name who owns the file.

The "r" variants will resolve any symbolic link to their target.

```
{owner}             - name of user owning the file
{group}             - name of group owning the file

{rowner}
{rgroup}
```

Example:

```bash
fpath -F {owner}
```

## File properties

These commands starting with a dot `{.` are a group of commands accessing some
information about file properties. This is quite fast operation as the file is
accessed once to get a bundle of information. So there are no performance
issues using many of them.

Normally these commands follow a symbolic links target to report information.
To suppress that and report information about the symbolic link itself, use the
`.l` variant of the same command. `{.isdir}` would become `{.lisdir}` and so
fort.

NOTE: The `OPTIONS` in `{.atime:OPTIONS}` are codes in a specific format:
[strftime](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes)

```
{.mode}             - file permisssions in form of drwxr-xr-x

{.isdir}            - place a path separator character, if its an existing directory
{.isdir:TEXT}       - place any TEXT, if its an existing directory
{.isreg:TEXT}       - place any TEXT, if its an existing regular file
{.islink:TEXT}      - place any TEXT, if its an existing symbolic link

{.size}             - file size with auto detect size and unit type, 1 rounding digit
{.b}                - file size in bytes
{.kb}               - file size in kilobytes, 2 rounding digits
{.mb}               - file size in megabytes, 2 rounding digits
{.gb}               - file size in gigabytes, 2 rounding digits

{.atime}            - last access timestamp in human readable format
{.mtime}            - last modified timestamp in human readable format
{.ctime}            - last change timestamp in human readable format
{.atime:OPTIONS}    - last access with custom strftime format codes
{.mtime:OPTIONS}    - last modified with custom strftime format codes
{.ctime:OPTIONS}    - last change with custom strftime format codes

{.lmode}
{.lisdir}
{.lisdir:TEXT}
...
```

Example:

```bash
fpath -F {path}{.isdir}

fpath -F '{.isdir:directory}{.isreg:regular file}\t{name}'

fpath -F {.atime:%Y-%M-%d}
```

## Text layout

Commands that add space or other characters to fill in, to organize a specific
layout. These have a start and associated end tag. Everything in between them
will be transformed. Such as `{center:NUM} TEXT {/center}` would transform
`TEXT` accordingly. `NUM` specifies the final size that will be calculated in
conjunction with the text size.

```
{center:NUM} {/center}  - center text by padding left and right with spaces
{left:NUM} {/left}      - left justify text by padding left with spaces
{right:NUM} {/right}    - right justify text by padding right with spaces
{fill:NUM} {/fill}      - fill left side with zeros, preserve leading +- sign
```

Example:

```bash
fpath -F {center:80}{path}{/center}
```

## Filter with any program

Filter command running an arbitrary program has two operational forms: a) with
stdin data, and b) without stdin data. `{!` marks the start of the command,
followed by the name of the program (without spaces), such as `{!PROG` .
Optionally arguments to the program can be specified with a double colon and
the arguments `:ARGS` .

Then depending if `DATA` is available between opening `{!PROG}` and closing tag
`{!/}` , will be taken and given as stdin stream for the program, or if missing
then the program is run without stdin data. Either way the output from the
program will replace whats in the area between opening and closing tag.

Also your shell might interpret the `!` character in a special way, if its not
enclosed in quotes. Generally multiple layers of quoting can be challenging, so
use this command only when really needed.

IMPORTANT: Unlike the `filter` command, this command will run the program in a
sub shell, for every path and line! This means it will slow down the execution
of the script considerably. Not recommended for use with a huge number of
files. Normally this command shouldn't be needed for most cases.

```
{!PROG:ARGS} {/!}
```

Example:

```bash
fpath -F '{!grep:-i "o"}{name}{/!}'

fpath -F '{!md5sum:"{path}"}{/!}'
```
