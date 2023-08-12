---
title: "C with Assembly"
date: "2023-08-12"
tags: ["assembly", "c", "gcc"]
ShowToc: true
---

Short one,<br>
incase you want to write assembly functions and use these in C, 
maybe to speed up some things or just circumvent AV's by not calling hooked functions<br>
<br>

first you should at least somewhat understand assembly (you can find resources for learning here: <TODO>)<br>
there are multiple ways you can write assembly in c.

## Lets start with "Inline Assembly"
This is probably the most usefull for smaller things, altho gcc's inline assembly is very complicated and hard to read (in my opinion),<br>
its still somewhat usefull, lets use this example: I just want to get the PEB (Process Environment Block) (on an x64 system)<br>
I can just do this: 
``` c
int main() {
    uintptr_t peb_addr;
    __asm__ (
        "mov %%gs:(0x60), %0"
        : "=r" (peb)
    );

    PEB* peb = (PEB*)peb_addr;
}
```
in this example I use inline assembly to get the address of the PEB (stored at the offset `0x60` from `gs`)<br>
and then cast it to a pointer to the PEB structure (which can be found in `winternl.h`)<br>

this way of writing assembly is convinient for small things like the above example, but its very annoying to use for entire functions, <br>
So lets move on to

## "External" Assembly
Here we write our assembly code in a seperate file (I like to use the `.s` or `.asm` extension)<br>
lets make a function to add two numbers together (again x64 assembly), I will also use intel syntax<br>
because I prefer it over AT&T syntax<br>

asm.s 
``` S
.intel_syntax noprefix      ;;# tell gcc to use intel syntax
.global add                 ;;# make label add accessible to the linker

add:
    push    rbp             ;;# function prolouge
    mov     rbp, rsp
    sub     rsp, 0x30

    mov     eax, ecx        ;;# ecx is the first 
    add     eax, edx        ;;# add esi to eax (value of edi) and store result in eax

    add     rsp, 0x30       ;;# function epilouge
    pop     rbp
    ret
```
main.c
``` c
#include <stdio.h>

extern __fastcall int add(int, int);
int main(void) {
    int a = 0xc;
    int b = 0xd;

    int c = add(a, b);
    printf("%d + %d = %d\n", a, b, c);

    return 0;
}
```
we define the function as __fastcall, altho it is not needed since that is the default calling convention on Windows 10 (x64)<br>
<br>

Ok, so now comes compilation, with inline assembly it was as simple as just compiling your c file like normal.<br>
Now that we have a seperate file containing our Assembly code we need to somehow compile this file aswell... which actually is really simple<br>
``` sh
$ gcc -masm=intel ./asm.s ./main.c -o main
$ ./main
11 + 12 = 23
$
```
you may notice that I added `-masm=intel` which is not needed, but I do it anways<br>
