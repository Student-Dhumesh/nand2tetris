# Project 1: Boolean Logic

[← All projects](../README.md)

Build a set of elementary logic gates, using only `Nand` as the starting primitive (and previously built gates after that).

## Contents

| Elementary logic gates    | 16-bit variants       | Multi-way variants            |
| ------------------------- | --------------------- | ----------------------------- |
| [Not](./Not.hdl)          | [Not16](./Not16.hdl)  |                               |
| [And](./And.hdl)          | [And16](./And16.hdl)  |                               |
| [Or](./Or.hdl)            | [Or16](./Or16.hdl)    | [Or8Way](./Or8Way.hdl)        |
| [Xor](./Xor.hdl)          |                       |                               |
| [Mux](./Mux.hdl)          | [Mux16](./Mux16.hdl)  | [Mux4Way16](./Mux4Way16.hdl)  |
|                           |                       | [Mux8Way16](./Mux8Way16.hdl)  |
| [DMux](./DMux.hdl)        |                       | [DMux4Way](./DMux4Way.hdl)    |
|                           |                       | [DMux8Way](./DMux8Way.hdl)    |

---

## Elementary logic gates

### Not

`Not(in) = Nand(in, in)`

| in | out |
|:--:|:---:|
| 0  | 1   |
| 1  | 0   |

### And

`And(a, b) = Not(Nand(a, b))`

| a | b | out |
|:-:|:-:|:---:|
| 0 | 0 | 0   |
| 0 | 1 | 0   |
| 1 | 0 | 0   |
| 1 | 1 | 1   |

### Or

