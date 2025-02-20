# Pointery, lístočky na chladničke a turistické značky

Tak ako v každej rozprávke nájdeme nielen šarmantné princezné, ale aj nešarmantných drakov, tak v jazyku C (C++) okrem krásneho kódu narazíme aj na škaredé pointery. Pri prvom stretnutí s nimi častokrát nechápeme, čo sa deje, a náš program elegantne spadne do priepasti segmentation faults. No čím viac nocí nad nimi preplačeme, tým viac zistíme, že tieto škaredé káčatká sú v skutočnosti krásne labute. A aby tých bezsenných nocí bolo čo najmenej, pripravili sme pre vás tento článok, v ktorom vám pointery predstavíme tak trochu inak.

## Hello world, pointery
Vaše prvé stretnutie s pointermi mohlo vyzerať nejako takto:
```c
int num = 45;
int *ptr = &num; // pointer na num, '*' je označenie pointera
```
K tomu pravdepodobne nasledoval takýto popis: 
*Premenná `ptr` je špecifická tým, že neuchováva priamo hodnotu, ale adresu pamäťového miesta, kde sa táto hodnota nachádza. Operátor `&` vracia adresu premennej `num`, zatiaľ čo operátor `*` označuje, že `ptr` je pointer. Tento pointer umožňuje manipulovať s obsahom premennej `num` prostredníctvom jej adresy v pamäti. To dosiahneme opätovným použitím operátora `*` na premennú `ptr`. Tentoraz však `*` neslúži na označenie pointera, ale na jeho **dereferencovanie** – získanie hodnoty uloženej na danej adrese.*

```c
*ptr = 32;   // teraz je hodnota num = 32, '*'je hodnota pointera
print(*ptr);
```

Ale počkať, naskytá sa tu prirodzená otázka: *Načo nám je nejaká divná premenná `ptr`, keď môžeme hodnotu premennej `num` zmeniť priamo?*
```c
num = 32;
```
*Why you have to go and make things so complicated?“* - ako spieva kanadská speváčka Avril Lavigne. Odpoveď na túto zákernú otázku si vyžaduje hlbšie pochopenie toho, čo je to pamäť a ako vlastne funguje. Preto sa musíme vydať na dlhú cestu do jej hlbín.

## Vitajte v pamäti, priatelia  

Priatelia, vitajte v pamäti! Tu sa všetko odohráva, tu sa všetko deje. V prvom rade nám ale dovoľte túto pamäť predstaviť jej pravým menom, a to: **operačná pamäť**. Väčšina z vás ju však pozná skôr ako pamäť RAM (Random Access Memory), teda pamäť, ktorá umožňuje prístup na ľubovoľnú jej pozíciu v konštantnom čase. Dôležité je ale pochopiť, že *RAM* nie je názov konkrétnej pamäte, ale iba spôsob ukladania dát. Používať označenie *RAM* je to isté ako povedať, že jazdíme v *benzíne* namiesto toho, že jazdíme v *aute*.

Dobre teda, ako si túto *operačnú pamäť* môžeme bližšie predstaviť? V podstate ide o obrovskú tabuľku, kde každá bunka predstavuje práve jeden bajt. Do tejto tabuľky sa ukladajú všetky možné aj nemožné premenné, ktoré sa v programe vytvoria. Skúsme si teraz zjednodušiť situáciu a predstavme si model 16-bajtovej pamäte a pozrime sa, ako sa do nej budú ukladať jednotlivé premenné. Začnime s jednoduchými dátovými typmi ako sú `int`, `double` a `char`:

```c
int x = 3;          // int = 4B
double y = 12.5;    // double = 8B
char z = 'A';       // char = 1B
```
Rozloženie v pamäti:
|Pozícia| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|:-----:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|
|Dáta| x | x | x | x | y | y | y | y | y | y |  y |  y |  z |    |    |    |

Vidíme, že každá premenná zaberá v pamäti určitý počet bajtov, ktoré sú vedľa seba naukladané. Štandartne `int` má 4 bajty, `double` 8 bajtov a `char` 1 bajt. Tento princíp platí aj pri zložitejších dátových typoch, ako sú polia a štruktúry. Skúsme sa najprv pozrieť na polia:

```c
int arr[3] = {1, 2, 3};
```

Rozloženie v pamäti:
|Pozícia| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|:-----:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|
|Dáta| arr[0] | arr[0] | arr[0] | arr[0] | arr[1] | arr[1] | arr[1] | arr[1] | arr[2] | arr[2] |  arr[2] | arr[2] |    |    |    |    |

Tak ako pri jednoduchých dátových typoch, aj pri poliach vidíme rovnaký vzor organizácie dát: najskôr sa uložia bajty prvého prvku, potom bajty druhého prvku a tak ďalej až po posledný. Z pohľadu pamäte sa teda nič nemení. Teraz sa pozrime na štruktúry:

```c
typedef struct
{ 
    int n; // n/d 
    int d;

} fraction_t;

fraction_t f = {2, 3}; // 2/3
```

Rozloženie v pamäti:
|Pozícia| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|:-----:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|
|Dáta| f.n | f.n | f.n | f.n | f.d | f.d | f.d | f.d |  |  |   |   |   |    |    |    |

Tak ako pri poliach, aj pri štruktúrach sa jednotlivé prvky ukladajú rovno za sebou. Z pohľadu pamäte teda neexistujú "samostatné premenné" – pamäť vníma len súvislý rad bajtov, na ktorých sú uložené dáta. Zjednodušene povedané: **pamäť je jedno veľké pole bajtov**. V našom modeli má toto pole iba 16 bajtov, no v reálnych operačných pamätiach sa bavíme o gigabajtoch – teda miliardách takýchto políčok!

A ako sa v takej obrovskej pamäti vyznať? Ako zistiť, kde sú uložené jednotlivé premenné, keď ide o obrovské, takmer nekonečné pole bajtov? Keďže tento článok je tak trochu o pointeroch, tak skúsme to práve s nimi. 😀

## Pointery a cesta z Daidalovho labyrintu

Daidalov labyrint, pochádzajúci z gréckych bájí, je jedným z najznámejších symbolov zložitosti a bezvýchodiskovosti. Podľa legendy ho postavil geniálny remeselník Daidalos pre kráľa Minoa. Jeho účel bol v tom,  aby uväznil obávaného Minotaura – napoly človeka, napoly býka. Labyrint bol taký zložitý, že prakticky z neho neexistovala cesta von, a ktokoľvek doň vstúpil, už nikdy nenašiel východ. A tu prichádza na scénu Ariadna, krétska princezná, ktorá pomohla hrdinovi Théseovi prekonať tento labyrint. Darovala mu niť, ktorú pri vstupe priviazal a postupne odvíjal. Keď porazil Minotaura, stačilo mu sledovať niť a bezpečne sa tak rátiť na začiatok. Tento jednoduchý trik premenil beznádejný chaos labyrintu na prehľadnú a zvládnuteľnú štruktúru.

Presne takto fungujú v programovaní pointery. Pamäť môžeme vnímať ako obrovský labyrint bajtov, kde neexistuje žiadne pevné usporiadanie, iba surové údaje roztrúsené v priestore. Bez jasných orientačných bodov by sme sa v nej rýchlo stratili. Pointery sú našou **Ariadninou niťou** – ukazujú na konkrétne miesta v pamäti, umožňujú nám sledovať vzťahy medzi údajmi a efektívne sa pohybovať v tomto zdanlivo chaotickom priestore. Bez nich by sme len blúdili naslepo, dúfajúc, že náhodou trafíme správnu hodnotu.

### Takže čo je to vlastne pointer?

Vráťme sa na začiatok a definujme si, čo to vlastne pointer je. Pointer je jednoducho premenná, ktorá uchováva celé číslo reprezentujúce pozíciu v pamäti. Štandardne má pointer veľkosť 8B, aby pokryl všetky možné adresy súčasných 64-bitových systémov; na starších 32-bitových systémoch sa môžeme stretnúť aj s veľkosťou 4B. Ak si zoberieme náš prvý príklad a premietneme ho na tabuľkový model pamäte, vyzeralo by to asi takto:

