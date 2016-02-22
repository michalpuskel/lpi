Cvičenie 1
==========

Vašou hlavnou úlohou na tomto cvičení je:
* vytvoriť si konto na https://github.com/ (ak ešte nemáte)
* poslať mailom na  adresu `siska@ii.fmph.uniba.sk` vaše univerzitné
  prihlasovacie id (login do AIS-u, v tvare priezviskoCISLO) a prihlasovacie
  meno do github-u.  **Do predmetu uveďte `Mat4 registracia`**.
* po vyriešení týchto cvík si **odložiť riešenie 2. príkladu**,
  na budúcich cvičeniach si ukážeme, ako sa odovzdá v github-e.

SAT solver
----------
SAT solverov je veľa, budeme používať [MiniSAT](http://minisat.se/).
Binárka pre Windows sa dá stiahnuť priamo na ich stránke, ale potrebuje ešte
2 knižnice (cygwin1.dll, cygz.dll). Všetky tri súbory sa nachádzajú v adresári
s [nástrojmi](../tools/).

Všetky potrebné súbory k tomuto cvičeniu si môžete stiahnuť pohromade
ako jeden zip súbor
[cv01.zip](https://github.com/FMFI-UK-1-AIN-412/lpi/archive/cv01.zip).

1. príklad
----------
Chceme na párty pozvať aspoň niekoho z trojice Jim, Kim a Sára, bohužiaľ každý
z nich má nejaké svoje podmienky.

* Sára nepôjde na párty ak pôjde Kim.
* Jim pôjde na párty, len ak pôjde Kim.
* Sára nepôjde bez Jima.

Zapísané vo výrokovej logike (premenná `kim` znamená, že Kim pôjde na párty,
atď.):
```
kim → ¬sarah
jim → kim
sarah → jim
kim ∨ jim ∨ sarah
```

Prerobené do CNF (konjunktívnej normálnej formy):
```
¬kim ∨ ¬sarah
¬jim ∨ kim
¬sarah ∨ jim
kim ∨ jim ∨ sarah
```

### DIMACS CNF formát ###
```
p cnf VARS CLAUSES
1 2 -3 0
...
```
```
p cnf 3 4
c -kim v -sarah
-1 -3 0
c -jim v kim
-2 1 0
c -sarah v jim
-3 2 0
c kim v jim v sarah
1 2 3 0
```

Aby bola práca s DIMACS CNF súbormi vo Windows jednoduchšia, budeme im dávať
príponu `.txt`, t.j. budeme sa tváriť ako by to boli obyčajné textové súbory.

### SAT solver ###

Spustíme SAT solver, ako parameter dáme meno vstupného súboru. MiniSAT
normálne iba vypíše, či je vstup splniteľný. Ak chceme aj nejaký výstup, tak
dáme ešte meno výstupného súboru (MiniSAT ho vytvorí/prepíše.)
```
$ minisat party.txt party.out
...
SATISFIABLE
$ cat party.out
SAT
1 -2 -3 0
```

#### Hľadanie riešení ####

MiniSAT nájde len nejaké riešenie, ak chceme nájsť ďašie, môžeme mu povedať,
že toto konkrétne nechceme (nemajú naraz platiť tieto premenné). Toto riešenie
je `kim ∧ ¬jim ∧ ¬sarah`, znegovaním dostaneme `¬kim ∨ jim ∨ sarah`, čo je
priamo disjunktívna klauza a môžeme ju pridať k zadaniu:

```
p cnf 3 5
-1 -3 0
-2 1 0
-3 2 0
1 2 3 0
c nechceme riesenie 1 -2 -3
-1 2 3 0
```
```
$ minisat party.txt party.out
...
SATISFIABLE
$ cat party.out
SAT
1 2 -3 0
```

Ak to zopakujeme ešte raz, nenájdeme už žiadne riešenie:
```
p cnf 3 6
-1 -3 0
-2 1 0
-3 2 0
1 2 3 0
c nechceme riesenie 1 -2 -3
-1 2 3 0
c nechceme riesenie 1 2 -3
-1 -2 3 0
```
```
$ minisat party.txt party.out
...
UNSATISFIABLE
$ cat party.out
UNSAT
```

#### Dokazovanie ####

Vidíme, že v obidvoch riešeniach sme mohli pozvať Kim. Mali by sme teda byť
schopní **dokázať**, že z našich predpokladov (vstupné štyri tvrdenia)
**vyplýva**, že Kim pôjde na párty. So SAT solverom to môžeme spraviť tak, že
sa ho spýtame, či existuje možnosť, že naše predpoklady platia, ale Kim na
párty nepojde (t.j. negácia tvrdenia, ktoré chceme dokázať). Ak SAT solver
riešenie nájde, tak nám našiel „protipríklad“: predpoklady platia, ale naše
tvrdenie nie. Ak SAT solver riešenie nenájde, tak to znamená, že vždy, keď
platili predpoklady, tak platilo aj naše tvrdenie.

```
p cnf 3 4
c nasa "teoria"
-1 -3 0
-2 1 0
-3 2 0
1 2 3 0

c chceme dokazat cielove tvrdenie "kim"
c zapiseme jeho negaciu "¬kim"
-1 0
```
```
$ minisat party.txt party.out
...
UNSATISFIABLE
$ cat party.out
UNSAT
```

*Poznámka: Je dobré vždy najskôr napísať len „teóriu“ a overiť si, či je
„korektná“, t.j.  či má riešenie, a až potom pridať negáciu dokazovaného
tvrdenia.* Otázka: Je problém, že keď máme „spornú“ teóriu, tak SAT solver
vlastne povie, že z nej vyplýva čokoľvek?

2. Russian spy puzzle (3b)
--------------------------
(Andrei Voronkov, http://www.voronkov.com/lics.cgi)

> There are three persons: Stirlitz, Muller, and
> Eismann. It is known that exactly one of them is
> Russian, while the other two are Germans.
> Moreover, every Russian must be a spy.
> 
> When Stirlitz meets Muller in a corridor, he
> makes the following joke: "you know, Muller,
> you are as German as I am Russian". It is
> known that Stirlitz always tells the truth when
> he is joking.
> 
> We have to establish that Eismann is not a Russian spy.

Prvá úloha pri zápise nejakého problému v logike je vždy vymyslieť,
ako budeme reprezentovať jednotlivé objekty, vlastnosti, vzťahy. Musíme si dať
pozor, aby naša reprezentácia nepripúšťala nejaké nečakané možnosti („Eisman nie
je Rus ani Nemec“, „Eisman je zároveň Rus aj Nemec“), ale zároveň tiež, aby
nepredpokladala niečo, čo nemusí byť zrejmé (a teda nemusí byť pravda) priamo zo
zadania (byť špiónom nie je to isté, čo byť Rusom: dôkaz tejto úlohy by to
zrovna nemalo ovplyvniť, ale to si nemôžeme byť vopred istí).

Na ukážku budeme používať tri premenné pre každého z ľudí, ktoré hovoria, či je
dotyčný Rus, Nemec, respektíve špión (keďže zo zadania je zrejmé, že každý je
*buď Nemec alebo Rus*, tak by nám mohla stačiť aj iba jedna premenná namiesto
týchto dvoch).

|    |   |                   |    |   |                 |    |   |                 |
|----|---|-------------------|----|---|-----------------|----|---|-----------------|
| RS | 1 | Stirlitz je Rus   | RM | 2 | Müller je Rus   | RE | 3 | Eisman je Rus   |
| GS | 4 | Stirlitz je Nemec | GM | 5 | Müller je Nemec | GE | 6 | Eisman je Nemec |
| SS | 7 | Stirlitz je špión | SM | 8 | Müller je špión | SE | 9 | Eisman je špión |

Táto reprezentácia umožňuje, aby niekto z nich nebol ani Nemec, ani Rus, alebo
bol oboje. Ako prvé teda napíšeme formule, ktoré zabezpečia, že každý z nich je
buď Nemec, alebo Rus, ale nie oboje. Jedna z možností je napríklad tvrdenie
„X je Rus práve vtedy, keď X nie je Nemec“ čo zapíšeme ako `RX ⇔ ¬GX` (inými
slovami: povedali sme, že RX je to isté čo ¬GX, a ak zvolíme či platí jedno,
určili sme aj druhé).

Teraz môžeme zapísať všetky podmienky zo zadania:

1. X je buď Rus alebo špión: `(RS ⇔ ¬GS) ∧ (RM ⇔ ¬GM) ∧ (RE ⇔ ¬GE)`
2. práve jeden je Rus, ostatní dvaja sú nemci:
  `(RS ∧ GM ∧ GE) ∨ (GS ∧ RM ∧ GE) ∨ (GS ∧ GM ∧ RE)`
3. každý Rus musí byť špión:
  `(RS → SS) ∧ (RM → SM) ∧ (RE → SE)`
4. Stirlitz: „Müller, ty si taký Nemec, ako som ja Rus“:
  `GM ⇔ RS`

Samozrejme, aby sme vyrobili vstup pre SAT solver, musí všetky formuly upraviť
do konjunktívnej normálnej formy:
* Ekvivalencie sa len rozpíšu ako dve implikácie (spojené konjunkciou).
* Implikácie prepíšeme pomocou vzťahu `(a → b) ⇔ (¬a ∨ b)`
* Formulu z bodu 2 musíme „roznásobiť“, čím nám vznikne 27 disjunkcií s tromi
  premennými (z každej zátvorky jedna).

```
c X je buď Rus alebo špión
¬RS ∨ ¬GS
GS ∨ RS
¬RM ∨ ¬GM
GM ∨ RM
¬RE ∨ ¬GE
GE ∨ RE

c práve jeden je Rus, ostatní dvaja sú Nemci
RS ∨ GS ∨ GS
RS ∨ GS ∨ GM
RS ∨ GS ∨ RE

RS ∨ RM ∨ RM
RS ∨ RM ∨ GM
RS ∨ RM ∨ RE

GE ∨ GE ∨ RM
GE ∨ GE ∨ GM
GE ∨ GE ∨ RE

GM ∨ GS ∨ GS
GM ∨ GS ∨ GM
GM ∨ GS ∨ RE

GM ∨ RM ∨ RM
GM ∨ RM ∨ GM
GM ∨ RM ∨ RE

GM ∨ GE ∨ RM
GM ∨ GE ∨ GM
GM ∨ GE ∨ RE

GE ∨ GS ∨ GS
GE ∨ GS ∨ GM
GE ∨ GS ∨ RE

GE ∨ RM ∨ RM
GE ∨ RM ∨ GM
GE ∨ RM ∨ RE

GE ∨ GE ∨ RM
GE ∨ GE ∨ GM
GE ∨ GE ∨ RE

c  každý Rus musí byť špión
¬RS ∨ SS
¬RM ∨ SM
¬RE ∨ SE

c Stirlitz: „Müller, ty si taký Nemec, ako som ja Rus“
¬GM ∨ RS
¬RS ∨ GM
```

Teraz už stačí len zmeniť všetky premenné na čísla (search&replace je váš
priateľ :-), negácie na mínus, zmeniť disjunkcie na nulou ukončené postupnosti
čísel a môžeme pustiť SAT solver.

SAT solver by teraz našiel možné riešenie, ako by to mohlo byť. Môžete použiť
podobný postup ako v úlohe 1 a získať všetky možnosti (malo by ich byť 8).

Na vyriešenie úlohy ale potrebujeme dokázať, že *Eisman nie je ruský špión*.
Potrebujeme teda ešte zapísať toto tvrdenie, znegovať ho, previesť do CNF
a pridať k ostatným. Ak SAT solver povie, že formula je už nesplniteľná,
negácia nemôže platiť spolu s predpokladmi, a teda musí určite platiť pôvodné,
nenegované tvrdenie.
