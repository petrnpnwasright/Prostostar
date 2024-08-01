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
