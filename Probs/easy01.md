# easy01
``` C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [esp+Bh] [ebp-1Dh]
  int v5; // [esp+Fh] [ebp-19h]
  int v6; // [esp+13h] [ebp-15h]
  int v7; // [esp+17h] [ebp-11h]
  char v8; // [esp+1Bh] [ebp-Dh]
  unsigned int v9; // [esp+1Ch] [ebp-Ch]

  v9 = __readgsdword(0x14u);
  v4 = 0x67616C66;
  v5 = 0x6232337B;
  v6 = 0x655F7469;
  v7 = 0x7D21666C;
  v8 = 0;
  puts("it's 32bit program!");
  printf("here is flag : %s\n", &v4);
  return 0;
}
```
변수 v4, v5, v6, v7를 16진수가 아닌 Char 형으로 바꿔서 출력을 해보면 아래와 같다.

```
  v4 = 'galf';
  v5 = 'b23{';
  v6 = 'e_ti';
  v7 = '}!fl';
```
즉, flag가 **flag{32bit_elf!}** 임을 알 수 있다.
