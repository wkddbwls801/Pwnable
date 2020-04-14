# Logical Bugs
*Logical Bugs : 프로그램의 메모리 관리 실수로 인해 발생하는 메모리 커럽션 취약점과 달리 프로그램의 논리적 오류로 인해 발생
*예시
``` C
// gcc -o logical logical.c 
#include <stdio.h>
int withdraw(int balance, int money) {
    return balance - money;
}
int main() {
    int balance = 10000;
    int amount = 0;
    int result = 0;
    printf("Your balance: %d\n\n", balance);
    printf("Please enter the amount to withdraw: ");
    scanf("%d", &amount);
    result = withdraw(balance, amount);
    printf("Check your balance: %d", result);
    return 0;
}
```
withdraw함수에서는 잔고에서 출금 금액을 뺀 값이 반환되는데, 음수에 대한 예외처리가 없기 때문에 이는 잘못된 결과를 낼 수 있다.

# Command Injection
*인젝션 : 검증되지 않은 공격자의 입력을 셸 커맨드 또는 쿼리 일부로 처리해 정상적인 실행 흐름을 변결할 수 있는 취약점
*커맨드 인젝션 : 프로그램이 적절한 검증 없이 사용자의 입력을 셸 명령어로 실행할 때 발생하는 취약점

# Race Condition
*레이스 컨디션 : 프로세스 혹은 스레드 간 자원 관리 실수로 인해 발생하는 상태
*예제
```C
// gcc -o race2 race2.c -fno-stack-protector -lpthread -m32
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
int len;
void giveshell() {
    system("/bin/sh");
}
void * t_function() {
    int i = 0;
    while (i < 10000000) {
        len++;
        i++;
        sleep(1);
    }
}
int main() {
    char buf[4];
    int gogo;
    int idx;
    pthread_t p_thread;
    setvbuf(stdin, 0, 2, 0);
    setvbuf(stdout, 0, 2, 0);
    while (1) {
        printf("1. thread create\n");
        printf("2. read buffer\n");
        printf("> ");
        scanf("%d", &idx);
        switch (idx) {
        case 1:
            pthread_create( &p_thread, NULL, t_function, (void * ) NULL);
            break;
        case 2:
            printf("len: ");
            scanf("%d", &len);
            if(len > sizeof(buf)) {
                exit(0);
            }
            sleep(4);
            printf("Data: ");
            read(0, buf, len);
            printf("Len: %d\n", len);
            printf("buf: %s\n", buf);
            break;
        case 3:
            if (gogo == 0x41414141) {
                giveshell();
            }
        }
    }
    return 0;
}
```
1번 메뉴는 t_function함수를 스레드로 실행한다. t_function 함수는 len 전역 변수를 i가 10000000 값에 도달할 때 까지 1씩 증가 시킨다. 2번 메뉴는 len 전역 변수에 입력을 받고 buf배열의 크기보다 크다면 프로그램을 종료하여 스택 버퍼 오버플로우가 발생하지 않도록 하는 조건이 존재한다. 그리고 4초 후 buf에 입력을 받는다. 3번 메뉴는 gogo변수가 0x41414141이라면 giveshell함수를 호출한다.
len변수에 뮤텍스가 걸려있지 않기 때문에 스레드를 생성해 t_function함수를 실행하면 여러 스레드가 len변수를 동시에 참조할 수 있게 된다.
메뉴 2번에서 len 변수는 4보다 클 수 없다. 4를 입력하고 sleep이 호출되어 대기 중일 때 다른 스레드에서 t_function함수가 실행되고 있다면 len 값은 증가하게 된다. 그러므로 read함수가 호출될 때의 len값은 버퍼의 크기보다 큰 값이 된다. 따라서 버퍼 오버플로우가 발생해 buf뒤에 있는 gogo변수를 원하는 값으로 덮을 수 있다.
```
$ ./race2
1. thread create
2. read buffer
> 1
1. thread create
2. read buffer
> 2
len: 4
Data: BBBBAAAA
Len: 11
buf: BBBBAAAA
1. thread create
2. read buffer
> 3
$ id
uid=1001(theori) gid=1001(theori) groups=1001(theori)
```

# Path Travlersal
*Path Traversal : 프로그래머가 가정한 디렉토리를 벗어나 외부에 존재하는 파일에 접근할 수 있는 취약점

# Environment attack
*환경 변수 : 프로세스가 동작하는 방식에 영향을 미칠 수 있는 동적인 값들의 모임
PATH라는 환경 변수에 경로를 넣어두면 해당 경로에 있는 파일을 현재 디렉토리에 있는 파일과 같이 실행할 수 있다.
LD_PRELOAD 환경 변수는 포더가 프로세스에 로드 할 라이브러리 파일을 지정할 수 있다. 프로그램에서 함수가 호출되면 제일 먼저 LD_PRELOAD 환경 변수에 등록된 라이브러리 파일을 참조하여 호출한다.
라이브러리를 새로 생성하여 프로그램에서 호출하는 함수의 이름과 동일한 함수를 만들고 LD_PRELOAD 환경 변수에 해당 라이브버리 파일을 전달하면 원하는 코드를 실행할 수 있다.



