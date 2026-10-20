<img width="721" height="46" alt="image" src="https://github.com/user-attachments/assets/1f6ecb2f-e08f-4f89-8939-10a15dfbb62a" /><img width="583" height="717" alt="image" src="https://github.com/user-attachments/assets/2073e279-d5b0-4dae-8e64-fcd8533d9c00" />## Oppgave 1
### Kapittel 7, Oppgave 9

```assembly
movl    teller, %eax     # les verdien inn i et register
addl    $1, %eax         # legg til 1
movl    %eax, teller     # skriv den tilbake
```

1) Hva er den høyeste verdien eller kan ha til slutt?
- Den høyeste verdien teller kan ha til slutt er 2.
- To tråder legger hver til 1. Hvis de gjør det etter tur, uten å forstyrre hverandre så blir det 0->1->2.
  
2) Hva er den laveste?
- Den laveste verdien er 1.
- Dette er en race condition. teller++ er en read-modify-write som ikke er atomisk og den kritiske sektoren mangler mutual exclusion.
- Ved en context switch mellom movl og movl kan begge trådene laste 0 inn i sitt eget %eax. Begge skriver 1, og den ene skrivingen overskriver den andre.
  
3) Vis en rekkefølge på instruksjonene som gir den laveste verdien
- T1: movl teller, %eax   # T1 leser 0
- T2: movl teller, %eax   # T2 leser 0
- T1: addl $1, %eax       # T1: eax = 1
- T2: addl $1, %eax       # T2: eax = 1
- T1: movl %eax, teller   # teller = 1
- T2: movl %eax, teller   # teller = 1
- Begge trådene leser teller før noen av dem har skrevet tilbake. Begge regner derfor ut 1 i sitt eget register, og den siste skrivingen overskriver den første. Én økning går tapt (lost update).
  
4) Kompilerer du med -O2, blir det én instruksjon (addl $1, teller) i stedet for tre. Kjører du nå begge trådene på samme kjerne med taskset -c 0, er problemet borte – men med taskset -c 0,1 er det tilbake. Forklar begge deler.
- En kjerne: Context switch skjer bare mellom instruksjoner, så en instruksjon kan ikke avbrytes. Problemet med overskriving blir borte.
- To kjerner: Trådene kjører samtidig og begge kan lese 0 inne i addl før noen skriver. Problemet med overskriving er tilbake
  
5) Hva må til for at koden skal bli riktig uansett hvor mange kjerner den kjører på?
- Operasjonen må gjøres atomisk, slik at ingen annen tråd eller kjerne kan komme inn midt i den.
- Mutex: Lås rundt teller++, så bare en tråd om gangen kommer inn
  - pthread_mutex_lock(&m);
  - teller++;
  - pthread_mutex_unlock(&m);
- En atomisk instruksjon gjør les, legg til og skriv i ett eneste steg som ikke kan deles opp. Med lock foran sier CPU-en til de andre kjernene: «Ikke rør denne variabelen før jeg er ferdig.» Da kan ingen andre lese den gamle verdien midt i operasjonen, og ingen økninger går tapt.

---

## Oppgave 2
### Kapittel 7, Oppgave 10
1) Vis en rekkefølge som fører til deadlock.
- T1: lås(A)       # T1 holder A
- T2: lås(B)       # T2 holder B
- T1: lås(B)       # venter på T2
- T2: lås(A)       # venter på T1
  
2) Hvilken betingelse fra kapittelteksten er det som er oppfylt her, og som gjør deadlock mulig i det hele tatt?
- Sirkulær venting (circular wait): T1 venter på T2, som venter på T1. Dette oppstår fordi trådene tar låsene i motsatt rekkefølge.
  
3) Skriv om reglene for hvordan trådene tar låsene, slik at deadlock ikke kan skje. Virker løsningen også hvis det kommer en tredje tråd som flytter penger mellom B og en konto C?
- Gi kontoene en fast rekkefølge (A<B<C) og la alle tråder alltid låse den laveste først. Da tar både T1 og T2 A før B, og det kan ikke oppstå sirkulær venting. Det virker også med en tredje tråd mellom B og C, fordi alle følger samme rekkefølge.
  
4) En kollega foreslår å løse det ved å la hver tråd vente et tilfeldig antall millisekunder og prøve på nytt hvis den ikke får begge låsene. Hvorfor er nummerering en bedre løsning?
-Tilfeldig venting fjerner ikke årsaken, den bare prøver igjen. Trådene kan fortsatt kollidere gang på gang, det sløser tid, og det gir ingen garanti. Nummerering gjør deadlock umulig fra start, er enkel og har ingen ekstra ventetid.

