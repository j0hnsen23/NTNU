## Oppgave 1
### Kapittel 5, Oppgave 9
Page = 4k ->2^12
Adresse = 20000 -> 2^15
15-12 = 3 bits for VPN

100 111000100000
VPN Offset

VPN = 4
Offset = 3616

---

Page = 4k ->2^12
Adresse = 32769 -> 2^16

16-12 = 4 bits for VPN

1000 000000000001
VPN  Offset

VPN = 8
Offset = 1

---

Page = 4k -> 2^12
Adresse = 2^16

16 - 12 = 4 bits for VPN

1110 101001100000
VPN  Offset

VPN = 14
Offset = 2656

---

Page = 8k -> 2^13
Adresse 20000 -> 2^15

15-13 = 2 bits for VPN

10 0111000100000

VPN = 2
Offset = 3616

---

Page = 8k -> 2^13
Adresse = 32769 -> 2^16

16 - 13 = 3 bits for VPN

100 0000000000001

VPN = 4
Offset = 1

---

Page = 8k -> 2^13
Adresse = 60000 -> 2^16

16 - 13 = 3 bits for VPN

111 0101001100000

VPN = 7
Offset = 2656



## Oppgave 2
### Kapittel 5, Oppgave 10

Page = 4k -> 2^12
Adresse = 16 bits
16 - 12 = 4 bits for VPN

---

Adresse: 0010 1101 1011 1010

0010 110110111010
VPN  Offset

VPN = 2 -> PFN = 1111, present 1

Offset blir uendret, men VPN byttes med PFN
Fysisk adresse: 1111 1101 1011 1010

---
Adresse: 0110 1001 1101 0010

0110 100111010010
VPN  Offset

VPN = 6 -> PFN = 0000, present 0 
Siden ligger ikke i fysisk minne, så oversettelsen feiler.


## Oppgave 3
### Kapittel 5, Oppgave 11

4GB = 2^32
4KB = 2^12

2^32 / 2^12 = 2^(32-12) = 2^20 = 1048576 chunks

---
Bitmapen bruker 1 bit per chunk (1 = i bruk, 0 = ledig).
2^20 chunks -> 2^20 bits
2^20 bits / 8 = 2^20 / 2^3 = 2^17 bytes = 128 KB

--- 
Chunk size 2KB
2KB = 2^11
2^32 / 2^11 = 2^21 chunks -> 2^21 bits
2^21 / 2^3 = 2^18 bytes = 256 KB


## Oppgave 4
### Kapittel 5, Oppgave 13

```c
#include <stdio.h>
#include <stdlib.h>

int global = 1;

int main(void){
  static int s = 2;
  int i = 3;
  int *p = malloc(sizeof(int));

    printf("global: %p\n", (void *)&global);
    printf("static: %p\n", (void *)&s);
    printf("malloc: %p\n", (void *)p);
    printf("local:  %p\n", (void *)&i);

  free(p);
  return 0;
}
```

- Rekkefølge? global og static ligger lavest (data), så kommer malloc(heap) og local ligger høyest(stack). Det er slikt minne er bygget opp.
- Like adresser? Nei, de endrer seg hver gang fordi OS-et flytter ting tilfeldig (ASLR). Rekkefølgen er likevel alltid den samme.

## Oppgave 5
### Kapittel 5, Oppgave 14

```c
#include <stdio.h>

int main(void){
  int *p = NULL;
  printf("%d\n", *p);
  return 0;
}
```

- Programmet krasjer. NULL er adresse 0, og den er ikke mappet inn i prosessens adresserom. Når programmet prøver å lese derfra gir maskinvaren en feil og OS-et avslutter prosessen med en segmentation fault.
