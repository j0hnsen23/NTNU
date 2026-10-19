## Oppgave 1
### Kapittel 7, Oppgave 9

```assembly
movl    teller, %eax     # les verdien inn i et register
addl    $1, %eax         # legg til 1
movl    %eax, teller     # skriv den tilbake
```

1) Hva er den høyeste verdien teller kan ha til slutt?

2) Hva er den laveste?

3) Vis en rekkefølge på instruksjonene som gir den laveste verdien

4) Kompilerer du med -O2, blir det én instruksjon (addl $1, teller) i stedet for tre. Kjører du nå begge trådene på samme kjerne med taskset -c 0, er problemet borte – men med taskset -c 0,1 er det tilbake. Forklar begge deler.
  
5) Hva må til for at koden skal bli riktig uansett hvor mange kjerner den kjører på?      