---
## Oppgave 3 
### Kapittel 7, Oppgave 12
#### Homework 1
<img width="667" height="523" alt="Valgrind" src="https://github.com/user-attachments/assets/21428ab3-4c7b-4f43-9cff-d5b5a6524e8f"/>
- Data race: Både worker (linje 8) og main (linje 15) gjør balance++ uten lås, så trådene kan overskrive hverandres oppdateringer.
- Riktige linjer? Ja, helgrind peker på linje 8 og 15, der balance endres.
- Annen informasjon: Hvilke tråder som er involvert (#1 og #2) og hvor tråd #2 ble opprettet (linje 14), at det gjelder både lesing og skriving av 4 bytes, at variabelen heter balance, og at ingen låser var holdt («Locks held: none»).

#### Homework 2
1. Fjerne en linje: Kommenterer ut balance++
```c
#include <stdio.h>

#include "common_threads.h"

int balance = 0;

void* worker(void* arg) {
    //balance++; // unprotected access
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    Pthread_create(&p, NULL, worker, NULL);
    balance++; // unprotected access
    Pthread_join(p, NULL);
    return 0;
}
```
- Helgrind rapporterer 0 feil. Bare én tråd bruker balance, så det er ingen data race.
---
2. Lås rundt en oppdatering
```c
#include <stdio.h>

#include "common_threads.h"

int balance = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

void* worker(void* arg) {
    pthread_mutex_lock(&m);
    balance++; // protected access
    pthread_mutex_unlock(&m);
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    Pthread_create(&p, NULL, worker, NULL);
    balance++; // unprotected access
    Pthread_join(p, NULL);
    return 0;
}
```
- Helgrind rapporterer fortsatt data race. Den ene tråden har «Locks held: 1» og den andre «Locks held: none». En lås hjelper bare hvis alle tråder bruker den.

3. Låse rundt begge oppdateringer
```c
#include <stdio.h>

#include "common_threads.h"

int balance = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

void* worker(void* arg) {
    pthread_mutex_lock(&m);
    balance++; // protected access
    pthread_mutex_unlock(&m);
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    Pthread_create(&p, NULL, worker, NULL);
    pthread_mutex_lock(&m);
    balance++; // protected access
    pthread_mutex_unlock(&m);
    Pthread_join(p, NULL);
    return 0;
}
```
- Helgrind rapporterer 0 feil. Begge tilgangene til balance er beskyttet av samme mutex, så bare én tråd kan endre den om gangen.
---
#### Homework 3
Problemet er at de to trådene tar de samme to låsene i motsatt rekkefølge: den ene låser m1 og så m2, mens den andre låser m2 og så m1. Hvis tråd 1 får m1 og tråd 2 får m2 samtidig, venter tråd 1 på m2 og tråd 2 på m1. Ingen av dem slipper låsen sin, så begge venter for alltid. Dette er deadlock på grunn av sirkulær venting, og det skjer bare hvis trådene flettes på akkurat denne måten.
---
#### Homework 4
<img width="474" height="507" alt="image" src="https://github.com/user-attachments/assets/713c6f7c-6ea2-4d94-a61f-ba6d9308890e" />
- Helgrind rapporterer et lock order violation: m1 og m2 tas i motsatt rekkefølge av de to trådene, og den viser hvilke linjer det skjer på. Den advarer selv om programmet ikke hang i akkurat den kjøringen.

---

## Oppgave 4
### Kapittel 7, Oppgave 13
- Forkcount:
- <img width="646" height="750" alt="image" src="https://github.com/user-attachments/assets/755aab59-9159-4ef3-815d-8376305b1ffe" />

- Threadcount:
- <img width="643" height="384" alt="image" src="https://github.com/user-attachments/assets/ddce0022-602f-4042-b6fc-9437753fac34" />

- I forkcount har hver prosess sin egen kopi av g_ant, så begge teller til 30 hver for seg. I threadcount deler trådene samme g_ant, så de teller til 30 sammen, og fordi det ikke er noen lås, kan det oppstå race conditions.

---

## Oppgave 5
### Kapittel 8, Oppgave 9
```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

sem_t s;

void *child(void *arg) {
    printf("child\n");
    sem_post(&s);
    return NULL;
}

int main(void) {
    sem_init(&s, 0, X);
    printf("parent: begin\n");
    pthread_t c;
    pthread_create(&c, NULL, child, NULL);
    sem_wait(&s);
    printf("parent: end\n");
    return 0;
}
```

1) Hvilken verdi må X ha for at utskriften alltid skal bli parent: begin, child, parent: end?
- Verdien som X må være for å få den utskriften er 0.
  
