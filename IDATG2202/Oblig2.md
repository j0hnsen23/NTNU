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
- Like adresser? Nei, de endrer seg hver gang fordi OS-et flytter ting tilfeldig. Rekkefølgen er likevel alltid den samme.

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
1. *i ber CPU om å lese 4 byte fra virtuell adresse 0
2. MMU deler adresse: VPN=0 Offset=0
3. OS har page 0 umappet, present bit = 0 (For å fange NULL pekere)
4. MMU kan ikke oversette -> utløser exception -> OS tar over.
5. OS ser at adressen ikke hører til et gyldig område i prossessen (Ulovlig adresse)
6. OS sender SIGSEGV -> prosessen avsluttes.
(exit status til forrige kommando)
gir 139 = 128 + 11
Bash legger til 128
Segfault = SIGSEGV = 11

- Programmet krasjer. NULL er adresse 0, og den er ikke mappet inn i prosessens adresserom. Når programmet prøver å lese derfra gir maskinvaren en feil og OS-et avslutter prosessen med en segmentation fault.


## Oppgave 6 
### Kapittel 6, Oppgave 11
<img width="245" height="194" alt="Oppgave6" src="https://github.com/user-attachments/assets/d7ab6b3a-92b4-4e7c-b52d-74b8fcbfc42a" />

Offset: Page størrelse er 4KB = 2^12 byte så 12 bit offset
Sidenummer = 32-12 = 20 bit til sidenummeret
Tabellene har 1024 oppføringer, og 1024 = 2¹⁰, så 10 bit til hvert nivå. 10 - Top level | 10 - Second level

1) Store bokstaver (top level) - rammenummeret (PFN) til en andre-nivå page table
2) Små bokstaver (second level) - PFN til page-framen der dataene ligger
3) Biten til høyre - present-bit

4)  0000001001 | 0000000110 | 110110111010
    9            6            110110111010
    A            b            110110111010

    A (present-bit 1) peker på andre-nivå-tabellen. Der gir indeks 6 rammenummeret b (present-bit 1). Offset kopieres uendret.
    
    Den fysiske adressen blir b | 1101 1011 1010



## Oppgave 7
### Kapittel 6, Oppgave 12
Arrayen: a[0] - a[2999]
Page størrelse: 4KB -> 4096 byte
En int: 4 byte

1) 4096 / 4 = 1024 int-er
2) 3000 / 1024 = 2.929 ≈ 3 pager
3) 3000-3 = 2997 , hit rate = 2997/3000 * 100 = 99.9% (En miss hver gang en ny page brukes og resten av oppslagene blir hits(Arrayen dekker 3 pager og får dermed kun 3 misser)).
4) TLB-en lagrer en oversettelse per page, ikke per element. Etter første oppslag ligger oversettelsen i TLB-en, så resten av elementene i samme page blir hits. Siden programmene ligger etter hverandre så bruker man romslig (Spatial - ligger etter hverandre) lokalitet.


## Oppgave 8
### Kapittel 6, Oppgave 13
En prosess gjør oppslag i denne rekkefølgen med pages: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

1)
- FIFO (First in First Out) og 3 page frames: 9 PAGE FAULTS
- 1 // PAGE FAULT
- 2 // PAGE FAULT
- 3 // PAGE FAULT
- 4 // PAGE FAULT - Kaster 1 ut
- 1 // PAGE FAULT - Kaster 2 ut
- 2 // PAGE FAULT - Kaster 3 ut
- 5 // PAGE FAULT - Kaster 4 ut
- 1 // Allerede i page frame
- 2 // Allerede i page frame
- 3 // PAGE FAULT - Kaster 1 ut
- 4 // PAGE FAULT - Kaster 2 ut
- 5 // Allerede i page frame

2)
- LRU (Least recently used) og 3 page frames: 10 PAGE FAULTS
- 1 // PAGE FAULT
- 2 // PAGE FAULT
- 3 // PAGE FAULT
- 4 // PAGE FAULT - Kaster ut 1
- 1 // PAGE FAULT - Kaster ut 2
- 2 // PAGE FAULT - Kaster ut 3
- 5 // PAGE FAULT - Kaster ut 4
- 1 // Allerede i page frame
- 2 // Allerede i page frame
- 3 // PAGE FAULT - Kaster ut 5
- 4 // PAGE FAULT - Kaster ut 1
- 5 // PAGE FAULT - Kaster ut 2

3)
- optimal (se inn i fremtiden) og 3 page frames: 7 PAGE FAULTS
- 1 // PAGE FAULT
- 2 // PAGE FAULT
- 3 // PAGE FAULT
- 4 // PAGE FAULT - Kaster ut 3 (Lengst i fremtiden)
- 1 // Allerede i page frame
- 2 // Allerede i page frame
- 5 // PAGE FAULT - Kaster ut 4 (Lengst i fremtiden)
- 1 // Allerede i page frame
- 2 // Allerede i page frame
- 3 // PAGE FAULT - Kaster ut 1 (Kunne brukt 2, lengst i fremtiden)
- 4// PAGE FAULT - Kaster ut 2 (Kunne brukt 3, lengst i fremtiden)
- 5 // Allerede i page frame
  
4)
- FIFO (First in First Out) og 4 page frames: 10 PAGE FAULTS
- 1 // PAGE FAULT
- 2 // PAGE FAULT
- 3 // PAGE FAULT
- 4 // PAGE FAULT
- 1 // Allerede i page frame
- 2 // Allerede i page frame
- 5 // PAGE FAULT - Kaster ut 1
- 1 // PAGE FAULT - Kaster ut 2
- 2 // PAGE FAULT - Kaster ut 3
- 3 // PAGE FAULT - Kaster ut 4
- 4 // PAGE FAULT - Kaster ut 5
- 5 // PAGE FAULT - Kaster ut 1

Kalles Belady's Anomaly - flere pages gir flere faults. (FIFO)


## Oppgave 9
### Kapittel 6, Oppgave 14
1) 0.99 * 100 + 0.01 * 200 = 99 + 2 = 101 ns fordi den må kjøre ett ekstra minne oppslag.
2) (990000 * 100ns + 1 * 10000200ns + 9999 * 200ns) / 1000000 = 111ns
   - 990000 = TLB hits = 100ns
   - 1 major page fault = 10000200ns
   - 9999 light faults = 200ns 
3) 10 ms = 10 000 000 ns (10 000 000ns / 1ns = 10 000 000 oppslag) -> Man kan høyst ha omtrent en major page fault per 10 millioner oppslag.
4) Fordi en major page fault koster ca 100 000 ganger mer enn et vanlig minneoppslag, så skal det svært få faults til før de tar mesteparten av kjøretiden. Når maskinen swapper skjer page faults ofte, og prosessoren bruker nesten all tiden på å vente på disk. Derfor føles det ut som om maskinen har stoppet opp helt.
