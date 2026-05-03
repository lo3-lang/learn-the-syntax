# Syntax Reference

## Line Structure

Every lo3 instruction occupies one line and follows this format:

```
#<cmd> <arg0> <arg1>
```

| Part | Description |
| :--- | :--- |
| `#` | Starting character (default). Can be changed with `@.` syntax sugar. |
| `<cmd>` | A single character identifying the command. |
| `<arg0>` | First argument — a **type-prefixed** value. |
| `<arg1>` | Second argument — a **type-prefixed** value. |

Arguments are separated by spaces. A space inside an argument starts the next argument.

Any line **not** beginning with the starting character is ignored (treated as a comment). There is no dedicated comment syntax — you can write anything.

---

## Type Prefixes

Every argument starts with a **type prefix** character, followed immediately by the value.

| Prefix | Type | Description | Example |
| :--- | :--- | :--- | :--- |
| `$` | Integer | Signed integer literal | `$100`, `$0`, `$-5` |
| `_` | String | String literal (no quotes needed) | `_Hello`, `_myVar` |
| `%` | Var resolve | Resolves the named variable and substitutes its value | `%counter` |
| `*` | Global array resolve | Resolves `g[index]` — the index must be a number | `*0`, `*100` |
| `/` | Double | Floating-point literal *(planned, not yet implemented)* | `/3.14` |

### How `%` (Var Resolve) Works

`%name` looks up the variable `name` and replaces the argument with that variable's current value **before** the command runs.

```
#n _abc $0
#= _abc $42
#o %abc $0
```

Here `%abc` resolves to `$42`, so this prints `42`. Be careful: because the name is replaced by the raw value, using `%name` as a write target (e.g., `#= %abc $10`) will fail — the name is gone after resolution.

### How `*` (Global Array Resolve) Works

`*N` fetches the value stored at `g[N]` and substitutes it as the argument.

If `g[100]` contains the string `myVar`, then `*100` resolves to `_myVar`.
If `g[100]` contains the integer `123`, then `*100` resolves to `$123`.

---

## Commands

### Variables

| Command | Syntax | Description |
| :--- | :--- | :--- |
| `#n` | `#n <name> <type>` | **New** — Create a variable. `<type>`: `$0` = integer, `$3` = string. |
| `#=` | `#= <name> <value>` | **Assign** — Set a variable's value. |
| `#f` | `#f <name> <any>` | **Free** — Delete a variable. `arg1` is ignored (use `$0` as placeholder). |

```
#n _score $0
#= _score $100
#f _score $0
```

### I/O

| Command | Syntax | Description |
| :--- | :--- | :--- |
| `#o` | `#o <value> <any>` | **Output** — Print `arg0` to stdout. `arg1` is ignored. |
| `#i` | `#i <name> <type>` | **Input** — Read from stdin into the variable `<name>`. `<type>` determines parsing. |

```
#o _Hello $0
#o $42 $0

#n _answer $0
#i _answer $0
```

### ALU (Arithmetic)

All ALU commands modify the variable named by `arg0` in place.

| Command | Syntax | Operation |
| :--- | :--- | :--- |
| `#+` | `#+ <name> <int>` | `name += int` |
| `#-` | `#- <name> <int>` | `name -= int` |
| `#*` | `#* <name> <int>` | `name *= int` |
| `#/` | `#/ <name> <int>` | `name /= int` (division by zero is an error) |

```
#n _x $0
#= _x $10
#+ _x $5
#o %x $0
```

This prints `15`.

### Comparison

Comparison commands store the result in **`g[0]`** (the first slot of the global array): `1` = true, `0` = false. Both arguments must be integers.

| Command | Syntax | Condition |
| :--- | :--- | :--- |
| `#?` | `#? <int> <int>` | `arg0 == arg1` |
| `#<` | `#< <int> <int>` | `arg0 < arg1` |
| `#>` | `#> <int> <int>` | `arg0 > arg1` |

```
#? $10 $10
#o *0 $0
```

Prints `1` (true, because 10 == 10).

### Control Flow (Labels & Jumps)

| Command | Syntax | Description |
| :--- | :--- | :--- |
| `#l` | `#l <name> <any>` | **Label** — Define a named jump target. |
| `#d` | `#d <label> <int>` | **Jump** — If `g[0] == arg1`, jump to `<label>`. |

The jump command (`#d`) checks the global array slot `g[0]` against `arg1`. If they match, execution jumps to the label. Labels are resolved by name.

```
#l _loop $0
#+ _x $1
#? %x $10
#d _end $1
#d _loop $0
#l _end $0
```

### External Calls *(not yet implemented)*

| Command | Syntax | Description |
| :--- | :--- | :--- |
| `#c` | `#c <path> <any>` | **Call** — Call external library (fixed path). |
| `#C` | `#C <path> <any>` | **Call (relative)** — Call external library (relative path). |

### Program Exit

| Command | Syntax | Description |
| :--- | :--- | :--- |
| `#0` | `#0 <any> <any>` | Exit with code **0** (success). |
| `#1` | `#1 <any> <any>` | Exit with code **1** (error). |

Both arguments are ignored — convention is `#0 $0 $0`.

---

## Global Array `g[]`

lo3 provides a global array of 100 slots (`g[0]` through `g[99]`) that can store integers or strings.

- **Write** to `g[]` by assigning to the special name `_gN` (e.g., `_g0`, `_g42`):
  ```
  #= _g0 $100
  ```
- **Read** from `g[]` using the `*` prefix (e.g., `*0`):
  ```
  #o *0 $0
  ```
- Comparison commands (`#?`, `#<`, `#>`) write their result to `g[0]`.
- The jump command (`#d`) reads from `g[0]` to decide whether to jump.

### Batch Initialization

Use `@{...}` to initialize multiple global array slots on one line:

```
@{0:$10,1:_Hello,5:$99}
```

Format: `@{index:value,index:value,...}`

---

## Syntax Sugar

### Changing the Starting Character

By default, `#` is the starting character for commands. You can change it with:

```
@.X
```

where `X` is the new starting character. After this line, only lines beginning with `X` are treated as commands.

```
@.!
!o _Now-using-exclamation $0
!0 $0 $0
```

---

## Summary Table

| Prefix / Cmd | Character | Purpose |
| :--- | :--- | :--- |
| **Type: integer** | `$` | Integer literal |
| **Type: string** | `_` | String literal |
| **Type: var** | `%` | Resolve variable by name |
| **Type: g[]** | `*` | Resolve global array slot |
| **Type: double** | `/` | Double literal *(planned)* |
| **Cmd: new** | `n` | Create variable |
| **Cmd: assign** | `=` | Assign to variable |
| **Cmd: free** | `f` | Delete variable |
| **Cmd: output** | `o` | Print to stdout |
| **Cmd: input** | `i` | Read from stdin |
| **Cmd: add** | `+` | Add to variable |
| **Cmd: sub** | `-` | Subtract from variable |
| **Cmd: mul** | `*` | Multiply variable |
| **Cmd: div** | `/` | Divide variable |
| **Cmd: compare** | `?` | Equality check → `g[0]` |
| **Cmd: less than** | `<` | Less-than check → `g[0]` |
| **Cmd: greater than** | `>` | Greater-than check → `g[0]` |
| **Cmd: label** | `l` | Define label |
| **Cmd: jump** | `d` | Conditional jump to label |
| **Cmd: call** | `c` | External call *(not impl.)* |
| **Cmd: call relative** | `C` | External call relative *(not impl.)* |
| **Cmd: exit 0** | `0` | Exit success |
| **Cmd: exit 1** | `1` | Exit error |

