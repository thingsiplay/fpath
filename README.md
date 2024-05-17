# fpath

Reformat and stylize file path like text output.

- Author: Tuncay D.
- Source: [Github](https://github.com/thingsiplay/fpath)
- License: [MIT License](LICENSE)

![screenshot: mix blue and red style](img/blue_and_red.png)

## Introduction

Convenience script to style and reformat output from a list of file path like
data. Transform output from `ls`, `find` or any other file listing into a new
structure or just add colors. Operates mostly on text based data, but has
special options and commands to access file system for additional information.

## Installation

This is a Python script. It is written for Linux, has no external library
dependency, but rely on a standard Linux program called `file` for very few
operations. Just download the script and give it the executable bit and put
it into a directory where your executable files are located.

```bash
git clone https://github.com/thingsiplay/fpath
cd fpath
chmod +x fpath
```

## Usage

```bash
paths... | fpath [options] [--] [path...]

\s -1 | fpath

fpath *.txt
```

By default stdin is parsed as a list of paths split by newline character.
Additionally paths can be given as arguments too. All input paths are output
back to stdout. Without any options, following operations are done
automatically:

- remove surrounding quotes `''` and double quotes `""` (see global option `-q`
  and `-qq`)
- remove leading `./` for relative paths (see global option `-d`)
- remove trailing `/`
- expand leading `~` to users home directory path (see global option `-t`)
- print path to stdout (see global option `-F`)

The stdin can be ignored with option `-z`. Double dash `--` tells the script to
stop looking for options that start with a dash (such as `-t`) and treat
everything after it as an input path instead. List all available global options
with `-h` or `--help` .

### Colorize the path separator

Option `-s` for short or `--sep-style` will change the look of the path
separator, which is the slash `/` by default. Use option `--explain style`
to list available colors and effects.

```bash
fpath --explain style

fpath -s red
```

### Advanced formatting

Option `-F` for short or `--format` controls look and structure of output
entirely. Using this option, by default nothing is output unless you tell it
with specific control sequences. These control sequences (also sometimes called
commands) have to be enclosed between curly braces `{` and `}` to be recognized
and will be replaced with their evaluated content. Use option `--explain fmt`
to list available control sequences.

```bash
fpath --explain fmt

fpath -F {path}
```

Note: Advanced formatting with `-F` also supports styles, but they have to be
put in curly braces too here. Also style commands can have an end tag to
specify what portion should be affected, in example `{red}some text{/color}` .

## Documentation

For more in depth explanation, have a look at the docs:

- [Literals](doc/literals.md): A list of escape sequences, that are
  translated into a specific character or control sequence for the terminal.
- [Styles](doc/styles.md): List of available style commands.
- [Fmt](doc/fmt.md): List of available format commands.

And there is a step by step tutorial going through some commands and showing
their output:

- [Tutorial](doc/tutorial.md): Starts off with basic command usage and
  gradually gets more advanced.
