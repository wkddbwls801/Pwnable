# easy03

```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  puts("this is C!");
  puts("flag is in data section!");
  return 0;
}
```
data section에 flag가 존재한다고 한다. IDA를 통해 data section을 확인해 보면 아래와 같은 flag가 존재한다.

```
.data:0000000000601040 flag            db 'flag{this_is_C!}',0
```
즉, flag는 **flag{this_is_C!}** 이다.
