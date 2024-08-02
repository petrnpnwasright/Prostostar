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
Pada level ini kita ditargetkan untuk mengubah variabel **modified** menjadi tidak sama dengan 0 untuk mencetak string "you have changed the 'modified' variable". Pada program diatas, input kita disimpan pada kedalam buffer menggunakan fungsi gets(). Karena gets() bukanlah fungsi yang aman, kita dapat menggunakannya untuk menimpa memori dimana variabel **modified** disimpan.
