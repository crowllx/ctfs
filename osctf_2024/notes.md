### leaky_pipes
first argument to fgets in func readflag
[sp + 0x0] = 0xffffc8a0 → 0xffffc908 → 0x00000000,

observed that the scanf used to get user input has a parameter
"%127s" and that's it, I was able to leak values of the stack using %x

I believe the program reads the flag into memory first and the exploit is to
abuse scanf in order to leak the flag value, if I understand right the memory
address should be the first argument to fgets listed above, I just need to
construct a payload the right size to where a % operand will leak that
address

`$esp   : 0xffffc7e0  →  0xffffc8a0  →  "WINNING\n"`

was a simple format string vuln where are input was being printed back to us
and we could use `%{n}$s` to print string values from stack.
fuzzed with incrementing values for n to leak the flag value that had previously
been read into memory.

#### shellmischief

call to gets taking user input in `vuln` function.

```
0x8048989 -> comparison happening seemed important
0xffffc720 -> code execution
0xffffc6b8 -> payload
0xfff9d388
0xfff9d318
```

program seems to take our payload and store it somewhere and then execute a
'random' section of code, I think there is a small range to this, not sure what
it is but placing a nop sled of 120 bytes at the start of my payload I was
able to get it to consistently execute my code.


OSCTF{u_r_b3rry_mischievous_xD}