```c
int num = 45;
int *ptr = &num; // hodnota ptr je 0x000002266bf35fb0
```

Rozloženie v pamäti:
|Pozícia| ... | 0x000002266bf35fb0 | 0x000002266bf35fb0 + 1 | 0x000002266bf35fb0 + 2 | 0x000002266bf35fb0 + 3 | ... |
|:-----:|:-:|:-:|:-:|:-:|:-:|:-:|
|Dáta| ... | num | num | num | num | ... |

Teraz sa nám odhalila pravá tvár pointera: Môžeme vidieť, že premenná `ptr` v podstate predstavuje prvý riadok našej tabuľky. Čiže **hodnota pointera je poradové číslo prvého bajtu v pamäti, kde sú uložené dáta.** Len to nie je jednoduché číslo ako napríklad `10`, ale je to obrovské číslo, dokonca zapísané v šestnástkovej sústave, napríklad `0x000002266bf35fb0`.

Ak si chceme pozrieť túto hodnotu pointera, môžeme použiť nasledovný príkaz:

```c
printf("%p", ptr);
```

Dôležité je však uvedomiť si, že pointer nie je samostatný dátový typ – je to typ, ktorý sa vždy viaže na iný dátový typ (okrem `void*`, ale o tom potom). To znamená, že vždy musíme špecifikovať aj to, na aký dátový typ náš pointer ukazuje:
```c
// *p - chyba, nie je špecifikovaný dátový typ
int *p1; // pointer na int
double *p2; // pointer na double
char *p3; // pointer na char
```

Prečo teda pointer nemôže byť samostatný dátový typ, keď vždy obsahuje 8-bajtové (alebo 4-bajtové) číslo? Nuž, tak ako peniaze nie sú všetko, tak ani adresa nie je všetko. Keď sa pozrieme vyššie na náš tabuľkový model, vidíme, že premenné zaberajú rôzny počet bajtov a dokonca môžu mať aj rôzne kódovanie. Napríklad `int` a `float` zaberajú oba tradične 4 bajty, ale majú diametrálne odlišné kódovanie. Aby sme to zhrnuli, pointer musí mať špecifikované tri kľúčové vlastnosti, aby bol plne funkčný – teda aby vedel čítať a zapisovať údaje do pamäte (premenných):
* adresu prvého bajtu,
* počet bajtov, na ktorý ukazuje,
* kódovanie dát.

```c
// int = počet bajtov = 4, kódovanie = 'signed integer'
// &a = adresa prvého bajtu premennej a
int *p = &a;
```

Prvá vlastnosť je obsiahnutá v jeho hodnote a zvyšné dve sú skryté v jeho type.

### Polia sú v skutočnosti pointery
Keď už sme sa ako-tak zoznámili s tým, že pointer má podobu čísla, je načase odkryť jeho druhú tvár – tvár poľa:
Každé pole, či už statické alebo dynamické, je v skutočnosti **pointer na jeho prvý prvok v pamäti**. A tajomstvo je odhalené (dramatická hudba):

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", *ptr); // 1
```

Čiže si to zhrňme, dátová štruktúra `array` je v jazyku C implementovaná ako **pointer na blok pamäte**. To znamená, že existuje medzi pointermi a poľami určitá dualita. Ale ešte je tu jedna vec: aby sme mohli s poľami narábať, musíme ich indexovať. Čo majú pointery?

Pre pointery existuje niečo, ako **pointerová aritmetika**, ktorá opisuje dve ďalšie možnosti, s ktorými sa dá s pointermi pracovať:

* Pripočítanie / odpočítanie celého čísla – posunutie pointera o jeden dátový typ. Napríklad operácia `ptr + 1` znamená posunutie pointera o jeden `int`, čiže o 4 bajty. Keby bol `ptr` typ `double`, tak sa pointer posunie o jeden `double` (8B).
* Odčítanie dvoch pointerov – určenie vzdialenosti medzi nimi: `size_t d = (ptr2 - ptr1)` 

Potom indexovanie poľa bude vyzerať takto nejako:

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", *ptr);       // 1
printf("%d", *(ptr + 1)); // 2
printf("%d", *(ptr + 2)); // 3
```

