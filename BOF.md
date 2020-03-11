# Buffer Overflow란?

**Buffer Overflow**는 C언어나 C++에서 버퍼에 데이터를 입력 받을 때 **입력 값의 크기를 검증하지 않아** 
버퍼가 흘러 넘쳐 다른 변수나 메모리를 덮어 씌우게 되는 버그이다.   
이 취약점을 이용해 **Return address**를 원하는 주소로 덮어 씌워 **IP(Instruction Pointer)를 제어**할 수 있게 된다.   
Buffer Overflow에는 **Stack Overflow**와 **Heap Overflow**가 있다.

## 실습
``` C
# include <stdio.h>
# inslucd <string.h>

int main(int argc, char *argv[])
{
    char buf[256];
    
    strcpy(buf, argv[1]);
    printf("%s\n", buf);
}
```
**strcpy 함수로 argv[1]을 buf로 복사하고 printf로 출력한다.   
여기서 취약점은 **strcpy** 함수에서 발생한다.   
strcpy 함수는 null byte(문자열의 끝)를 만날 때까지 문자열을 복사하는 함수이다. 즉, 복사하는 버퍼의 크기를 검증하지 않는 취약한 함수이다.   
그래서 buf의 크기인 256byte를 훨씬 넘는 데이터도 buf에 복사할 수 있게 된다.

```
oojin@ubuntu:~/Study/System/BOF$ gcc -m32 -fno-stack-protector -mpreferred-stack-boundary=2 -z execstack -no-pie -o bugfile1 bugfile1.c
yoojin@ubuntu:~/Study/System/BOF$ ls
bugfile1  bugfile1.c
yoojin@ubuntu:~/Study/System/BOF$ ./bugfile1 AAAA
AAAA
```
32bit로 컴파일 시켰다. AAAA를 입력하면 AAAA가 출력되는 그런 형식이다.

```
yoojin@ubuntu:~/Study/System/BOF$ gdb -q bugfile1
Reading symbols from bugfile1...(no debugging symbols found)...done.
(gdb) disas main
Dump of assembler code for function main:
   0x0804843b <+0>:	push   ebp
   0x0804843c <+1>:	mov    ebp,esp
   0x0804843e <+3>:	sub    esp,0x100 //0x100은 10진수로 256
   0x08048444 <+9>:	mov    eax,DWORD PTR [ebp+0xc]
   0x08048447 <+12>:	add    eax,0x4
   0x0804844a <+15>:	mov    eax,DWORD PTR [eax]
   0x0804844c <+17>:	push   eax
   0x0804844d <+18>:	lea    eax,[ebp-0x100]
   0x08048453 <+24>:	push   eax
   0x08048454 <+25>:	call   0x8048300 <strcpy@plt>
   0x08048459 <+30>:	add    esp,0x8
   0x0804845c <+33>:	lea    eax,[ebp-0x100]
   0x08048462 <+39>:	push   eax
   0x08048463 <+40>:	call   0x8048310 <puts@plt>
   0x08048468 <+45>:	add    esp,0x4
   0x0804846b <+48>:	mov    eax,0x0
   0x08048470 <+53>:	leave  
   0x08048471 <+54>:	ret    
End of assembler dump.
```
gdb로 bugfile1을 분석해 보았다. main함수를 disassemble하면 위와 같다.
main+3에서 stack을 0x100만큼 확장하였다.(변수 할당)

<img src="C:\Users\wkddb\Desktop\stack.jpg">

