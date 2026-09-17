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

    printf("Yo, Im %s and Im at least %s years old\n", navn, alder);

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

```c
  #include <stdio.h>     /* printf */
  #include <stdlib.h>    /* exit */
  #include <unistd.h>    /* fork */
  #include <sys/wait.h>  /* waitpid */
  #include <sys/types.h> /* pid_t */
  /* Note: pid_t is probably just an int, but it might be different
     kind of ints on different platforms, so using pid_t instead of
     int helps makes the code more platform-independent 
  */

  void process(int number, int time) {
    printf("Process %d is running\n", number);
    sleep(time);
    printf("Prosess %d ran for %d seconds\n", number, time);
  }

int start[6] = {0,1,0,3,1,4}
int duration[6] = {1,2,3,2,3,3}

int main(){
	pid_t pids[6];

	for(int i = 0; i < 6; i++){
		pids[i] = fork();
		if(pids[i] == 0){
			sleep(start[i]);
			process(i,duration[i]);
			exit(0);
	}
}

for(int i = 0; i < 6; i++){
	waitpid(pids[i],NULL,0);
}

printf("Alle prosesser er ferdige.\n);
return 0;
}
```

### Oppgave 6 - Kapittel 4 ((OBLIG-1) MLFQ har følgende regler)
- 1) Når P0 er ferdig, forlater den kø:
	- Frekvensen er lavere enn timeslice så forlater aldri kø 3
- 2) Når P1 er ferdig, forlater den kø:
	- Den forlater kø 3, ettersom den fikk priority boost
- 3) Når P2 er ferdig, forlater den kø:
	- Q0, den får boost ved t=50 så ved t=60 rykker ned til Q2, ved t=65 rykker ned til Q1, ved Q70 rykker ned til Q0
- 4) Hvilken kø ligger P0 i på tidspunkt 15?
 - Q3 fordi den går aldri ned fra Q3
- 5) Turnaround time for P1 (ms):
	- Turnaround time = Sluttid - Ankomstid = 60-0 = 60ms
- 6) Gjennomsnittlig turnaround time (ms):
	- p0 = 34-0 = 34ms
   	- p1 = 60-0 = 60ms
   	- p2 = 80-0 = 80ms
   	- Gjennomsnittlig turnaround tid = 58 ms
- 7) Response time for P2 (ms):
	- 9 - se tabell
- 8) Gjennomsnittlig response time (ms):
	- p0 = 1
  	- p1 = 4
  	- p2 = 9
  	- 14/3 = 4.67 ms
- 9) Er CPU-en opptatt hele tiden, eller står den idle en periode?
	- Ja den er opptatt hele tiden fra 0-80ms, alltid en prosess i køen.
- 10) På tidspunkt 20 ankommer en ny prosess P3 med kjøretid 10 og uten I/O. Hva blir gjennomsnittlig turnaround time og gjennomsnittlig response time nå?
  - Turnaround time:
  - p0 = 36ms
  - p1 = 70ms
  - p2 = 90ms
  - p3 = 45-20 = 25ms
  - total = 221
  - gjn = 55.25ms
 
  - Response time:
  - p0 = 1ms
  - p1 = 4ms
  - p2 = 9ms
  - p3 = 23-20 = 3 ms
  - gjn = 4.25ms
