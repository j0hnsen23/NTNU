### Oppgave 1 - Kapittel 1 (Med utgangspunkt i eksemplene på C-kode og assembly-kode vi har gått gjennom i dette kapitlet, forklar hva hver linje i følgende assembly-kode gjør:)
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

### Oppgave 2 - Kapittel 1 (En cache line er 64 Byte, og en int er 4 Byte. Vi har et todimensjonalt array int a[1000][1000], som ligger radvis i minnet (altså a[0][0], a[0][1], a[0][2] … etter hverandre))
- 64 byte/4 byte = 16 int-er
- a[i][j] cacheline for cacheline
- a[j][i] element for element
- 1000000/16 = 62500 -> 1000000/62500 = 16

### Oppgave 3 - Kapittel 2 (Studer C-koden i læreboka, for eksempel eksempelet i figur 2.1 (cpu.c). For å forsikre oss om at vi får til å bruke kommandolinjeargumenter og printf(), skriv et enkelt C-program me.c som tar navnet og alderen din som kommandolinjeargumenter og skriver dem ut med printf. Programmet skal kompilere og kjøre slik)
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

### Oppgave 4 - Kapittel 3 ((OBLIG-1) Gjør Homework (Code) oppgave 1 i kapittel fem (bygg koden din på p1.c).)
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int
main(int argc, char *argv[])
{
    int x = 100;
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        int x = 200;
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else {
        // parent goes down this path (original process)
        int x = 300;
        printf("hello, I am parent of %d (pid:%d)\n",
	       rc, (int) getpid());
    }
    return 0;
}
```

### Oppgave 5 - Kapittel 3 ( (OBLIG-1) Skriv et C-program som kjører seks prosesser etter følgende tidsplan (S betyr start, T betyr terminate/avslutt):)

### Oppgave 6 - Kapittel 4 ((OBLIG-1) MLFQ har følgende regler)