`Or(a, b) = Nand(Not(a), Not(b))` (De Morgan's law)

| a | b | out |
|:-:|:-:|:---:|
| 0 | 0 | 0   |
| 0 | 1 | 1   |
| 1 | 0 | 1   |
| 1 | 1 | 1   |

### Xor

`Xor(a, b) = Or(And(a, Not(b)), And(Not(a), b))`, i.e. `(a AND NOT b) OR (NOT a AND b)`

| a | b | out |
|:-:|:-:|:---:|
| 0 | 0 | 0   |
| 0 | 1 | 1   |
| 1 | 0 | 1   |
| 1 | 1 | 0   |

### Mux (multiplexer)

Selects one of two inputs: `if sel == 0 then out = a, else out = b`

| a | b | sel | out |
|:-:|:-:|:---:|:---:|
| 0 | 0 | 0   | 0   |
| 0 | 1 | 0   | 0   |
| 1 | 0 | 0   | 1   |
| 1 | 1 | 0   | 1   |
| 0 | 0 | 1   | 0   |
| 0 | 1 | 1   | 1   |
| 1 | 0 | 1   | 0   |
| 1 | 1 | 1   | 1   |

### DMux (demultiplexer)

Routes the input to one of two outputs: `if sel == 0 then {a = in, b = 0}, else {a = 0, b = in}`

| in | sel | a | b |
|:--:|:---:|:-:|:-:|
| 0  | 0   | 0 | 0 |
| 0  | 1   | 0 | 0 |
| 1  | 0   | 1 | 0 |
| 1  | 1   | 0 | 1 |

---

## 16-bit variants

Each 16-bit gate applies its 1-bit version to every bit independently: `out[i] = Gate(in[i])` (or `Gate(a[i], b[i])`) for `i = 0..15`.

| Gate  | Inputs                         | Output      | Behaviour                                         |
| ----- | ------------------------------ | ----------- | ------------------------------------------------- |
| Not16 | `in[16]`                       | `out[16]`   | `out[i] = Not(in[i])`                             |
| And16 | `a[16]`, `b[16]`               | `out[16]`   | `out[i] = And(a[i], b[i])`                        |
| Or16  | `a[16]`, `b[16]`               | `out[16]`   | `out[i] = Or(a[i], b[i])`                         |
| Mux16 | `a[16]`, `b[16]`, `sel`        | `out[16]`   | `out = a` if `sel == 0`, else `out = b`           |

Example (`And16`):

```
a   = 1111 0000 1010 1010
b   = 1010 1010 1010 0101
out = 1010 0000 1010 0000
```

---

## Multi-way variants

### Or8Way

`out = in[0] Or in[1] Or ... Or in[7]`, built as a tree of 7 `Or` gates.

| in[0..7]            | out |
|:-------------------:|:---:|
| `00000000`          | 0   |
| any bit set to 1    | 1   |

### Mux4Way16

Two-bit selector, picks one of four 16-bit inputs. Built from two `Mux16`s feeding into a third.

| sel[1] | sel[0] | out |
|:------:|:------:|:---:|
| 0      | 0      | a   |
| 0      | 1      | b   |
| 1      | 0      | c   |
| 1      | 1      | d   |

### Mux8Way16

Three-bit selector, picks one of eight 16-bit inputs. Built from two `Mux4Way16`s feeding into a `Mux16`.

| sel[2] | sel[1] | sel[0] | out |
|:------:|:------:|:------:|:---:|
| 0      | 0      | 0      | a   |
| 0      | 0      | 1      | b   |
| 0      | 1      | 0      | c   |
| 0      | 1      | 1      | d   |
| 1      | 0      | 0      | e   |
| 1      | 0      | 1      | f   |
| 1      | 1      | 0      | g   |
| 1      | 1      | 1      | h   |

### DMux4Way

Routes `in` to one of four outputs; all others are 0. Built as a tree of 3 `DMux` gates (one feeding two).

| sel[1] | sel[0] | a  | b  | c  | d  |
|:------:|:------:|:--:|:--:|:--:|:--:|
| 0      | 0      | in | 0  | 0  | 0  |
| 0      | 1      | 0  | in | 0  | 0  |
| 1      | 0      | 0  | 0  | in | 0  |
| 1      | 1      | 0  | 0  | 0  | in |

### DMux8Way

Routes `in` to one of eight outputs; all others are 0. Built as a three-level tree of 7 `DMux` gates.

| sel[2] | sel[1] | sel[0] | a  | b  | c  | d  | e  | f  | g  | h  |
|:------:|:------:|:------:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 0      | 0      | 0      | in | 0  | 0  | 0  | 0  | 0  | 0  | 0  |
| 0      | 0      | 1      | 0  | in | 0  | 0  | 0  | 0  | 0  | 0  |
| 0      | 1      | 0      | 0  | 0  | in | 0  | 0  | 0  | 0  | 0  |
| 0      | 1      | 1      | 0  | 0  | 0  | in | 0  | 0  | 0  | 0  |
| 1      | 0      | 0      | 0  | 0  | 0  | 0  | in | 0  | 0  | 0  |
| 1      | 0      | 1      | 0  | 0  | 0  | 0  | 0  | in | 0  | 0  |
| 1      | 1      | 0      | 0  | 0  | 0  | 0  | 0  | 0  | in | 0  |
| 1      | 1      | 1      | 0  | 0  | 0  | 0  | 0  | 0  | 0  | in |

> **Note:** `sel[0]` is the least significant bit. `sel = 011` selects the 4th output (`d`).

---

## NAND count

Minimum number of `Nand` gates when each gate is built as efficiently as possible.

| Gate      | NANDs used | How                                                         |
| --------- | :--------: | ----------------------------------------------------------- |
| Not       | 1          | `Nand(in, in)`                                              |
| And       | 2          | `Nand` + `Not`                                              |
| Or        | 3          | `Nand` + two `Not`                                          |
| Xor       | 4          | Optimised NAND-only form (see note below)                   |
| Mux       | 4          | `Nand(Nand(a, Not(sel)), Nand(b, sel))`                     |
| DMux      | 5          | `And(in, Not(sel))` and `And(in, sel)`                      |
| Not16     | 16         | 16 × Not                                                    |
| And16     | 32         | 16 × And                                                    |
| Or16      | 48         | 16 × Or                                                     |
| Mux16     | 64         | 16 × Mux                                                    |
| Or8Way    | 21         | 7 × Or                                                      |
| Mux4Way16 | 192        | 3 × Mux16                                                   |
| Mux8Way16 | 448        | 7 × Mux16                                                   |
| DMux4Way  | 15         | 3 × DMux                                                    |
| DMux8Way  | 35         | 7 × DMux                                                    |

> <br/> **Note on Xor:** building it from `Or`, `And` and `Not` as in the formula above costs 9 NANDs (2 × Not + 2 × And + 1 × Or = 2 + 4 + 3). The optimised version needs only 4 because it shares one intermediate result:
>
> ```
> n   = Nand(a, b)
> x   = Nand(a, n)
> y   = Nand(b, n)
> out = Nand(x, y)
> ```
> <br/>