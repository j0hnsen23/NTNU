# Oppgave 1
- Her starter kodedelen
- Gjør main synlig utenfra
- Her starter selve funksjonen main
- Lagrer unna den gamle rammepekeren
- Setter opp ny rammepeker for denne funksjonen
- Setter en lokal variabel til 0
- Hopper ned til betingelsen for å sjekke løkken for kroppen kjøres
- Merkelapp i start på lølle kroppen
- i = i+1
- i = i+1
- Merkelapp: betingelsesjekken
- sammenligner i med 9
- hvis i <= 9 hopp til løkkekroppen
- legg 0 i %eax
- gjenopprett til den gamle rammepekeren
- returnerer til den som kalte main

# Oppgave 2
- 64 byte/4 byte = 16 int-er
- a[i][j] cacheline for cacheline
- a[j][i] element for element
- 1000000/16 = 62500 -> 1000000/62500 = 16

# Oppgave 3
```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    if (argc != 3) {
        fprintf(stderr, "usage: me <navn> <alder>\n");
        exit(1);
    }
    char *navn = argv[1];
    char *alder = argv[2];

    printf("Navn: %s\n", navn);
    printf("Alder: %s\n", alder);

    return 0;
}
```