2) Gå gjennom de to mulige rekkefølgene med den verdien: én der child-tråden rekker å kjøre først, og én der parent kommer til sem_wait() først. Hva er verdien til semaforen underveis?
- Child kjører først: Semaforen starter på 0. Parent skriver «parent: begin» og lager child. Child kjører, skriver «child» og gjør sem_post, så semaforen blir 1. Parent kommer til sem_wait, trekker 1 (semaforen blir 0) og fortsetter uten å vente. Parent skriver «parent: end».
- Parent kommer til sem_wait() først: Semaforen starter på 0. Parent skriver «parent: begin», lager child og kommer til sem_wait. Semaforen blir −1, og parent sover. Child skriver «child» og gjør sem_post. Semaforen blir 0, og parent vekkes. Parent skriver «parent: end».
  
3) Hva skjer hvis X settes til 1?
- Med x=1 starter semaforen på 1, så sem_wait i parent går rett gjennom å vente. Parent er da ikke lenger avhengig av at child har kjørt. Eller til og med at programmet avslutter før child rekker å skrive «child» i det hele tatt. Rekkefølgen er altså ikke garantert. Hvis child likevel rekker å kjøre før sem_wait, blir semaforen 2 etter sem_post, og 1 etter sem_wait.
  
4) Hvorfor kan vi ikke bruke en mutex i stedet for semaforen her?
-En semafor kan brukes til å vente på at en annen tråd gir beskjed. En mutex kan bare låses og låses opp av samme tråd, så den egner seg ikke til dette.


---

## Oppgave 6
### Kapittel 7, Oppgave 11
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

int g_ant = 0;

void *writeloop(void *arg) {
    while (g_ant < 10) {
        g_ant++;
        usleep(rand() % 10);
        printf("%d\n", g_ant);
    }
    exit(0);
}

int main(void) {
    pthread_t tid;
    pthread_create(&tid, NULL, writeloop, NULL);
    writeloop(NULL);
    pthread_join(tid, NULL);
    return 0;
}
```
Forklar hvordan programmet virker, og hvorfor det sannsynligvis ikke skriver ut dette:
- 1
- 2
- 3
- 4
- 5
- 6
- 7
- 8
- 9
- 10
- Programmet har en global variabel g_ant så alle tråder deler den samme kopien.
- I main startes en ny tråd med pthread_create, og den tråden kjører writeloop. Rett etterpå kaller main selv writeloop(NULL). Dermed kjører to tråder den samme løkka samtidig, og begge jobber på den samme g_ant.
- Hver tråd: Sjekker om g_ant < 10, øker g_ant med 1, sover et tilfeldig antall mikrosekunder 0-9, skriver ut verdien av g_ant slik den er akkurat da.

- Grunnen til at det ikke skriver ut 1-10 er fordi utskriften skjer etter søvnen og ikke rette etter økning.Mens en tråd sover kan en annen rekke å øke antallet og deretter skrives en annen verdi ut.


Kompiler og kjør programmet. Legg til en låsemekanisme du selv velger, slik at det bare skriver ut tallene fra én til ti i rekkefølge, som vist over. Begrunn valgene dine.
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

int g_ant = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

void *writeloop(void *arg) {
    pthread_mutex_lock(&m);
    while (g_ant < 10) {
        g_ant++;
        usleep(rand() % 10);
        printf("%d\n", g_ant);
    }
    pthread_mutex_unlock(&m);
    exit(0);
}

int main(void) {
    pthread_t tid;
    pthread_create(&tid, NULL, writeloop, NULL);
    writeloop(NULL);
    pthread_join(tid, NULL);
    return 0;
}
```
- g_ant er en delt ressurs, og uten synkronisering oppstår en race condition. Kritisk seksjoner sjekken, økningen og utskriften fordi trådene kan bli preemptet midt i, spesielt under usleep. Låsen er plassert før while, slik at hele løkka blir en kritisk seksjon med mutual exclusion. Den første tråden som får mutexen, utfører alle iterasjonene atomisk sett utenfra og terminerer prosessen med exit mens den fortsatt holder låsen. Den andre tråden blir blokkert.Å låse inne i løkka med unlock utenfor ville gitt vranglås (deadlock), fordi tråden prøver å ta en lås den allerede holder. Ulempen er at løsningen serialiserer arbeidet og fjerner parallelliteten.

