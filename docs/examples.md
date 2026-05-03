# Examples

Annotated lo3 examples showing the syntax in action. Each example is also available as a `.lo3` file in the `examples/` directory.

---

## 01 — Hello World

```
#o _Hello-World $0
#0 $0 $0
```

| Line | What it does |
| :--- | :--- |
| `#o _Hello-World $0` | Print the string `Hello-World` to stdout. `$0` is a dummy second argument. |
| `#0 $0 $0` | Exit the program with code 0. |

Any line that does **not** start with `#` is simply ignored — no comment character needed.

---

## 02 — Type Prefixes

```
#o $100 $0
#o _hello $0
#o /1.0 $0
#o *0 $0
#0 $0 $0
```

| Line | Prefix used | Output |
| :--- | :--- | :--- |
| `#o $100 $0` | `$` — integer | `100` |
| `#o _hello $0` | `_` — string | `hello` |
| `#o /1.0 $0` | `/` — double *(planned)* | — |
| `#o *0 $0` | `*` — global array resolve | value of `g[0]` |

---

## 03 — Variables

```
#n _myVar $0
#= _myVar $100
#o %myVar $0
#f _myVar $0
#0 $0 $0
```

| Line | Explanation |
| :--- | :--- |
| `#n _myVar $0` | Create an integer variable named `myVar`. (`$0` = integer type, `$3` = string type) |
| `#= _myVar $100` | Assign `100` to `myVar`. |
| `#o %myVar $0` | Resolve `myVar` and print its value (`100`). |
| `#f _myVar $0` | Free the variable. |

### Why `%` matters

Using `%myVar` resolves the variable **before** the command sees it. So:

```
#= %myVar $100
```

becomes (if `myVar` currently holds `50`):

```
#= $50 $100
```

This is an error — there is no variable named `50`. Always use `_name` when writing, and `%name` when reading.

---

## 04 — Global Array

```
#= _g0 $100
#o *0 $0
#0 $0 $0
```

| Line | Explanation |
| :--- | :--- |
| `#= _g0 $100` | Assign `100` to global array slot 0. `_gN` is special syntax: if the character after `g` is a digit, it targets `g[N]` instead of a regular variable. |
| `#o *0 $0` | Resolve `g[0]` and print it (`100`). |

Names like `_GOOD` or `_g100ee` are still regular variables — the parser only treats `_g` + digits as a global array write.

### Batch Init

```
@{0:$10,1:_Hello,5:$99}
```

Sets `g[0] = 10`, `g[1] = "Hello"`, `g[5] = 99` in a single line.

---

## 05 — Arithmetic

```
#n _x $0
#= _x $10
#+ _x $3
#- _x $1
#* _x $4
#/ _x $2
#o %x $0
#f _x $0
#0 $0 $0
```

Step by step: `x = 10` → `+3 = 13` → `-1 = 12` → `*4 = 48` → `/2 = 24`. Prints `24`.

---

## 06 — Comparison & Conditional Jump

```
#n _i $0
#= _i $0

#l _loop $0
#+ _i $1
#o %i $0

#? %i $5
#d _done $1

#d _loop $0

#l _done $0
#o _finished $0
#f _i $0
#0 $0 $0
```

This counts from `1` to `5`:

1. `#l _loop $0` — define the label `loop`.
2. Increment `i`, print it.
3. `#? %i $5` — compare `i == 5`, result stored in `g[0]` (1 if true, 0 if false).
4. `#d _done $1` — if `g[0] == 1` (i.e., `i` reached 5), jump to `done`.
5. `#d _loop $0` — if `g[0] == 0` (i.e., `i` has not reached 5), jump back to `loop`.
6. `#l _done $0` — the exit label.

---

## 07 — Changing the Starting Character

```
@.!
!o _Now-using-exclamation $0
!0 $0 $0
```

After `@.!`, the starting character becomes `!` instead of `#`. Only lines beginning with `!` are parsed as instructions.

