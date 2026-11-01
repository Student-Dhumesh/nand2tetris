# Project 2: Boolean Arithmetic

[All projects](../../README.md)

**Given:** the chips built in [Project 1](../01/README.md).

**Goal:** build the chips that perform arithmetic, ending with the ALU, the computational heart of the CPU.

## Contents

| Chip                          | Purpose                                            |
| ----------------------------- | -------------------------------------------------- |
| [HalfAdder](./HalfAdder.hdl)  | Adds two bits                                      |
| [FullAdder](./FullAdder.hdl)  | Adds three bits (two inputs and a carry in)        |
| [Add16](./Add16.hdl)          | Adds two 16-bit numbers                            |
| [Inc16](./Inc16.hdl)          | Adds 1 to a 16-bit number                          |
| [ALU](./ALU.hdl)              | Arithmetic Logic Unit of the Hack computer         |

---

## HalfAdder

Adds two bits and outputs a 2-bit result: `sum` (low bit) and `carry` (high bit).

- `sum = Xor(a, b)`
- `carry = And(a, b)`

| a | b | carry | sum |
|:-:|:-:|:-----:|:---:|
| 0 | 0 | 0     | 0   |
| 0 | 1 | 0     | 1   |
| 1 | 0 | 0     | 1   |
| 1 | 1 | 1     | 0   |

## FullAdder

Adds three bits (`a`, `b`, `c`), where `c` is the carry from the previous column.

A common design uses two half adders: add `a + b`, then add `c` to that sum, and `Or` the two carries.

| a | b | c | carry | sum |
|:-:|:-:|:-:|:-----:|:---:|
| 0 | 0 | 0 | 0     | 0   |
| 0 | 0 | 1 | 0     | 1   |
| 0 | 1 | 0 | 0     | 1   |
| 0 | 1 | 1 | 1     | 0   |
| 1 | 0 | 0 | 0     | 1   |
| 1 | 0 | 1 | 1     | 0   |
| 1 | 1 | 0 | 1     | 0   |
| 1 | 1 | 1 | 1     | 1   |

## Add16

Adds two 16-bit numbers in two's complement. It is a chain of one `HalfAdder` (or a `FullAdder` with carry in 0) for bit 0, followed by 15 `FullAdder`s, each passing its carry to the next bit (a *ripple-carry adder*).

The carry out of the most significant bit is **ignored**, so overflow wraps around.

```
  a   = 0000 0000 0000 0111   (7)
  b   = 0000 0000 0000 0101   (5)
  out = 0000 0000 0000 1100   (12)
```

## Inc16

Computes `out = in + 1` for a 16-bit input. It can be built from an `Add16`, adding the constant `1` (`0000 0000 0000 0001`) to the input. Overflow wraps around, so `-1 + 1 = 0`.

```
  in  = 0000 0000 0000 1111   (15)
  out = 0000 0000 0001 0000   (16)
```

## ALU

The ALU takes two 16-bit inputs `x` and `y`, plus six 1-bit control inputs, and outputs a 16-bit result `out` and two status flags.

| Pin    | Direction | Meaning                                    |
| ------ | --------- | ------------------------------------------ |
| `x[16]`, `y[16]` | in  | Data inputs                              |
| `zx`   | in        | If 1, set `x = 0`                          |
| `nx`   | in        | If 1, set `x = !x`                         |
| `zy`   | in        | If 1, set `y = 0`                          |
| `ny`   | in        | If 1, set `y = !y`                         |
| `f`    | in        | If 1, `out = x + y`; if 0, `out = x & y`  |
| `no`   | in        | If 1, set `out = !out`                     |
| `out[16]` | out    | The result                                 |
| `zr`   | out       | 1 if `out == 0`, else 0                    |
| `ng`   | out       | 1 if `out < 0`, else 0                     |

The operations are applied in this order: `zx` → `nx` on x, `zy` → `ny` on y, then `f`, then `no`.

### Function table

| zx | nx | zy | ny | f | no | out     |
|:--:|:--:|:--:|:--:|:-:|:--:| ------- |
| 1  | 0  | 1  | 0  | 1 | 0  | `0`     |
| 1  | 1  | 1  | 1  | 1 | 1  | `1`     |
| 1  | 1  | 1  | 0  | 1 | 0  | `-1`    |
| 0  | 0  | 1  | 1  | 0 | 0  | `x`     |
| 1  | 1  | 0  | 0  | 0 | 0  | `y`     |
| 0  | 0  | 1  | 1  | 0 | 1  | `!x`    |
| 1  | 1  | 0  | 0  | 0 | 1  | `!y`    |
| 0  | 0  | 1  | 1  | 1 | 1  | `-x`    |
| 1  | 1  | 0  | 0  | 1 | 1  | `-y`    |
| 0  | 1  | 1  | 1  | 1 | 1  | `x + 1` |
| 1  | 1  | 0  | 1  | 1 | 1  | `y + 1` |
| 0  | 0  | 1  | 1  | 1 | 0  | `x - 1` |
| 1  | 1  | 0  | 0  | 1 | 0  | `y - 1` |
| 0  | 0  | 0  | 0  | 1 | 0  | `x + y` |
| 0  | 1  | 0  | 0  | 1 | 1  | `x - y` |
| 0  | 0  | 0  | 1  | 1 | 1  | `y - x` |
| 0  | 0  | 0  | 0  | 0 | 0  | `x & y` |
| 0  | 1  | 0  | 1  | 0 | 1  | `x \| y` |

### Design notes

- Each control bit is a `Mux16` selecting between a value and its modified version. For example, `zx` selects between `x` and `0`, and `nx` selects between the result and `Not16` of it.
- `f` selects between `Add16(x, y)` and `And16(x, y)`.
- `zr` and `ng` need the final `out`. `ng` is simply the sign bit `out[15]`, and `zr` is `Not(Or(all 16 bits))`, which can be built with `Or8Way` on each half of `out`.
- In HDL you cannot read bits of an output pin, so route `out` through an internal pin (for example `outInternal`) and split it as `outInternal[0..7]` and `outInternal[8..15]`.

---