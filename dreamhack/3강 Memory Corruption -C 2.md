# FSB
* **FSB** : printf나 sprintf와 같이 포맷 스트링을 사용하는 함수에서 발생하는 취약점
       "%x"나 "%s"와 같이 프로그래머가 지정한 문자열이 아닌 사용자의 입력이 포맷 스트링으로 전달될 때 발생하는 취약점
* 예시
``` C
// fsb-1.c
#include <stdio.h>

int main(void) {
  char buf[100] = {0, );
  read(0, buf, 100);
  printf(buf);
}
```
```
$ ./fsb-1

$%x %d

bf3977c0 1001

$
```

# Double Free & Use-After-Free
* **Double Free** : 이미 해제된 메모리를 다시 한번 해제하는 취약점
* **UAF(Use-After-Free)** : 해제된 메모리에 접근해서 값을 쓸 수 있는 취약점
* 예제
``` C
// uaf1.c
#include <stdio.h>
#include <string.h>
#include <malloc.h>
int main(void) {
    char *a = (char *)malloc(100);
    memset(a, 0, 100);
    
    strcpy(a, "Hello World!");
    printf("%s\n", a);
    free(a);
    
    char *b = (char *)malloc(100); 
    strcpy(b, "Hello Pwnable!");
    printf("%s\n", b);
    
    strcpy(a, "Hello World!");
    printf("%s\n", b);
}
```
```
# gcc -o uaf1 uaf1.c
# ./mem
Hello World!
Hello Pwnable!
Hello World!
# 
```
100 바이트 크기의 메모리 a를 할당 후 "Hello World!" 문자열을 복사한다.   
![image](https://user-images.githubusercontent.com/59531805/79181891-6b3b1280-7e48-11ea-9c92-6e602c35ef34.png)   
그 다음 a를 해제하고 새로운 100 바이트 크기의 메모리 b를 할당한다. 새로 할당된 메모리에는 strcpy 함수를 통해 메모리 b에 
"Hello Pwnable!" 문자열을 복사한다.   
![image](https://user-images.githubusercontent.com/59531805/79182046-cbca4f80-7e48-11ea-9cd8-49fae3e9bc2b.png)   
포인터 a에 저장된 메모리 주소 값은 바뀌지 않고 메모리 a와 메모리 b가 같은 주소를 가리키고 있다.
이미 해제되었던 메모리 a가 메모리 할당자로 들어가고, 새로운 메모리 영역을 할당할 때 메모리를 효율적으로 관리하기 위해 기존에 해제되었던 메모리가 그대로 반환되어 일어나는 일이다.
strcpy(a, "Hello World!")에서 Hello World를 복사하는 포인터는 b가 아니라 해제된 메모리 포인터 a이다.   
![image](https://user-images.githubusercontent.com/59531805/79182168-206dca80-7e49-11ea-806d-21fb28410da0.png)   
이미 해제된 포인터 a와 새로이 할당한 포인터 b가 같은 메모리 영역을 가리키고 있기 때문에 포인터 a에 Hello World! 문자열을 복사하고 포인터 v의 내용을 출력하면
Hello World! 문자열이 출력된다.

# Integer issues
* **정수의 범위**   
![image](https://user-images.githubusercontent.com/59531805/79182338-82c6cb00-7e49-11ea-9719-01bb92ce0681.png)
