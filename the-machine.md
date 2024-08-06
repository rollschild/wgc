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

- most floating-point formats use some number of bits to represent
  - a **mantissa**, and
  - a smaller number of bits to represent an exponent
- **mantissa** - a base value
  - usually falls within a limited range
- **exponent** - a multiplier
  - applied to mantissa, producing values outside of the range
- **The order of evaluation can affect the accuracy of the result.**
- **When performing a chain of calculations involving addition, subtraction, multiplication, and division, perform the multiplication and division operations first.**
- **When multiplying and dividing sets of numbers, try to multiply and divide numbers that have the same relative magnitudes.**
- You should _NEVER_ compare two floating-point values to see if they are equal
  - use `if (abs(value1 - value2) <= error)`
- the **miserly approach** vs. **eager approach**
  - **miserly approach**
    - compare whether two values are equal - within error tolerance
    - if not equal, compare to see if one is less than (or greater than) the other
  - **eager approach**
    - make the result of the comparison `true` as often as possible
    ```
    if (A < (B + error)) then Eager_A_lessthan_B;
    if (A > (B - error)) then Eager_A_greaterthan_B;
    ```
- the **KCS Floating-Point Standard**
  - single precision
    - 24-bit mantissa
    - 8-bit exponent
    - one's complement
  - double precision
    - 53-bit mantissa
    - 11-bit excess-1023 exponent
  - extended precision
    - uses 80 bits
    - 64-bit mantissa
    - 15-bit excess-16383 exponent
- on 80x86 FPUs, _all_ computations use extended-precision form
- **normalized** vs. **denormalized** values
  - a floating-point computation will be more accurate if it involves _only_ normalized values
  - because the mantissa has that many fewer bits of precision available for computation if HO bits of the mantissa are all `0`
- **guard bits** - maintain the extra precision
- **rounding** - facilitated by guard bits
  - truncation
    - standard for coercing floating-point value to integer
  - rounding up
  - rounding down
  - rounding to nearest
- **SNaN** vs. **QNaN** values
  - if the exponent contains _all_ `1`s, and
  - the mantissa is non-zero (discounting the implied bit)
  - then HO bit of mantissa (discounting the implied bit) determines whether the value represents a **quiet non-a-number** (QNaN) or **signaling not-a-number** (SNaN)
  - **QNaN** - indeterminate results
  - **SNaN** - invalid operation
