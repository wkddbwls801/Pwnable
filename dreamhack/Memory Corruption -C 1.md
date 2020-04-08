# BOF
* 버퍼 : 지정된 크기의 메모리 공간
* BOF :  버퍼가 허용할 수 있는 양의 데이터보다 더 많은 값이 저장되어 버퍼가 넘치는 취약점   
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
