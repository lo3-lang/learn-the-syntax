# lo3 Syntax Reference

**lo3** is a token-based interpreted programming language written in C. Every instruction follows a strict line format built around **prefixes** and **type sigils**.

## Line Format

```
#<cmd> <arg0> <arg1>
```

- Lines starting with `#` are instructions (by default — see [Syntax Sugar](#syntax-sugar) to change the starting character).
- Everything else is a comment — no special comment character required.
- Every command takes exactly **two arguments**, separated by spaces.

## Quick Example

```
#n _counter $0
#= _counter $10
#o _counter $0
#f _counter $0
#0 $0 $0
```

This creates a variable `counter`, assigns it the value `10`, prints it, frees it, and exits with code 0.

## Contents

- [Syntax Reference](syntax.md) — Full reference for types, commands, control flow, and the global array.
- [Examples](examples.md) — Annotated `.lo3` example files.

