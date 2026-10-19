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
