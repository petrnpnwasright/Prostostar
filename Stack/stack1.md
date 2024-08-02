# Stack 1
## Tentang
Level ini memperkenalkan konsep memodifikasi variabel ke nilai tertentu dalam program, dan bagaimana variabel diletakkan dalam memori.

## Source Code
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
```

## Target Exploits
Pada level ini kita ditargetkan untuk mengubah variabel **modified** menjadi 0x61626364 untuk mencetak string "you have correctly got the variable to the right value". Pada program diatas, input kita disimpan pada kedalam buffer menggunakan fungsi **strcpy()**.

## Kerentanan Program
**strcpy()** bukanlah fungsi yang berbahaya seperti gets(), namun pada program diatas, ia tidak menentukan seberapa banyak buffer bisa disimpan di dalam alamat memory sehingga kita dapat menimpa alamat alamat seterusnya. Ini yang akan kita manfaatkan untuk menimpa nilai dari modified variabel.


## Lets Dive Deeper
**[!] Disclaimer** saya disini memindahkannya dari Protostar machine kedalam local maschine saya, agar lebih mudah untuk menganalisis programnya. Disini saya juga menggunakan ekstensi dari GDB yaitu gef. Jadi alamat pada machine anda mungkin berbeda.

> **gdb -q stack0** </br>
> **disass main**
```
   0x08048464 <+0>:	push   ebp
   0x08048465 <+1>:	mov    ebp,esp
   0x08048467 <+3>:	and    esp,0xfffffff0
   0x0804846a <+6>:	sub    esp,0x60
   0x0804846d <+9>:	cmp    DWORD PTR [ebp+0x8],0x1
   0x08048471 <+13>:	jne    0x8048487 <main+35>
   0x08048473 <+15>:	mov    DWORD PTR [esp+0x4],0x80485a0
   0x0804847b <+23>:	mov    DWORD PTR [esp],0x1
   0x08048482 <+30>:	call   0x8048388 <errx@plt>
   0x08048487 <+35>:	mov    DWORD PTR [esp+0x5c],0x0
   0x0804848f <+43>:	mov    eax,DWORD PTR [ebp+0xc]
   0x08048492 <+46>:	add    eax,0x4
   0x08048495 <+49>:	mov    eax,DWORD PTR [eax]
   0x08048497 <+51>:	mov    DWORD PTR [esp+0x4],eax
   0x0804849b <+55>:	lea    eax,[esp+0x1c]
   0x0804849f <+59>:	mov    DWORD PTR [esp],eax
   0x080484a2 <+62>:	call   0x8048368 <strcpy@plt>
   0x080484a7 <+67>:	mov    eax,DWORD PTR [esp+0x5c]
   0x080484ab <+71>:	cmp    eax,0x61626364
   0x080484b0 <+76>:	jne    0x80484c0 <main+92>
   0x080484b2 <+78>:	mov    DWORD PTR [esp],0x80485bc
   0x080484b9 <+85>:	call   0x8048398 <puts@plt>
   0x080484be <+90>:	jmp    0x80484d5 <main+113>
   0x080484c0 <+92>:	mov    edx,DWORD PTR [esp+0x5c]
   0x080484c4 <+96>:	mov    eax,0x80485f3
   0x080484c9 <+101>:	mov    DWORD PTR [esp+0x4],edx
   0x080484cd <+105>:	mov    DWORD PTR [esp],eax
   0x080484d0 <+108>:	call   0x8048378 <printf@plt>
   0x080484d5 <+113>:	leave
   0x080484d6 <+114>:	ret
```
