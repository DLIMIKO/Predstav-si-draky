# Pointery, lístočky na chladničke a turistické značky

Tak ako v každej rozprávke nájdete nielen šarmantné princezné, ale aj nešarmantných drakov, tak aj v jazyku C (C++) okrem krásneho kódu narazíte na škaredé pointery. Pri prvom stretnutí s nimi sa možno budete cítiť ako pri rozhovore s drakom – nepochopíte, čo sa deje, a váš kód sa vám odmení elegantným pádom do priepasti segmentation faults. No čím viac nocí nad nimi preplačete, tým viac pochopíte, že tieto škaredé káčatká sú v skutočnosti krásne labute. A aby tých bezsenných nocí bolo čo najmenej, pripravili sme pre vás tento článok, v ktorom vám pointery predstavíme tak trochu inak.

## Hello world, pointery
Vaše prvé stretnutie s pointermi mohlo vyzerať takto nejako:
```c
int num = 45;
int *ptr = &num; 
```
K tomu pravdepodobne nasledoval takýto popis: <i>Premenná ```ptr``` je špecifická v tom, že neuchováva priamo hodnotu, ale adresu pamäťového miesta, kde sa táto hodnota nachádza. Tento ukazovateľ (pointer) umožňuje manipulovať s obsahom premennej ```num``` prostredníctvom jej adresy v pamäti</i>:
```c
*ptr = 32; // teraz je hodnota num = 32
```

Ale počkať! Načo mi je nejaká divná premenná ptr, keď môžem jednoducho priamo zmeniť hodnotu num?
```c
num = 32;
```

