#### the voice

0x7ffe07ca2ea8 => gaaahaaa

```
 ► 0x40130c <main+166>    call   __stack_chk_fail@plt        <__stack_chk_fail@plt>
        rdi: 0xa
        rsi: 0
        rdx: 0x63b045494769d367
        rcx: 0x7ffe07ca2e90 ◂— 'aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa'
```

```
0x4012e9 <main+131>    cdqe
   0x4012eb <main+133>    mov    qword ptr fs:[rax*8 - 0x50], 0x27cf
   0x4012f8 <main+146>    mov    eax, 0                                  EAX => 0
   0x4012fd <main+151>    mov    rdx, qword ptr [rbp - 8]
   0x401301 <main+155>    sub    rdx, qword ptr fs:[0x28]
```
writing to local thread storage at `fs:[rax*8 - 0x50]`.

sub comes write before je call before jumping away from stack check,
so if i can write 0x27cf to fs:[0x28] I should know the canary value.

fairly certain 0x7ffe07ca2ea8 is the where the canary is and is 'gaaahaaa' of my cyclic input.


`hex(15*8 - 0x50) = 0x28`

once passing stack check even when sending massive input, getting segfault at 
offset 12 of my inital payload.

UMDCTF{pwn_g3ss3r1t_sk1ll5_d0nt_tak3_a5_many_y3ar5_t0_l3arn_pau1}


#### the spice

creates an array structs of len 8

struct :
    int spice amount
    string name[20]
    
when adding a buyer we can specify the length of their name, and then write
to that buyers name field with characters up to that length. I believe this is the
buffer overflow exploit.

setting buyer at index 7 with a name length of 70 and i was able to cause stack smashing.

next step is to see if i can identify which part of my input caused this.
slightly confused since I thought the struct name I was writing into was on the heap,
but I think i can analyze the heap and stack now and see what is going on.


```
 ► 0x4017eb <main+1374>    mov    rdx, qword ptr [rbp - 8]      RDX, [0x7ffcf4ca4298] => 0x6166616161656161 ('aaeaaafa')
   0x4017ef <main+1378>    sub    rdx, qword ptr fs:[0x28]
   0x4017f8 <main+1387>    je     main+1394                   <main+1394>

   0x4017fa <main+1389>    call   __stack_chk_fail@plt        <__stack_chk_fail@plt>
```
0x7ffcf4ca4298 is where the canaray value lives on the stack,
while aaeaaafa was the portion of my cyclic input that was overwriting

the method to print out the name and spice of a buyer doesn't check info bounds
so I can use this is to leak the canary value at run time and use it as part of my payload.
we are also given a function to leak the pointer to the array of structs so my next steps are
to overwrite a return address with that pointer+some offset, probably 8, and place a nop sle
in buyer 0's name to verify the program will execute my code.

NX is enabled so this is false. This requires ROP chaining.
I managed to get a solution working locally but it did not work remote.
part of my solution needed rop gadgets from libc. From looking at others solutions
I think my error was assuming since there was no PIE I could just use the base address of libc
locally for a remote exploit as well.

making assumptions about the address of linked libraries because theres theres no PIE
was definately a big mistake. I also was working with a different version of libc and ld
so although my exploit worked locally it was never going to work remote.

the solution was to leak an address in libc or ld and calculate the base of either
in order to make use the rop gadgets available there. The gadgets in the program itself were lacking.

this solidifies my previous conclusion last fall that my skills in rop chaining are lacking.
Additionally from reading others thoughts on the challenge, using SROP was one way of solving this challenge.
I have not looked into this at all and I think exploring ROP/SROP should be my current goals.