Keďže operácia posunu a dereferencie `*(ptr + i)` je pri pointeroch veľmi používaná, autori jazyka C vymysleli nasledovné zjednodušenie (takzvaný syntaktický cukor): `*(ptr + i) = ptr[i]`. Takže indexovanie poľa je vlastne nič iné, iba posun pointera a jeho následná dereferencia? No wow! Teraz dáva perfektný zmysel aj to, prečo polia číslujeme od nuly. ``arr[0]`` v podstate znamená *„daj mi prvok na adrese s nulovým posunom“*.

Potom môžeme náš kód výrazne zjednodušiť:

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", ptr[0]); // 1
printf("%d", ptr[1]); // 2
printf("%d", ptr[2]); // 3
```

**Aby sme to zhrnuli: pointery sú polia a smerníková aritmetika spôsobuje ich indexovanie.**

💡 Zaujímavosť: konštrukcia `*(ptr + i) = ptr[i]` nekontroluje dátové typy, a keďže plus je komutatívna operácia, daný výraz možno zapísať aj ako: `*(i + ptr) = i[ptr]`. Potom by náš príklad do tretice vyzeral takto:

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", 0[ptr]); // 1
printf("%d", 1[ptr]); // 2
printf("%d", 2[ptr]); // 3
```

Síce tento kód je funkčný, ale prosím nerobte to, je to nebezpečné. 😃

### Pointery sú vstupno-výstupné parametre
Ak ste si mysleli, že pointery majú iba dve tváre, tak tu máte tretiu, a to je ich vzťah s funkciami. Pri funkciách poznáme niečo ako **prenášanie hodnotou vs. prenášanie odkazom**, čo si pozrieme v nasledujúcom príklade:

```c
void scan_arr(int arr[], int len)
{
    for (int i = 0; i < len; ++i)
        scanf("%d", &arr[i]);
}
```

Funkcia `scan_arr` má dva parametre: **pole `arr` a jeho dĺžku `len`**. Medzi nimi je však zásadný rozdiel (vieme už že pole je vlastne pointer). Keď sa funkcia spustí, pod kapotou si zo svojich parametrov vytvorí svoje vlastné kópie s ktorými pracuje. Takže vo vnútri `scan_arr` vzniknú premenné `arr2` a `len2`.

- **Premenná `len2` nemá okrem rovnakej hodnoty žiadne spojenie s pôvodnou `len`**, čiže keď ju funkcia zmení, nepremietne sa to vo zvyšnom programe. Tomuto hovoríme prenášanie hodnotou.
- **Premenná `arr2` je však kópia pointera `arr`**, čo znamená, že síce `arr2` a `arr'` sú síce dve rôzne premenné, ale stále ukazujú na to isté pole (resp. pamäť) a môžu ho meniť. Toto je prenášanie pointerom. Pri poliach sa používa natívne, pretože to má aj ekonomický dôvod: **namiesto kopírovania desiatok či stoviek bajtov stačí skopírovať len 8 bajtov!**

Prenášanie pointerom je niečo ako lístoček na chladničke. Keď vám niekto nechá lístoček na chladničke s popisom, kde je večera, tak viete si tú večeru nájsť a narábať s ňou (zjesť ju). Analogicky to funguje aj pri funkciách. Čiže pointer keď vlezie do funkcie, stáva sa  **vstupno-výstupným parametrom**. Upravme si danú funkciu, aby načítala nie len pole ale aj jeho dĺžku a vrátila výsledok cez parametre:

```c
void scan_arr(int *arr, int *len)   // arr[] je to isté ako *arr
{
    scanf("%d", len);

    for (int i = 0; i < *len; ++i)
        scanf("%d", &arr[i]);
}
```

Teraz môžeme vidieť, že aj `arr` aj `len` sú pointery, čiže je možné ich hodnoty vo funkcii meniť. Aké pôvabné. 