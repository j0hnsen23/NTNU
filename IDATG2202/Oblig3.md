## Oppgave 1
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
- Problemet er at tråd 1 holder lås A og venter på B, mens tråd 2 holder lås B og venter på A. De venter på hverandre, og ingen kommer videre.
- Regelen er at alle tråder tar låsene i fast rekkefølge, alltid kontoen med lavest nummer først. Da kan en tråd bare vente på en lås som kommer senere i rekkefølgen, så det kan aldri oppstå en ring der trådene venter på hverandre.
- Den tredje tråden følger samme regel og tar B før C. Løsningen virker fortsatt, fordi rekkefølgen A, B, C gjelder for alle tråder, uansett hvor mange det er.
  
4) En kollega foreslår å løse det ved å la hver tråd vente et tilfeldig antall millisekunder og prøve på nytt hvis den ikke får begge låsene. Hvorfor er nummerering en bedre løsning?
- Nummererering er bedre fordi den fjerner årsaken til deadlock. Når trådene tar låsene i samme rekkefølge, oppstår det ikke en ring der trådene venter på hverandre. Tilfeldig venting fjerner ikke årsaken, den gjør problemet mindre sannsynlig. Trådene kan fortsatt kollidere uten å komme videre og kan fortsatt kaste bort tid på å slippe låsene og vente.
---
## Oppgave 3 
### Kapittel 7, Oppgave 12
#### Homework 1
<img width="667" height="523" alt="Valgrind" src="https://github.com/user-attachments/assets/21428ab3-4c7b-4f43-9cff-d5b5a6524e8f"/>
- I main-race.c øker både hovedtråden og worker-tråden variabelen balance med balance++ uten lås. Det er en data race, fordi begge trådene kan bruke variabelen samtidig og da kan en av økningene bli overskrevet.
- I rapporten fra Helgrind så finner den racet og peker på linje 15 i main og linje 8 i worker.
- Helgrind forteller også hvilke tråder som krasjer med hverandre, hvor tråd #2 ble laget (linje 14), at ingen av trådene brukte lås og at det er variabelen balance det gjelder. Den finner også to feil fordi balance++ gjør to ting: den leser verdien og skriver en ny. Begge delene skjer uten lås, så begge blir meldt som feil.  

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
- Helgrind rapporterer fortsatt data race. Den ene tråden har «Locks held: 1» og den andre «Locks held: none». Den andre tråden bruker ikke låsen og kan derfor endre balance samtidig. En lås hjelper bare hvis alle tråder bruker den.

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
```c
#include <stdio.h>

#include "common_threads.h"

pthread_mutex_t m1 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t m2 = PTHREAD_MUTEX_INITIALIZER;

void* worker(void* arg) {
    if ((long long) arg == 0) {
        Pthread_mutex_lock(&m1);
        Pthread_mutex_lock(&m2);
    } else {
        Pthread_mutex_lock(&m2);
        Pthread_mutex_lock(&m1);
    }
    Pthread_mutex_unlock(&m1);
    Pthread_mutex_unlock(&m2);
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p1, p2;
    Pthread_create(&p1, NULL, worker, (void *) (long long) 0);
    Pthread_create(&p2, NULL, worker, (void *) (long long) 1);
    Pthread_join(p1, NULL);
    Pthread_join(p2, NULL);
    return 0;
}
```

- De to trådene tar de samme to låsene i motsatt rekkefølge. Tråd 1 tar m1 også m2, mens tråd 2 tar m2, også m1. Hvis tråd 1 tar m1 og tråd 2 tar m2 samtidig, venter begge på hverandre og den låsen som den andre holder og ingen kommer seg videre. Det blir en deadlock, men den skjer ikke hver gang bare når trådene bytter på akkurat feil tidspunkt. 
---
#### Homework 4
<img width="474" height="507" alt="image" src="https://github.com/user-attachments/assets/713c6f7c-6ea2-4d94-a61f-ba6d9308890e" />
- Helgrind rapporterer en feil ''lock order violated''. Den viser at den første tråden tok låsene i rekkefølgen m1 og så m2, mens tråd #3 tok dem i motsatt rekkefølge m2 og så m1. Helgrind finner feilen selv om programmet ikke hang, fordi den ser at låsene tas i bruk i ulik rekkefølge og at deadlock kan oppstå.

