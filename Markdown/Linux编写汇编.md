
## 安装环境
```shell
# sudo apt install nasm
```

## 程序实列
```asm
[SECTION .bss]
[SECTION .data]
_data db "Hello,world!",0x0a
_len equ $-_data
[SECTION .text]
global _start
_start:
	mov eax,4 ;Call4
   mov ebx,1 ;1:Stdout,2:Stdin
	mov ecx,_data ;Load string
	mov edx,_len ;Load len
	int 0x80 ;Linux system call
	mov ebx,0 ;Returnment
	mov eax,1 ;Call 1
	int 0x80 ;Linux system call
```

## 编译命令：
```shell
# nasm -f elf32 Hello.asm -o Hello.o
# ld Hello.o -o Hello
```
注意<font color ="red">elf32</font>是编译32位