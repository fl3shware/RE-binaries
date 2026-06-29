[Challenge Link](https://crackmes.one/crackme/67fa22568f555589f3530a94)

---

## Overview

This challenge is a small **64-bit ELF binary** that prompts the user for a serial.  
The objective is to reverse the validation logic and recover the correct key **without relying on brute force**.

The challenge was solved **purely through static analysis** using **Ghidra**, followed by a short Python script to reverse the validation logic.

---

## Detonation Analysis

Running the binary produces:

```

Enter your serial:

````

Any input results in rejection, indicating that the validation logic is fully implemented internally.

Static disassembly in Ghidra reveals a function named `check_serial`, which performs the serial validation.

---

## Encoded Serial

Inside `check_serial`, the binary defines an array of **8 integers**:

```c
local_38[0] = 0x58;
local_38[1] = 0x6e;
local_38[2] = 0x60;
local_38[3] = 0x6b;
local_38[4] = 0x7b;
local_38[5] = 0x56;
local_38[6] = 0x66;
local_38[7] = 0x75;
````

Observations:

* The expected serial length is **exactly 8 bytes**
* Each byte of user input is transformed and compared against this array

---

## Transform Function

Each character of the serial is passed through the following transformation:

```c
uint transform(char a, int b)
{
  return ((int)a ^ b + 7U) + 0xd & 0x7f;
}
```

### Breakdown

For each input character:

1. Take the character `a`
2. XOR it with its index `b`
3. Add `7`
4. Add `0xD` (13)
5. Mask the result with `0x7F`

Validation occurs inside a loop:

```c
x = transform((int)input[num], num);
if (x != local_38[num]) {
    return 0;
}
```

If any transformed byte does not match the expected value, the serial is rejected.

---

## Solving It

Instead of brute-forcing each character, the math can be **reversed directly**.

Approach:

1. Hardcode the expected transformed values
2. Reverse the transform equation
3. Recover the original input characters

This yields the correct serial instantly.

---

## Result

```
Key: LiZTeETf
```

---

## Conclusion

* No dynamic analysis or debugging was required
* The validation logic is fully visible through static disassembly
* Reversing the transformation function directly yields the serial
* This challenge highlights how **simple arithmetic transforms** can appear complex but are trivial to reverse

---

## Code Dump

```python
def main():
    expected_values = [0x58, 0x6e, 0x60, 0x6b, 0x7b, 0x56, 0x66, 0x75]
    length = 8

    correct_serial = ""
    for b in range(length):
        a = ((expected_values[b] & 0x7f) - 0xd) ^ (b + 7)
        correct_serial += chr(a)

    print("Correct serial is:", correct_serial)

if __name__ == "__main__":
    main()
```