---

## Oppgave 8
### Kapittel 7, Oppgave 12
b) Start med fila reader-writer.c, som er en kombinert versjon av reader-writer.c og rwlock.c, der readerne og writerne har fått nummer. Kompiler og kjør den med én writer og to readers i ti runder:
./reader-writer 2 1 10
<img width="1239" height="721" alt="image" src="https://github.com/user-attachments/assets/7ad6f105-a38d-4954-81b7-fb34a0a82f41" />

c) Kjør med to writers og ti readers for å se starvation-problemet.
<img width="454" height="880" alt="image" src="https://github.com/user-attachments/assets/73b301e2-ab8b-45e7-8fc2-efeb05c9acd8" />

d) Endre koden slik at nye readers ikke slipper til å lese hvis en writer vil skrive, se side 75 i The Little Book of Semaphores (hint: du trenger bare legge til seks linjer kode).

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#include "common_threads.h"

#ifdef linux
#include <semaphore.h>
#elif __APPLE__
#include "zemaphore.h"
#endif

#define MAX_MOD_READ 100
#define MAX_MOD_WRITE 100

typedef struct _rwlock_t {
  sem_t writelock; 
  sem_t lock;
  sem_t turnstille; // 1
  int readers;
} rwlock_t;

void rwlock_init(rwlock_t *lock) {
  lock->readers = 0;
  Sem_init(&lock->lock, 1);
  Sem_init(&lock->writelock, 1);
  Sem_init(&lock->turnstille,1); // 2
}

void rwlock_acquire_readlock(rwlock_t *lock) {
  Sem_wait(&lock->turnstille); // 3
  Sem_wait(&lock->turnstille); // 4
  Sem_wait(&lock->lock);
  lock->readers++;
  if (lock->readers == 1) Sem_wait(&lock->writelock);
  Sem_post(&lock->lock);
}

void rwlock_release_readlock(rwlock_t *lock) {
  Sem_wait(&lock->lock);
  lock->readers--;
  if (lock->readers == 0) Sem_post(&lock->writelock);
  Sem_post(&lock->lock);
}

void rwlock_acquire_writelock(rwlock_t *lock) {
  Sem_wait(&lock->writelock);
  Sem_wait(&lock->turnstille); // 5
}
  
void rwlock_release_writelock(rwlock_t *lock) {
Sem_post(&lock->writelock);
Sem_post(&lock->turnstille); // 6
}

int loops;
int value = 0;

rwlock_t lock;

void *reader(void *arg) {
  int i;
  for (i = 0; i < loops; i++) {
    rwlock_acquire_readlock(&lock);
    printf("reader %ld reads %d\n", (intptr_t)arg, value);
    usleep(random() % MAX_MOD_READ);
    rwlock_release_readlock(&lock);
    usleep(random() % MAX_MOD_READ);
  }
  return NULL;
}

void *writer(void *arg) {
  int i;
  for (i = 0; i < loops; i++) {
    rwlock_acquire_writelock(&lock);
    value++;
    printf("writer %ld writes %d\n", (intptr_t)arg, value);
    usleep(random() % MAX_MOD_WRITE);
    rwlock_release_writelock(&lock);
    usleep(random() % MAX_MOD_WRITE);
  }
  return NULL;
}

int main(int argc, char *argv[]) {
  assert(argc == 4);
  int num_readers = atoi(argv[1]);
  int num_writers = atoi(argv[2]);
  loops = atoi(argv[3]);

  pthread_t pr[num_readers], pw[num_writers];

  rwlock_init(&lock);

  printf("begin\n");

  intptr_t i;
  for (i = 0; i < num_readers; i++)
    Pthread_create(&pr[i], NULL, reader, (void *)i);
  for (i = 0; i < num_writers; i++)
    Pthread_create(&pw[i], NULL, writer, (void *)i);

  for (i = 0; i < num_readers; i++) Pthread_join(pr[i], NULL);
  for (i = 0; i < num_writers; i++) Pthread_join(pw[i], NULL);

  printf("end: value %d\n", value);

  return 0;
}
```
