# easy02

``` C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __int64 v4; // [rsp+0h] [rbp-20h]
  __int64 v5; // [rsp+8h] [rbp-18h]
  char v6; // [rsp+10h] [rbp-10h]
  unsigned __int64 v7; // [rsp+18h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  v4 = 7076340818149207142LL;
  v5 = 9016600544715699305LL;
  v6 = 0;
  puts("it's 64bit program!");
  printf("here is flag : %s\n", &v4, 7076340818149207142LL, 9016600544715699305LL, *(_QWORD *)&v6, v7);
  return 0;
}
```

``` C
 printf("here is flag : %s\n", &v4, 7076340818149207142LL, 9016600544715699305LL, *(_QWORD *)&v6, v7);
 ```
 위의 코드를 통해  &v4에 flag가 들어가 있음을 알 수 있다.
 v4와 v5를 문자열로 출력해보면 각각 아래와 같다.
 ```
  v4 = 'b46{galf';
  v5 = '}!fle_ti';
  ```
  즉, flag는 **flag{64bit_elf!}** 임을 알 수 있다.
