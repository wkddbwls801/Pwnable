# easy04
``` C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __int64 v3; // rax
  __int64 v4; // rdx
  __int64 v5; // rax

  v3 = std::operator<<<std::char_traits<char>>(&_TMC_END__, "this is C++!", envp);
  std::ostream::operator<<(v3, &std::endl<char,std::char_traits<char>>);
  v5 = std::operator<<<std::char_traits<char>>(&_TMC_END__, "flag is in data section!", v4);
  std::ostream::operator<<(v5, &std::endl<char,std::char_traits<char>>);
  return 0;
}
```
IDA로 분석을 해 보면 "flag is in data section!"이라고 나와 있다.

```
.data:0000000000601060 flag            db 'flag{this_is_Cplusplus!}',0
```
flag를 찾아보면 flag{this_is_Cplusplus!}임을 알 수 있다.
