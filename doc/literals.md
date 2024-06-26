# Literals

[fpath](https://github.com/thingsiplay/fpath)

A list of escape sequences, that are translated into a specific character or
control sequence for the terminal. A simple "backslash" and "n" will be
converted into a single newline character. This is an incomplete listing here.
Use option `--explain fmt` to list a more exhaustive list.

```bash
fpath --explain fmt

fpath -/ '\n'

fpath -F '\n'
```

## Escape sequences

```
"\\": literal single slash "\"
"\{": literal "{"
"\}": literal "}"
"\q": literal single quote '
"\Q": literal double quote "
"\n": newline
"\t": tab
"\r": carriage return
"\f": form feed
"\b": backspace
"\s": non breakable space
```

Example:

```bash
fpath -F 'hello\n\tworld'
```
