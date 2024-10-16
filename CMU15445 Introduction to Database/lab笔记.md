
# pro 0: C++ Primer



- First, the string is hashed to produce a hash value, which is then converted into its binary representation. 
- From the hash value (binary form), `b` bits are extracted, <u>starting from</u> the **most significant bit(MSB)**. 
- The register value is calculated from the extracted bits. (by default each register has a value of `0`).
- From the remaining set of bits, the position of the leftmost 1 is obtained (MSB), i.e the number of leading zeros from left plus 1 (as given in the picture given below).
- After this, using the `b` bits, the register value is calculated (which in the above case it’s 6). Hence, in register 6, `max(register[6], p)` will be stored.

左边取b位求值作为下标，然后剩下的数左边第一个1在第几位（起始0的个数+1）作为值。

You can use `BUSTUB_ASSERT` for assertions in debug mode.

`bitset[i]`的下标越小，输出的是越低的位数！

`AddElem`需要可并发化。TODO：让读也上mutex不然就是UB。

TODO：虚拟机内硬盘分配





