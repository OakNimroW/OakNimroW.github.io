---
title: BuffEMR
date: 2023-12-27 00:00:00 +/-TTTT 
categories: [Writeup]
tags: [writeup, crackme.one]     # TAG names should always be lowercase
---



## Read the assembly code

```bash
objdump -M intel -d ./main  > analyze.asm
```

## Analyze the code
The code has a subrutine that checks if the input is an palindrome

```markdown
./main:     file format elf64-x86-64

Disassembly of section .text:

0000000000401000 <.text>:
  401000:	48 be 32 20 40 00 00 	movabs rsi,0x402032           ; Storing base of string "Enter Password: "
  401007:	00 00 00 
  40100a:	bb 10 00 00 00       	mov    ebx,0x10               ; Store length of "Enter Password: "
  40100f:	e8 b0 00 00 00       	call   0x4010c4               ; Call subrutine to print "Enter Password: "
  401014:	b8 00 00 00 00       	mov    eax,0x0                ; Store value 0 in eax
  401019:	bf 00 00 00 00       	mov    edi,0x0                ; Store value 0 in edi
  40101e:	48 be 00 20 40 00 00 	movabs rsi,0x402000           ; Store base of buffer to store input
  401025:	00 00 00 
  401028:	ba 32 00 00 00       	mov    edx,0x32               ; Store value 50 in edx
  40102d:	0f 05                	syscall                       ; Syscall sys_read store input in rsi, with length of edx
  40102f:	e8 36 00 00 00       	call   0x40106a               ; Call subrutine to analyze if the input is an palindrome
  401034:	e8 68 00 00 00       	call   0x4010a1               ; Call subrutine to check if the input is longer than 3 characters
  401039:	48 be 42 20 40 00 00 	movabs rsi,0x402042           ; Store in rsi base of "Correct."
  401040:	00 00 00 
  401043:	bb 08 00 00 00       	mov    ebx,0x8                ; Store in ebx length of "Correct."
  401048:	eb 0f                	jmp    0x401059               ; Jump to 0x401059
  40104a:	48 be 4a 20 40 00 00 	movabs rsi,0x40204a           ; Store in rsi base of "Wrong."
  401051:	00 00 00 
  401054:	bb 06 00 00 00       	mov    ebx,0x6                ; Store in ebx length of "Wrong."
  401059:	e8 66 00 00 00       	call   0x4010c4               ; Subrutine sys_write prints "Correct."
  40105e:	b8 3c 00 00 00       	mov    eax,0x3c               ; Store value 60 in eax
  401063:	bf 00 00 00 00       	mov    edi,0x0                ; Store value 0 in edi
  401068:	0f 05                	syscall                       ; Syscall sys_exit with error code 0 (no errors) 
  40106a:	48 bf 00 20 40 00 00 	movabs rdi,0x402000           ; Store in rdi base of input
  401071:	00 00 00 
  401074:	80 3f 0a             	cmp    BYTE PTR [rdi],0xa     ; Compare character with "\n"
  401077:	74 05                	je     0x40107e               ; Jump if the character is "\n" 
  401079:	48 ff c7             	inc    rdi                    ; Continue to the next character
  40107c:	eb f6                	jmp    0x401074               ; Loop with the next character
  40107e:	48 ff cf             	dec    rdi                    ; Change the rdi direction from "\n" to the last character
  401081:	48 be 00 20 40 00 00 	movabs rsi,0x402000           ; Store in rsi the base of the String
  401088:	00 00 00 
  40108b:	48 39 f7             	cmp    rdi,rsi                ; Compare ends and starts of the input
  40108e:	7e 10                	jle    0x4010a0               ; Jump if result 0 or less, in case the input are zero or one characters and the "\n"
  401090:	8a 07                	mov    al,BYTE PTR [rdi]      ; Store in al the last character
  401092:	8a 26                	mov    ah,BYTE PTR [rsi]      ; Store in ah the first character
  401094:	38 e0                	cmp    al,ah                  ; Compare the characters
  401096:	75 b2                	jne    0x40104a               ; Jump if the characters are not equals
  401098:	48 ff cf             	dec    rdi                    ; Decrese rdi to use the previous character
  40109b:	48 ff c6             	inc    rsi                    ; Increse rsi to use the next character
  40109e:	eb eb                	jmp    0x40108b               ; Jump to loop the comparisions
  4010a0:	c3                   	ret
  4010a1:	48 bf 00 20 40 00 00 	movabs rdi,0x402000           ; Stores in rdi the beginning of the input
  4010a8:	00 00 00 
  4010ab:	bb 00 00 00 00       	mov    ebx,0x0                ; Store value 0 in ebx
  4010b0:	80 3f 0a             	cmp    BYTE PTR [rdi],0xa     ; Compare the first character with "\n"
  4010b3:	74 08                	je     0x4010bd               ; Jump if the first chaaracter is an "\n"
  4010b5:	48 ff c7             	inc    rdi                    ; Increase rdi to the direction of the next character
  4010b8:	48 ff c3             	inc    rbx                    ; Increase rbx
  4010bb:	eb f3                	jmp    0x4010b0               ; Loop with the next character
  4010bd:	48 83 fb 03          	cmp    rbx,0x3                ; Compare counter rbx with 3
  4010c1:	7c 87                	jl     0x40104a               ; Jump if the counter is less than 3
  4010c3:	c3                   	ret
  4010c4:	b8 01 00 00 00       	mov    eax,0x1                ; Store value 1 in eax
  4010c9:	bf 01 00 00 00       	mov    edi,0x1                ; Store value 1 in edi
  4010ce:	48 89 da             	mov    rdx,rbx                ; Store the value from rbx in rdx
  4010d1:	0f 05                	syscall                       ; Syscall sys_write prints in stdout the string that starts in rdx
  4010d3:	c3                   	ret
```

