## The_fake_binary_bypass

This was a binary analysis challenge. The challenge provided the binary file and asked the flag under de binary, the compiled host name of the binary, a flag related to a extension file, the OS name and version, also another flag related to the main() function name. 

#### Steps I followed to find relevant information and flags
First I wanted to know what kind of binary was the file. Using `file` command got the following output.

```
p3rplex@p3rpleX:~/HackMexCTF2020Write-up/Challenges/Binary_bypass$ file binary_analysis_bypass
binary_analysis_bypass: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=89047470b27fed12ce5e60b0b1c55e528c2e461f, not stripped

```
It is a not stripped 64-bit Executable Linkable Format file in little endian: LSB (least-significant byte).
 
So I proceeded to run the file by simply typing:
``` 
p3rplex@p3rpleX:~/HackMexCTF2020Write-up/Challenges/Binary_bypass$ ./binary_analysis_bypass 
ETSCTF_ffffffa0 
```

Gotcha! running the file prints our first flag, the flag under the binary and it was very easy to find.

Then, I wanted the raw strings of the file. Initially, I used`cat` command but got a lot of useless characters, so I used `strings` and saved the output in a txt file with the following command:
```
p3rplex@p3rpleX:~/HackMexCTF2020Write-up/Challenges/Binary_bypass$ strings binary_analysis_bypass > binary_analysis_bypass.txt
```
After a few minutes inspecting the txt file I founded the flag `ETSCTF_0b77fe27f4498bc158ca119a58043842.c` under a .c extention in line 4902. (I used gedit, default text editor of GNOME to read files)

Also, I founded the compiled hostname  flag in line 4790 :
diz b1n4ry fil3 w4s compiled on `ETSCTF_dizizahostname`. Cool, three flags! only two more,


Since I couldn't find the distro name and version in this file, I used `readelf` command which displays information about ELF format object files according to the flags. (Used `man readelf` to have a better grasp of the command).

```
p3rplex@p3rpleX:~/HackMexCTF2020Write-up/Challenges/Binary_bypass$ readelf -a binary_analysis_bypass > readelfbinary.txt
```

```
  Magic:   7f 45 4c 46 02 01 01 03 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - GNU
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x401a30
  Start of program headers:          64 (bytes into file)
  Start of section headers:          754128 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         8
  Size of section headers:           64 (bytes)
  Number of section headers:         29
  Section header string table index: 28
  ...
```
 Well it is a UNIX-GNU os. Since I didn't understand a lot of this log, I spent like 30 minutes reading this cool [blog](https://linux-audit.com/elf-binaries-on-linux-understanding-and-analysis/) which explains in detail the ELF files. Finally at the bottom of the log I found the `ABI` version in line  2031: `OS: Linux, ABI: 3.2.0`. Just one flag remaining!


In order to find the flag related to the `main()` function name I used [gdb](https://www.gnu.org/software/gdb/) debugger:

```
p3rplex@p3rpleX:~/HackMexCTF2020Write-up/Challenges/Binary_bypass$ gdb ./binary_analysis_bypass
```
```

Reading symbols from ./binary_analysis_bypass...
(No debugging symbols found in ./binary_analysis_bypass)
(gdb) info file
Symbols from "/home/p3rplex/Downloads/binary_analysis_bypass".
Local exec file:
	`/home/p3rplex/Downloads/binary_analysis_bypass', 
        file type elf64-x86-64.
	Entry point: 0x401a30
	0x0000000000400200 - 0x0000000000400220 is .note.ABI-tag
	0x0000000000400220 - 0x0000000000400244 is .note.gnu.build-id
	0x0000000000400248 - 0x0000000000400470 is .rela.plt
	0x0000000000401000 - 0x0000000000401017 is .init
	0x0000000000401018 - 0x00000000004010d0 is .plt
	0x00000000004010d0 - 0x000000000047b8d0 is .text
	0x000000000047b8d0 - 0x000000000047c357 is __libc_freeres_fn
	0x000000000047c358 - 0x000000000047c361 is .fini
	0x000000000047d000 - 0x00000000004963fc is .rodata
	0x0000000000496400 - 0x00000000004a06b8 is .eh_frame
	0x00000000004a06b8 - 0x00000000004a0764 is .gcc_except_table
	0x00000000004a20e0 - 0x00000000004a2100 is .tdata
	0x00000000004a2100 - 0x00000000004a2140 is .tbss
	0x00000000004a2100 - 0x00000000004a2110 is .init_array
	0x00000000004a2110 - 0x00000000004a2120 is .fini_array
	0x00000000004a2120 - 0x00000000004a4f14 is .data.rel.ro
	0x00000000004a4f18 - 0x00000000004a4ff8 is .got
	0x00000000004a5000 - 0x00000000004a50d0 is .got.plt
--Type <RET> for more, q to quit, c to continue without paging--c
	0x00000000004a50e0 - 0x00000000004a6bd0 is .data
	0x00000000004a6bd0 - 0x00000000004a6c18 is __libc_subfreeres
	0x00000000004a6c20 - 0x00000000004a72c8 is __libc_IO_vtables
	0x00000000004a72c8 - 0x00000000004a72d0 is __libc_atexit
	0x00000000004a72e0 - 0x00000000004a89f8 is .bss
	0x00000000004a89f8 - 0x00000000004a8a20 is __libc_freeres_ptrs
```

With `info file` I listed all the sections and their addresses, but I did not found a relevant memory directions nor function names (related to the main name). 

Then, it occurred to me to set a breakpoint when calling the main function and check that memory address. First, I reviewed the assembly code of main function by typing `disass main`.

```
(gdb) disass main 

Dump of assembler code for function main:
   0x0000000000401b9e <+0>:	push   %rbp
   0x0000000000401b9f <+1>:	mov    %rsp,%rbp
   0x0000000000401ba2 <+4>:	mov    $0x0,%eax
   0x0000000000401ba7 <+9>:	callq  0x401b4d <x375fc0ff33>
   0x0000000000401bac <+14>:	mov    $0x0,%eax
   0x0000000000401bb1 <+19>:	pop    %rbp
   0x0000000000401bb2 <+20>:	retq   
```
I used `b main` command, so gdb created a break point at the memory address `0x401ba2`. reviewing the above-disassembled code founded out the instruction on that address. 

The instruction on this address is    `0x0000000000401ba2 <+4>:	mov    $0x0,%eax`, where using eax register to store the value momentarily.

 
```
(gdb) b main #breakpoint in main function 
Breakpoint 1 at 0x401ba2

(gdb) run

Starting program: /home/p3rplex/Downloads/binary_analysis_bypass 

Breakpoint 1, 0x0000000000401ba2 in main ()

```
Using step command, the step command only stops at the first instruction of a source line
```
(gdb) step
Single stepping until exit from function main,
which has no line number information.
ETSCTF_ffffffa0
0x00000000004021f1 in __libc_start_main ()

```

The memory address called is `0x00000000004021f1` for `__libc_start_main()` function. Just typed the founded flags in HackMex dashboard and earned some points.


That was it for this challenge. Really cool reading binaries. =)
