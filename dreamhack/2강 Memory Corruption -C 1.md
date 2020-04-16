# BOF
* **버퍼** : 지정된 크기의 메모리 공간
* **BOF** :  버퍼가 허용할 수 있는 양의 데이터보다 더 많은 값이 저장되어 버퍼가 넘치는 취약점   
![image](https://user-images.githubusercontent.com/59531805/78787353-30b71d00-79e5-11ea-9b85-1bdc81105272.png)   
위와 같은 상황을 BOF가 발생했다고 하고, 이는 프로그램의 Undefined Behavior을 이끌어낸다.
만약 데이터 영역 B에 나중에 호출될 함수 포인터를 저장하고 있다면 이 값을 "AAAAAAAA"와 같은 데이터로 덮었을 때 Segmentation Fault를 발생시킬 것이다. 만약 공격자가 이를 악용한다면 어딘가에 기계어 코드를 삽입한 후 함수 포인터를 공격자의 코드의 주소로 덮어 코드를 실행할 수도 있다.
* 예시
``` C
// stack-1.c
#include <stdio.h>
#include <stdlib.h>
int main(void) {
    char buf[16];
    gets(buf);
    
    printf("%s", buf);
}
```
gets 함수는 사용자가 개행을 입력하기 전까지 입력했던 모든 내용을 첫 번째 인자로 전달된 버퍼에 저장하는 함수이다. 그러나 gets 함수에는 별도의 길이 제한이 없기 때문에 16바이트가 넘는 데이터를 입력한다면 stack BOF가 발생한다.

# OOB
* **OOB(Out Of Boudary)** : 버퍼의 길이 범위를 벗어나는 인덱스에 접근할 때 발생하는 취약점
* 예시
``` C
// oob-1.c
#include <stdio.h>
int main(void) {
    int win;
    int idx;
    int buf[10];
    
    printf("Which index? ");
    scanf("%d", &idx);
    printf("Value: ");
    scanf("%d", &buf[idx]);
    printf("idx: %d, value: %d\n", idx, buf[idx]);
    if(win == 31337){
        printf("Theori{-----------redeacted---------}");
    }
}
```
buf의 길이는 10이므로 buf의 인덱스로 사용될 수 있는 올바른 값은 0이상 10미만의 정수이다. 그러나 첫번째 scanf를 통해 입력받은 idx 값을 올바른 범위에 속해 있는지 검사하지 않는다.

# Off-by-one
* **Off-by-one** : 경계 검사에서 하나의 오차가 있을 때 발생하는 취약점이다. 버퍼의 경계 계산 혹은 반복문의 횟수 계산 시 < 대신 <=을 쓰거나, 0부터 시작하는 인덱스를 고려하지 못할 때 발생한다.
* 예시
``` C
// off-by-one-1.c
#include <stdio.h>
void copy_buf(char *buf, int sz) {
    char temp[16];
    
    for(i = 0; i <= sz; i++)
        temp[i] = buf[i];
}
int main(void) {
    char buf[16];
    
    read(0, buf, 16);
    copy_buf(buf, sizeof(buf));
}
```
copy_buf 함수에서 임시 버퍼 temp를 할당하고 반복문을 통해 buf의 데이터를 복사한다. 그러나 반복문은 i가 0일 때부터 sz일 떄까지 총 sz+1번 반복하게 된다. 따라서 sz+1만큼 데이터가 복사되고, off-by-one 취약점이 발생한다.
