# Understanding the Machine

## Numeric Representation

- **positional notation**
- **radix**
- **radix point**

### Binary

- Two conventions to represent binary in assemblers:
  - **MASM**
    - `1001b` - `9`
  - **HLA**
    - `%11_1011_0010_1101`

### Hex

- prefix `0x`
- **MASM** uses `h`/`H` suffix
  - `0deadh` - `DEAD` base 16
- **HLA** uses prefix `$`
  - `$FDEC_A012`

### Octal

- in C, prefix `0`
  - `0123` - decimal `83` (64 + 2 \* 8 + 3)
- **MASM** uses suffix `Q`/`q`

### Numeric/String conversions

- expensive
- floating-point values are most complicated & difficult string <-> numeric

### Internal Numeric Representation

- **nibble** - collection of 4 bits
- **byte** - 8 bits
  - smallest addressable data item on many CPUs
- a **word** can have different definitions on different CPUs
  - 16/32/64 bits
  - LO byte and HO byte
- Intel 80x86 platforms support **tbyte** - 80-bit type
- **two's complement**
  - `-2^(n-1)` to `2^(n-1) - 1`
- **sign extension** & **sign contraction** operations
  - Assigning a smaller integer to a larger one may require _more_ machine instructions (longer to execute) than moving data between two like-sized variables

### Saturation

- **saturation**
  - reduce size of an integer value
  - loss of precision!
- by copying LO bits of the larger object into the smaller one

### Binary-Coded Decimal Representation

- BCD consists of a sequence of **nibbles**
- encodes decimal values using a binary representation
- COBOL
- For most calculations, binary is _more_ accurate
  - although for _certain_ calculations, BCD can be more accurate

### Fixed-Point Representation

- format: `101.01` - `5.25`
- **FPU** - floating-point unit
- more cost-effective to use CPU's native floating-point format
- can _only_ represent a small subset of the real numbers

### Scaled Numeric Formats

- efficient

## BINARY ARITHMETIC AND BIT OPERATIONS

- `isOdd = (ValueToTest & 1) != 0`
- `isDivisibleBy16 = (ValueToTest & 0xf) == 0`
  - check if the LO 4 bits are all `0`s
- **modulo-n counter**
  - in C/C++: `cntr = (cntr + 1) % n`
  - however, division is _expensive_ - requiring far more time to execute than addition
  - a more efficient version:
    ```c
    cntr += 1;
    if (cntr >= n) cntr = 0;
    ```
  - bit **AND** is _much faster_ than division
  - on most CPUs, using AND operator is quite a bit faster than using `if`
  - a more efficient version for n = 32:
  ```c
  cntr = (cntr + 1) & 0x1f // AND with value 2^m - 1
  ```
  - assembly:
  ```assembly
  inc(eax);
  and($1f, eax);
  ```
- **arithmetic shift right**
  - to divide a _signed_ number by 2
  - the HO bit does _not_ change
- _rare_ for a high-level language to support both
  - logical shift right, and
  - arithmetic shift right
- How to simulate a 32-bit logical shift right _and_ arithmetic shift right, when the language does _NOT_ guarantee the type of shift used

  ```c
  // assuming 32-bit integers
  // logical shift right:
  bit30 = ((value_to_be_shifted & 0x80000000) != 0) ? 0x40000000 : 0;
  // shift bits 0..30
  value_to_be_shifted = (value_to_be_shifted & 0x7fffffff) >> 1;
  // merge in bit #30
  value_to_be_shifted = value_to_be_shifted | bit30;

  // arithmetic shift right:
  bit3031 = ((value_to_be_shifted & 0x80000000) != 0) ? 0xc0000000 : 0;
  // shift bits 0..30
  value_to_be_shifted = (value_to_be_shifted & 0x7fffffff) >> 1;
  // merge in bit #30 and #31
  value_to_be_shifted = value_to_be_shifted | bit3031;
  ```

- Built-in support for packed data in C:
  ```c
  // the :n specifies the minimum number of bits allocated by compiler
  struct {
  unsigned bits0_3 :4;
  unsigned bits4_11 :8;
  unsigned bits12_15 :4;
  unsigned bits16_23 :8;
  unsigned bits24_31 :8;
  } packed_data;
  ```
  - this ^^^ are almost guaranteed to be _nonportable_
  - Fields that begin at position 0 in a packed data object can be accessed most efficiently
    - arrange the fields in packed data type such that the field accessed the most begins at 0
- in HLA/x86 assembly, data can be accessed at any arbitrary **byte boundary** in memory
  - `movzx((type byte packed_value), eax);`
  - `movzx` - move with zero extension

## Floating-Point Representation

-
