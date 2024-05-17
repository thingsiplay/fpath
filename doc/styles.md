# Styles

[fpath](https://github.com/thingsiplay/fpath)

An incomplete list of available styles. See `--explain style` for a more
exhaustive list.

```bash
fpath --explain style

fpath --explain style_compact

fpath -s STYLE

fpath -F {STYLE}
```

## globals

These will reset any custom effects and of colors for following text. Usually
it is a good advice to reset at the line end, so the next line of output will
start off fresh.

```
{}
{default}
```

Example:

```bash
fpath -F {red}hello{blink}world{}
```

## reset colors

Reset custom colors at this end marker. There are dedicated font color and
background color commands, but also a combined `{/}` one to reset both at the
same time. Effects are unaffected with this.

```
{/}
{/color}
{/bg.color}
```

Example:

```bash
fpath -F {red}hello{/}world
```

## effects

Any text between the starting and closing tag will show their associated
effect.

```
{bold} {/bold}
{italic} {/italic}
{dim} {/dim}
{underline} {/underline}
{reverse} {/reverse}

{}
{default}
```

Example:

```bash
fpath -s blink

fpath -F {blink}hello{/blink}
```

## foreground colors

Any text after the tag will be recolored. There is no dedicated closing tag
name, but a general purpose `{/color}` to reset the foreground color up until
this point. The same colors exist in a second variant, called bright. Their
name are identical, but with a `b` prefix.

```
{dark}
{white}
{red}
{green}
{blue}
{yellow}
{magenta}
{cyan}

{bdark}
{bwhite}
{bred}
...

{/}
{/color}
{}
{default}
```

Example:

```bash
fpath -s red

fpath -F {red}text{/color}
```

## background colors

The same colors from foreground colors can be assigned to the background of a
text range as well. The names are identical, but have a `bg.` in front, and
`bg.b` for the bright variants respectively. Similarly to reset the background
color, there is a dedicated `{/bg.color}` command.

```
{bg.dark}
{bg.white}
{bg.red}
{bg.green}
{bg.blue}
{bg.yellow}
{bg.magenta}
{bg.cyan}

{bg.bdark}
{bg.bwhite}
{bg.bred}
...

{/}
{/bg.color}
{}
{default}
```

Example:

```bash
fpath -s bg.red

fpath -F {bg.red}text{/bg.color}
```
