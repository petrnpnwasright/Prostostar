# Stack 0
## Tentang
Level ini memperkenalkan konsep bahwa memori dapat diakses di luar wilayah yang dialokasikan, bagaimana variabel stack ditata, dan bahwa memodifikasi di luar memori yang dialokasikan dapat mengubah eksekusi program.

## Source Code
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}
```

## Target Exploits
Pada level ini kita ditargetkan untuk mengubah variabel **modified** menjadi tidak sama dengan 0. Pada program diatas, input kita disimpan pada kedalam buffer menggunakan fungsi gets(). Karena gets() bukanlah fungsi yang aman, kita dapat menggunakannya untuk menimpa memori dimana variabel **modified** disimpan.

## Kerentanan Program
Seperti yang dikatakan sebelumnya **gets()** adalah fungsi yang tidak aman. Jika kita melihat pada man pages gets() dengan mengetikkan *man gets* pada bagian BUGS disebutkan bahwa gets() rentan karena tidak mungkin untuk mengetahui berapa banyak karakter yang akan diterima oleh gets(). fungsi ini akan terus menyimpan karakter setelah akhir buffer.

## Lets Dive Deeper
**[!] Disclaimer** saya disini memindahkannya dari Protostar machine kedalam local maschine saya, agar lebih mudah untuk menganalisis programnya. Disini saya juga menggunakan ekstensi dari GDB yaitu gef.

Pertama tama kita coba liat bagaimana kodenya terlihat di assembly menggunakan command berikut
disass main
```
   0x080483f4 <+0>:	push   ebp
   0x080483f5 <+1>:	mov    ebp,esp
   0x080483f7 <+3>:	and    esp,0xfffffff0
   0x080483fa <+6>:	sub    esp,0x60
   0x080483fd <+9>:	mov    DWORD PTR [esp+0x5c],0x0
   0x08048405 <+17>:	lea    eax,[esp+0x1c]
   0x08048409 <+21>:	mov    DWORD PTR [esp],eax
   0x0804840c <+24>:	call   0x804830c <gets@plt>
   0x08048411 <+29>:	mov    eax,DWORD PTR [esp+0x5c]
   0x08048415 <+33>:	test   eax,eax
   0x08048417 <+35>:	je     0x8048427 <main+51>
   0x08048419 <+37>:	mov    DWORD PTR [esp],0x8048500
   0x08048420 <+44>:	call   0x804832c <puts@plt>
   0x08048425 <+49>:	jmp    0x8048433 <main+63>
   0x08048427 <+51>:	mov    DWORD PTR [esp],0x8048529
   0x0804842e <+58>:	call   0x804832c <puts@plt>
   0x08048433 <+63>:	leave
   0x08048434 <+64>:	ret

```