- **+infinity**/**-infinity**
  - when exponent has all `1`s and mantissa has all `0`s
- `-0`/`+0` - if exponent bits are all `0`
  - indicated by sign bit
  - Intel recommends using the sign bit to indicate `0` was produced via:
    - underflow of a negative value (with sign bit set), or
    - underflow of positive value (sign bit clear)

### Floating-Point Operations

#### Addition/Subtraction

- Use 32-bit `unsigned` integer type to hold the bit representation for floating-poing values
- To ask C/C++ compiler to treat the bit pattern it finds in the `uint32_t` as a `float` without doing any conversion:
  ```c
  typedef uint32_t real;
  #define asreal(x) (*((float *) &x))
  ```
- addition and subtraction
  ```c
  void fpadd(real left, real right, real* dest);
  void fpsub(real left, real right, real* dest);
  ```
- subtraction
  ```c
  void fpsub(real left, real right, real* dest) {
    right = right ^ 0x80000000; // invert sign bit of the right operand
    fpadd(left, right, dest);
  }
  ```
- Unpack the sign
  ```c
  inline int extract_sign(real from) {
    return (from >> 31);
    // a possibly more efficient way:
    // return (from & 0x80000000) != 0;
  }
  ```
- Extract exponent
  ```c
  inline int extract_exponent(real from) {
    return ((from >> 23) & 0xff) - 127;
  }
  ```
- Extract mantissa
  ```c
  inline int extract_mantissa(real from) {
    if ((from & 0x7fffffff) == 0) return 0;
    // mask out the exponent and sign bits
    // then insert the implied HO bit of 1
    return ((from & 0x7FFFFF) | 0x800000);
  }
  ```
- _Shifting mantissa bits to the right -> reduce the precision of the number_
  - so we should _NOT_ truncate the bits we shift out of mantissa
  - instead, we should round the result to the nearest value we can represent with remaining mantissa bits
- IEEE shifting & rounding rules:
  - truncate the result if the last bit shifted out was a `0`
  - increment the mantissa by `1` if
    - the last bit shifted out was a `1`, and
    - there was at least one bit set to `1` in all other bits shifted out
  - if the last bit shifted out was a `1`, and all other bits (shifted out) were `0`s,
    - then round the resulting mantissa up by `1` if the mantissa's LO bit contains a `1`
- Shifting and rounding

  ```c
  void shift_and_round(uint32_t* val_to_shift, int bits_to_shift) {
    // to mask out bits to check for "sticky" bits
    static unsigned masks[24] = {
      0, 1, 3, 7, 0xf, 0x1f, 0x3f, 0x7f,
      0xff, 0x1ff, 0x3ff, 0x7ff, 0xfff, 0x1fff, 0x3fff, 0x7fff,
      0xffff, 0x1ffff, 0x3ffff, 0x7ffff, 0xfffff, 0x1fffff, 0x3fffff, 0x7fffff
    };

    static unsigned HO_masks[24] = {
      0, 1, 2, 4, 0x8, 0x10, 0x20, 0x40, 0x80,
      0x100, 0x200, 0x400, 0x800, 0x1000, 0x2000, 0x4000, 0x8000,
      0x10000, 0x20000, 0x40000, 0x80000, 0x100000, 0x200000, 0x400000,
    };

    // holds the value that will be shifted out of mantissa during
    // denormalization
    int shifted_out;

    assert(bits_to_shift <= 23);
    shifted_out = *val_to_shift & masks[bits_to_shift];
    *val_to_shift = *val_to_shift >> bits_to_shift;

    if (shifted_out > HO_masks[bits_to_shift]) {
      // if bits shifted out are greater than 1/2 of the LO bit, then
      // round the value up by 1
      // rule applied:
      // increment the mantissa by `1` if
      //   the last bit shifted out was a `1`, and
      //   there was at least one bit set to `1` in all other bits shifted out
      *val_to_shift = *val_to_shift + 1;
    } else if (shifted_out == HO_masks[bits_to_shift]) {
      // if exactly 1/2 of LO bit's value,
      // round the value to the nearest number whose LO bit is 0
      // rule applied:
      // if the last bit shifted out was a `1`, and all other bits (shifted out) were `0`s,
      //   then round the resulting mantissa up by `1` if the mantissa's LO bit contains a `1`
      *val_to_shift = *val_to_shift + (*val_to_shift & 1);
    }
  }
  ```

- Pack the results

  ```c
  inline real pack_fp(int sign, int exponent, int mantissa) {
    return (real)(
      (sign << 31)
      | ((exponent + 127) << 23)
      | (mantissa & 0x7fffff)
    );
  }
  ```

- `fpadd`

  ```c
  void fpadd(real left, real right, real* dest) {
    int l_exponent;
    uint32_t l_mantissa;
    int l_sign;

    int r_exponent;
    uint32_t r_mantissa;
    int r_sign;

    l_exponent = extract_exponent(left);
    l_mantissa = extract_mantissa(left);
    l_sign = extract_sign(left);

    r_exponent = extract_exponent(right);
    r_mantissa = extract_mantissa(right);
    r_sign = extract_sign(right);

    // special operands (infinity and NaNs)
    if (l_exponent == 127) {
      if (l_mantissa == 0) {
        // depends on value of the right operand
        if (r_exponent == 127) {
          // either:
          //   - infinity (zero mantissa)
          //   - QNaN (mantissa = 0x80000000)
          //   - SNaN (non-zero mantissa not equal to 0x80000000)
          if (r_mantissa == 0) {
            // infinity + infinity = infinity
            // -infinity - infinity = -infinity
            // -infinity + infinity = NaN
            // infinity - infinity = NaN
            if (l_sign == r_sign) {
              *dest = right;
            } else {
              *dest = 0x6fC00000; // +QNaN
            }
          } else {
            // NaN
            *dest = right;
          }
        }
      } else {
        // l_mantissa non-zero; l_exponent all 1s
        *dest = left;
      }
      return;
    } else if (r_exponent == 127) {
      // right is either NaN or +infinity/-infinity
      *dest = right;
      return;
    }

    // now we have two actual floating point values
    d_exponent = r_exponent;
    if (r_exponent > l_exponent) {
      shift_and_round(&l_mantissa, (r_exponent - l_exponent));
    } else if (r_exponent < l_exponent) {
      shift_and_round(&r_mantissa, (l_exponent - r_exponent));
      d_exponent = l_exponent;
    }

    if (r_sign ^ l_sign) {
      // signs are _different_
      if (l_mantissa > r_mantissa) {
        d_mantissa = l_mantissa - r_mantissa;
        d_sign = l_sign;
      } else {
        d_mantissa = r_mantissa - l_mantissa;
        d_sign = r_sign;
      }
    } else {
      // signs are same
      d_sign = l_sign;
      d_mantissa = l_mantissa + r_mantissa;
    }

    // normalize the result
    // note that *overflow* can happen
    if (d_mantissa >= 0x1000000) {
      shift_and_round(&d_mantissa, 1);
      ++d_exponent;
    } else {
      if (d_mantissa != 0) {
        while ((d_mantissa < 0x800000) && (d_exponent > -127)) {
          d_mantissa = d_mantissa << 1;
          --d_exponent;
        }
      } else {
        d_sign = 0;
        d_exponent = 0;
      }
    }

    *dest = pack_fp(d_sign, d_exponent, d_mantissa);
  }
  ```