[„Why you have to go and make things so complicated?“](https://www.youtube.com/watch?v=5NPBIwQyPWE) – ako spieva Avril Lavigne. Aby sme pochopili skutočný význam pointerov, musíme sa na chvíľu vydať na cestu do hlbín pamäte.

## Vitajte v pamäti, priatelia

Vážení prítomní, vitajte v pamäti! Tu sa všetko odohráva, tu sa všetko deje. V prvom rade nám ale dovoľte túto pamäť predstaviť jej pravým menom a to operačná pamäť. Väčšina z vás ju však pozná skôr ako RAM (Random Access Memory), teda pamäť, ktorá umožňuje ľubovoľný prístup k dátam. Dôležité je pochopiť, že RAM nie je názov konkrétnej pamäte, ale spôsob ukladania dát. Používať označenie RAM je ako povedať, že jazdíme v benzíne namiesto toho, že jazdíme v aute.

Ako si túto operačnú pamäť vlastne môžeme predstaviť? V podstate ide o obrovskú tabuľku, kde každá bunka predstavuje práve jeden bajt. Do tejto tabuľky sa ukladajú všetky premenné, ktoré si v programe vytvoríte. Poďme si teda predstaviť zjednodušený model 16-bajtovej pamäte a pozrime sa, ako budú premenné uložené v pamäti. Začnime s jednoduchými premennými ako sú `int`, `double` alebo `char`:


```c
int x = 3;          // int = 4B
double y = 12.5;    // double = 8B
char z = 'A';       // char = 1B
```
Ich rozloženie v pamäti bude nasledovné:
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|
| x | x | x | x | y | y | y | y | y | y |  y |  y |  z |    |    |    |

Vidíme, že obsah jednotlivých premenných sa ukladá po jednotlivých bajtoch za sebou. Všimnime si jednu zvláštnosť: aj obyčajný `int` je v pamäti uložený ako bajtové pole, pretože zaberá viac ako jeden bajt (v tomto prípade 4). Tento jav môžeme pozorovať aj pri zložených dátových typoch, ako sú polia a štruktúry. Skúsme najprv polia:

```c
int arr[3] = {1, 2, 3, 4};
```

Rozloženie v pamäti:
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|
| arr[0] | arr[0] | arr[0] | arr[0] | arr[1] | arr[1] | arr[1] | arr[1] | arr[2] | arr[2] |  arr[2] | arr[2] |    |    |    |    |

Opäť nič nezvyčajné. Prvky resp. bajty poľa arr sa ukladajú presne za sebou, rovnako ako premenné v predchádzajúcom príklade. Čiže z pohľadu pamäte žiadna zmena. Ešte sa pozrime na štruktúry:

```c
typedef struct
{ 
    int n; // n/d 
    int d;

} fraction_t;

fraction_t f = {2, 3}; // 2/3
```

Rozloženie v pamäti:
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|
| f.n | f.n | f.n | f.n | f.d | f.d | f.d | f.d |  |  |   |   |   |    |    |    |

Tak ako pri poliach, aj pri štruktúrach sa bajty jednotlivých prvkov ukladajú rovno za sebou. Z pohľadu pamäti teda neexistujú samostatné "premenné" v tom zmysle, ako si ich zvyčajne predstavujeme. Pamäť vidí iba naukladané bajty, do ktorých je niečo vložené. Zjednodušene povedané: <b>pamäť je niečo ako pole bajtov</b>. V našom prípade toto pole má iba 15 bajtov. Ale reálne operačné pamäte majú kapacity rádovo v gigabajtoch, pričom jeden gigabajt je jedna miliarda takýchto políčok!

A ako sa potom v tejto obrovskej pamäti vyznať? Ako zistiť, kde sú uložené jednotlivé premenné, keď ide o akési skoro-nekonečné pole bajtov? Odpoveď na túto otázku je jasná, keďže tento článok je o pointeroch – môžete trikrát hádať, čo nám pomáha s orientáciou v pamäti. Aké pôvabné 😀.

## Daidalov labyrint a použitie pointerov

Daidalov labyrint, pochádzajúci z gréckych bájí, je jedným z najznámejších symbolov zložitosti a bezvýchodiskovosti. Podľa legendy ho postavil geniálny remeselník Daidalos pre kráľa Minoa, aby v ňom uväznil Minotaura – napoly človeka, napoly býka. Labyrint bol taký zložitý, že z neho neexistovala cesta von, a ktokoľvek doň vstúpil, už nikdy nenašiel východ. A tu prichádza na scénu Ariadna, krétska princezná, ktorá pomohla hrdinovi Théseovi prekonať labyrint. Darovala mu niť, ktorú pri vstupe priviazal a postupne odvíjal. Keď porazil Minotaura, stačilo mu sledovať niť a bezpečne sa vrátil na začiatok. Tento jednoduchý trik premenil beznádejný chaos labyrintu na prehľadnú a zvládnuteľnú štruktúru.

Presne takto fungujú v programovaní pointery. Pamäť môžeme vnímať ako obrovský labyrint bajtov, kde neexistuje žiadne pevné usporiadanie, iba surové údaje roztrúsené v priestore. Bez jasných orientačných bodov by sme sa v nej rýchlo stratili. Pointery sú našou **Ariadninou niťou** – ukazujú na konkrétne miesta v pamäti, umožňujú nám sledovať vzťahy medzi údajmi a efektívne sa pohybovať v tomto zdanlivo chaotickom priestore. Bez nich by sme len blúdili naslepo, dúfajúc, že náhodou trafíme správnu hodnotu.

### Čo je to pointer?

Takže vráťme sa teraz na začiatok a definujme si, čo je to vlastne pointer. Pointer je nič iné ako celé číslo, ktoré ukazuje pozíciu v pamäti. Štandardne má pointer veľkosť **8B**, aby pokryl všetky možné adresy. Ak si zoberieme prvý príklad a premietneme ho na náš tabuľkový model, vyzeralo by to asi takto:

```c
int num = 45;
int *ptr = &num; 
```

Rozloženie v pamäti:

| ... | 0x000002266bf35fb0 | 0x000002266bf35fb0 + 1 | 0x000002266bf35fb0 + 2 | 0x000002266bf35fb0 + 3 | ... |
|:-:|:-:|:-:|:-:|:-:|:-:|
| ... | num | num | num | num | ... |

Ako môžeme vidieť, `ptr` predstavuje prvý riadok tabuľky a `num` druhý. Čiže označenie bajtu je vlastne hodnota pointera? V podstate áno. **Pointer je poradové číslo prvého bajtu premennej, na ktorú ukazuje.** Len to nie je číslo 10, ale v **16-tkovej sústave** napríklad `0x000002266bf35fb0`.

Ak si chcete pozrieť skutočnú hodnotu pointera, použite nasledovný príkaz:

```c
printf("%p", ptr);
```

### Pointery a polia

Poďme ešte ďalej a pozrime sa na tajný vzťah medzi pointermi a poľami. Tajomstvo je v tom, že názov poľa je v skutočnosti **pointer na prvý bajt prvého prvku**:

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", *ptr); // 1
```

V jazyku C existuje medzi pointermi a poľami určitá dualita. Pozrime si ďalšiu vec – **pointerovú aritmetiku**. K pointeru možno pričítať alebo odčítať ľubovoľné celé číslo, čo znamená, že sa posunieme v pamäti o počet bajtov, na ktoré ukazuje:

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", *ptr);       // 1
printf("%d", *(ptr + 1)); // 2
printf("%d", *(ptr + 2)); // 3
```

Keďže operácia `*(ptr + i)` je bežná, autori jazyka C vymysleli syntaktický cukor: `*(ptr + i) = ptr[i]`.

No wow! Teraz dáva perfektný zmysel aj to, prečo polia číslujeme od nuly. **`arr[0]` znamená „daj mi prvok na adrese s nulovým posunom“**:

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", ptr[0]); // 1
printf("%d", ptr[1]); // 2
printf("%d", ptr[2]); // 3
```

### Prečo sú pointery dôležité?

Pointery sú síce užitočné, ale prečo s nimi musí pracovať aj samotný programátor? Poďme sa pozrieť na rozdiel medzi **prenášaním hodnotou vs. prenášaním odkazom**. Uvažujme nasledujúci príklad:

```c
void scan_arr(int arr[], int len)
{
    for (int i = 0; i < len; ++i)
        scanf("%d", &arr[i]);
}
```

Funkcia `scan_arr` má dva parametre: **pole `arr` a jeho dĺžku `len`**. Medzi nimi je však zásadný rozdiel. Keď sa funkcia spustí, pod kapotou si zo svojich parametrov vytvorí kópie, takže vo vnútri `scan_arr` vzniknú premenné `arr'` a `len'`.

- **Premenná `len'` nemá žiadne spojenie s pôvodnou `len`**, čiže keď ju funkcia zmení, nepremietne sa to vo zvyšnom programe.
- **Premenná `arr'` je však kópia pointera `arr`**, čo znamená, že síce `arr` a `arr'` sú síce dve rôzne premenné, ale stále ukazujú na to isté pole (resp. pamäť).

💡 **Namiesto kopírovania desiatok či stoviek bajtov stačí skopírovať len 8 bajtov!**

A presne toto je ten čistý pôvab pointerov – elegantné a efektívne riešenie, ktoré šetrí pamäť aj výkon. 😃
