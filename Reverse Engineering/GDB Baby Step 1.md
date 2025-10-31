# GDB Baby Step 1
This challenge is an executable file and our task is to do Reverse Engineering on it using GDB and find out the value contained in the EAX register at the end of the main function.

## Solution:

1. First, we download the challenge file debugger0_a.
2. We use the file command to learn more about the file. This tells us it's a 64-bit ELF executable, which means it runs on Linux.
```
$ file debugger0_a
debugger0_a: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=..., not stripped
```
3. Next, we open the program using GDB.
```
$ gdb debugger0_a
```
4. Once inside GDB, we can list the functions in the program to find our target, main.
```
(gdb) info functions
```
5. Before we look at the assembly, it's helpful to switch GDB's disassembly syntax from the default (AT&T) to Intel, which is often easier to read.
```
(gdb) set disassembly-flavor intel
```
6. Now, we disassemble the main function to see its assembly instructions.
```
(gdb) disassemble main
```
7. By looking at the output of disassemble main, we can see the instructions. Our goal is to find the value in the EAX register at the end. Near the end of the function, we see this instruction: mov eax, 0x86342

8. This instruction explicitly moves the hexadecimal value 0x86342 into the eax register.

9. The challenge wants the decimal value. We can use GDB's print command to convert this for us.
```
(gdb) print 0x86342
$1 = 549698
```
10. The result is 549698. This will be our flag.


## Flag:

```
picoCTF{549698}
```

## Concepts learnt:

1. GDB (GNU Debugger): A powerful tool used to debug programs. In reverse engineering, it's used to inspect the program's state, view memory, and step through assembly instructions.

2. ELF (Executable and Linkable Format): The standard file format for executables, object code, and shared libraries on Linux.

3. Assembly Disassembly: The process of converting machine code (binary) back into human-readable assembly language.

4. GDB Commands:

file <filename>: (A bash command) Used to determine the file type.

gdb <filename>: (A bash command) Starts GDB and loads the executable.

info functions: A GDB command to list all functions within the loaded program.

set disassembly-flavor intel: A GDB command to change the syntax of the assembly output.

disassemble <function_name>: A GDB command that shows the assembly instructions for a specific function.

print <value>: A GDB command that evaluates an expression or value. It's very useful for converting between bases (e.g., hex to decimal).

## Notes:

1. The challenge was straightforward because the value was "hardcoded" or placed explicitly into the eax register.

2. Remembering to set the disassembly flavor to intel is a common first step, as many reverse engineers find it more intuitive than the default AT&T syntax.

## Resources:

1. https://www.youtube.com/watch?v=gh2RXE9BIN8

2. https://medium.com/@Tvrpism/getting-started-with-pwndbg-without-getting-lost-a825245790af

3. https://hugsy.github.io/gef/

4. https://youtu.be/-iRG9_zFRC4?si=7yMszSrqad4S0P1i

5. https://youtube.com/playlist?list=PLMB3ddm5Yvh3gf_iev78YP5EPzkA3nPdL