---

## Oppgave 4
### Kapittel 7, Oppgave 13
- Forkcount:
- <img width="646" height="750" alt="image" src="https://github.com/user-attachments/assets/755aab59-9159-4ef3-815d-8376305b1ffe" />

- Threadcount:
- <img width="643" height="384" alt="image" src="https://github.com/user-attachments/assets/ddce0022-602f-4042-b6fc-9437753fac34" />

- I forkcount har hver prosess sin egen kopi av g_ant, så begge teller til 30 hver for seg. I threadcount deler trådene samme g_ant, så de teller til 30 sammen, og fordi det ikke er noen lås, kan det oppstå race conditions, der begge trådene leser samme verdi samtidig, slik at et tall telles to ganger eller en økning forsvinner.

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
- Child kjører først: Semaforen starter på 0. Parent skriver "parent: begin" og lager child. Child skriver "child" og gjør sem_post, så semaforen blir 1. Parent kommer til sem_wait, semaforen blir 0 og parent fortsetter uten å vente. Parent skriver "parent: end".
- Parent kommer til sem_wait først: Semaforen starter på 0. Parent skriver "parent: begin", lager child og kommer til sem_wait. Semaforen blir -1 og parent sover. Child skriver "child" og gjør sem_post. Semaforen blir 0 og parent vekkes og skriver "parent: end".
  
3) Hva skjer hvis X settes til 1?
- Med X = 1 starter semaforen på 1, så sem_wait i parent går rett gjennom uten å vente. Da venter ikke parent på child, så «child» kan bli skrevet før eller etter «parent: end», eller ikke i det hele tatt hvis programmet avslutter først. Rekkefølgen er altså ikke garantert.
  
4) Hvorfor kan vi ikke bruke en mutex i stedet for semaforen her?
- En semafor kan brukes til å vente på at en annen tråd gir beskjed. En mutex må låses opp av samme tråd som låste den, så parent kan ikke låse den og så la child låse den opp. Derfor passer den ikke til å vente på en annen tråd.


---

## Oppgave 6
### Kapittel 8, Oppgave 11
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

- To tråder (den nye tråden og main) kjører writeloop samtidig og deler samme g_ant.
- Hver tråd øker g_ant, sover litt og skriver så ut verdien.
- Mens en tråd sover kan den andre øke g_ant, så verdien kan ha endret seg før utskriften skjer. Derfor kan tall hoppes over eller skrives ut to ganger.
- Siden det er ikke er noen lås her så kan begge trådene se 9 samtidig og øke slik at g_ant blir 11.
- exit(0) avslutter hele programmet når den første tråden er ferdig, så den andre tråden kan bli stoppet før den skriver ut sitt siste tall.


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
- g_ant deles av begge trådene, så uten lås oppstår det race conditions.
- Den kritiske seksjonen er sjekken, økningen og utskriften, fordi en tråd kan bli avbrutt midt i, særlig under usleep.
- Bruker en mutex og låser før while-løkka slik at hele løkka blir en kritisk seksjon. Den første tråden som får låsen teller fra 1 til 10 alene.
- Når den er ferdig, avslutter exit(0) hele programmet, så den andre tråden skriver aldri ut noe.
- Å låse inne i løkka og låse opp utenfor ville gitt deadlock, fordi tråden prøver å ta en lås den allerede holder.
- Ulempen er at trådene ikke lenger jobber samtidig, så vi mister parallelliteten.

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
  int readers;
} rwlock_t;

void rwlock_init(rwlock_t *lock) {
  lock->readers = 0;
  Sem_init(&lock->lock, 1);
  Sem_init(&lock->writelock, 1);
}

void rwlock_acquire_readlock(rwlock_t *lock) {
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

void rwlock_acquire_writelock(rwlock_t *lock) { Sem_wait(&lock->writelock); }

void rwlock_release_writelock(rwlock_t *lock) { Sem_post(&lock->writelock); }

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
