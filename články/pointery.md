# Pointery, lístočky na chladničke a Minotaurus

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

Vidíme, že každá premenná zaberá v pamäti určitý počet bajtov, ktoré sú vedľa seba naukladané. Štandartne `int` má 4 bajty, `double` 8 bajtov a `char` 1 bajt. Tento princíp platí aj pri zložitejších dátových typoch, ako sú polia a štruktúry. Skúsme sa najprv pozrieť na polia (ale nie také, po ktorých behajú traktory):

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

} fraction_t; // štruktúra zlomok

fraction_t f = {2, 3}; // 2/3
```

Rozloženie v pamäti:
|Pozícia| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|:-----:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|
|Dáta| f.n | f.n | f.n | f.n | f.d | f.d | f.d | f.d |  |  |   |   |   |    |    |    |

Tak ako pri poliach, aj pri štruktúrach sa jednotlivé prvky ukladajú rovno za sebou. Z pohľadu pamäte teda neexistujú "samostatné premenné" – pamäť vníma len súvislý rad bajtov, na ktorých sú uložené dáta. Zjednodušene povedané: **pamäť je jedno veľké pole bajtov**. V našom modeli má toto pole iba 16 bajtov, no v reálnych operačných pamätiach sa bavíme o gigabajtoch – teda miliardách takýchto políčok!

A ako sa v takej obrovskej pamäti vyznať? Ako zistiť, kde sú uložené jednotlivé premenné, keď ide o obrovské, takmer nekonečné pole bajtov? Keďže tento článok je tak trochu o pointeroch, tak skúsme to práve s nimi. 😀

## Pointery a cesta z Daidalovho labyrintu

Daidalov labyrint, pochádzajúci z gréckych bájí, je jedným z najznámejších symbolov zložitosti a bezvýchodiskovosti. Podľa legendy ho postavil geniálny remeselník Daidalos pre kráľa Minoa. Jeho účel bol v tom,  aby uväznil obávaného Minotaura – napoly človeka, napoly býka. Labyrint bol taký zložitý, že prakticky z neho neexistovala cesta von, a ktokoľvek doň vstúpil, už nikdy nenašiel východ. A tu prichádza na scénu Ariadna, krétska princezná, ktorá pomohla hrdinovi Théseovi prekonať tento labyrint. Darovala mu niť, ktorú pri vstupe priviazal a postupne odvíjal. Keď porazil Minotaura, stačilo mu sledovať niť a bezpečne sa tak vrátiť na začiatok. Tento jednoduchý trik premenil beznádejný chaos labyrintu na prehľadnú a zvládnuteľnú štruktúru.

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

💡 Zaujímavosť: konštrukcia `*(ptr + i) = ptr[i]` nekontroluje dátové typy, a keďže plus je komutatívna operácia, daný výraz možno zapísať aj ako: `*(i + ptr) = i[ptr]`. Potom by náš príklad do tretice vyzeral takto:

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", 0[ptr]); // 1
printf("%d", 1[ptr]); // 2
printf("%d", 2[ptr]); // 3
```

Síce tento kód je funkčný, ale prosím nerobte to, je to nezdravé. 😃

**Aby sme to zhrnuli: pointery sú polia a smerníková aritmetika spôsobuje ich indexovanie.**

Hej! To isté platí aj pre dynamické polia, kde je tento mechanizmus explicitnejší. Funkcia `malloc()` v C, alebo operátor `new` v C++ si vypýtajú od operačného systému blok pamäte, ktorého začiatočná adresa sa uloží do pointera lokálneho pointera. Na rozdiel od statických poli je nutné alokovanú pamäť po skončení operácií manuálne uvoľniť.

*Dôležité upozornenie: pri `malloc()` treba špecifikovať počet bajtov, nie počet prvkov. Preto je vhodné použiť `sizeof(int)`, aby sa zabránilo nesprávnemu prideľovaniu pamäte. Zároveň je vhodné výsledok pretypovať na `int*`.*

```c
int *arr = (int*)malloc(10 * sizeof(int)) // 10 int prvkov
arr[0] = 1;
arr[1] = 2;
arr[2] = 3;

free(arr); // nezabudnime uvoľniť
```

Môžeme vidieť, že práca s dynamickými poľami je na 99 % rovnaká ako pri poliach statických.
Hlavný rozdiel medzi statickými a dynamickými poliami je v spôsobe ich správy. Statické polia majú pevnú veľkosť určenú pri kompilácii, kým dynamické polia možno alokovať a meniť za behu programu. Navyše, zatiaľ čo statické polia sú implicitne (skryto) spravované a automaticky uvoľňované kompilátorom, dynamické polia vyžadujú manuálne uvoľnenie pamäte programátorom. Žiaľ, programátorom sme my. Keby bol niekto zvedavý, tu je prehľadová tabuľka, ktorá popisuje rozdiel medzi jednotlivými poľami a lúkami.

| Vlastnosť          | Statické pole                 |            Dynamické pole            |
|--------------------|-------------------------------|--------------------------------------|
| Alokácia           | Kompilátor (stack/static)     | `new` / `malloc()` (heap)            |
| Veľkosť            | Fixná pri kompilácii          | Možno meniť za behu programu         |
| Je to pointer?     | Áno, implicitne (skryto)      | Áno, explicitne (okato)              |
| Manuálne mazanie?  | Nie                           | Áno (`delete[]`)                     |

### Pointery sú vstupno-výstupné parametre
Ak ste si mysleli, že pointery majú iba dve tváre, tak tu máte tretiu - funkčnú tvár. Pri funkciách poznáme niečo ako **prenášanie hodnotou vs. prenášanie odkazom**, čo si pozrieme v nasledujúcom príklade:

```c
void scan_arr(int arr[], int len)
{
    for (int i = 0; i < len; ++i)
        scanf("%d", &arr[i]);
}
```

Funkcia `scan_arr` má dva parametre: **pole `arr` a jeho dĺžku `len`**. Medzi nimi je však zásadný rozdiel (vieme už že pole je vlastne pointer). Keď sa funkcia spustí, pod kapotou si zo svojich parametrov vytvorí svoje vlastné kópie s ktorými pracuje. Takže vo vnútri `scan_arr` vzniknú premenné `arr2` a `len2`.

- **Premenná `len2` nemá okrem rovnakej hodnoty žiadne spojenie s pôvodnou `len`**, čiže keď ju funkcia zmení, nepremietne sa to vo zvyšnom programe. Tomuto hovoríme prenášanie hodnotou.
- **Premenná `arr2` je však kópia pointera `arr`**, čo znamená, že síce `arr2` a `arr` sú síce dve rôzne premenné, ale stále ukazujú na to isté pole (resp. pamäť) a môžu ho meniť. Toto je prenášanie pointerom. Pri poliach sa používa natívne, pretože to má aj ekonomický dôvod: **namiesto kopírovania desiatok či stoviek bajtov stačí skopírovať len 8 bajtov!**

Prenášanie pointerom je niečo ako lístoček na chladničke. Keď vám niekto nechá lístoček na chladničke s popisom, kde je večera, tak viete si tú večeru nájsť a narábať s ňou (zjesť ju). Analogicky to funguje aj pri funkciách. Keď pointer vlezie do funkcie, stáva sa jej  **vstupno-výstupným parametrom**. Upravme si danú funkciu, aby načítala nie len pole ale aj jeho dĺžku a vrátila výsledok cez parametre:

```c
void scan_arr(int *arr, int *len)   // arr[] je to isté ako *arr
{
    scanf("%d", len);

    for (int i = 0; i < *len; ++i)
        scanf("%d", &arr[i]);
}
```

Teraz môžeme vidieť, že aj `arr` aj `len` sú pointery, čiže je možné ich hodnoty vo funkcii meniť. Aké pôvabné. 

## Bratracni a sesternice pointerov

Okrem klasického pointera `*ptr` ich existuje celá rodina. Pointery sú rôznorodé – všetky síce ukazujú do pamäte, ale popritom plnia aj ďalšie špecifické funkcie. V tomto rodostrome sa môžeme ľahko stratiť, preto si ich teraz postupne predstavíme.

### Pointer na `void`

No super! Minule sme si povedali, že každý pointer ukazuje na *nejaký* dátový typ. Ale `void` dátový typ nie je. Používame ho síce pri funkciách, aby sme mali alibi, že funkcia nič nevracia. Je to totiž *fantómový typ*. A presne tak funguje aj `void*`. Skúsme si to predstaviť: `void` pointer uchováva iba adresu nejakých dát, ale neobsahuje informáciu o tom, na koľko bajtov ukazuje a aké kódovanie nesie:
```C
int num = 30;
void *ptr = &num;
```

Takýto pointer môže iba existovať a nedá sa s ním nič iné robiť. Keby slovenská pieseň od skupiny Elán *Voda, čo ma drží nad vodou* bola o `void` pointeroch, tak by sa tam namiesto *"Iba ďalej buď, nič viac nemusíš"* spievalo *"Iba ďalej buď, nič viac nemôžeš"*.

```C
void *ptr = &num;
*ptr = 29; // nedefinovaná operácia
```

Načo nám je teda takýto pointer? Ukazuje niekam, ale nevieme, čo tam je. V skutočnosti sú void pointery užitočné pri všeobecných operáciách, ako sú kopírovacie, triediace a vyhľadávacie algoritmy. Vďaka nim môžeme napísať jednu funkciu, ktorá funguje pre rôzne dátové typy rovnako. Vytvorme si preto funkciu `array_copy`, ktorá dokáže skopírovať pole `src` do poľa `dst`, pričom obe polia majú kapacitu `bytes` bajtov:
```c
void array_copy(void *dst, void *src, size_t bytes)
{
    char *dst_bytes = (char*)dst;
    char *src_bytes = (char*)src;

    for (size_t i = 0; i < bytes; i++)
        dst_bytes[i] = src_bytes[i];
}

int main()
{
    double a[3] = {1.2, 3.3, 4.5};
    double b[3];
    array_copy(b, a, 3 * sizeof(double));
}
```

Tu môžeme vidieť jednu krásnu zvláštnosť: Vo funkcii `array_copy` si vytvoríme dva pointery na `char`, a to `dst_bytes` a `src_bytes`. Je to z toho dôvodu, že dátový typ `char` zaberá 1 bajt v pamäti, takže `char*` je vlastne *byte* pointer. Potom už iba v cykle skopírujeme jednotlivé bajty z jednej pamäte do druhej. V `main` nezabudnime, že `bytes` je počet bajtov, nie počet prvkov. Tým pádom môžeme mať len jednu funkciu na všetky dátové typy, lebo bajty sú bajty. Nádherné! 😃

Ale prečo potom `void*` nie je natívne `char*`? Opäť je to kvôli alibi voči kompilátoru. Keď špecifikujeme parameter ako `void*`, vypne sa typová kontrola. Vo všeobecnosti je pretypovávanie pointerov veľmi nebezpečná operácia, ktorá by sa mala robiť čo najmenej a pod dohľadom odborníkov (vedie to k fatálnym chybám). Deklarovaním premennej ako `void` pointer hovoríme kompilátoru, že vstupujeme do *danger zone*.


### Pointer na pointer

Toto sa môže zdať ako nepodarený matematický vtip. Už jeden pointer je pre niektorých príliš, a pointer na pointer znie takmer nemožne. Existujú dva spôsoby, ako sa na takúto konštrukciu pozerať.

Prvý pohľad je pomerne jednoduchý: dvojrozmerné polia. Čo je to dvojrozmerné pole? V podstate pole, ktorého každý prvok nie je jedna hodnota, ale ďalšie pole. A keďže pole je v C vlastne pointer, dostávame definíciu pointera na pointer: pointer na pointer predstavuje pole pointerov, pričom každý z nich môže ukazovať na ďalšie pole. To dáva zmysel:

```C
int r = 3; // počet riadkov
int c = 2; // počet stĺpcov
// m = [[a, b], [c, d], [e, f]]
int **m = (int**)malloc(r * sizeof(int*));  // vytvoríme pole pointerov na int

for (int i = 0; i < r; i++)
    m[i] = (int*)malloc(c * sizeof(int)); // každému prvku priradíme pamäť

m[0][1] = 2; // použitie matice

for (int i = 0; i < r; i++)  // uvoľnenie pamäte najskôr vnútorných polí
    free(m[i]);

free(m); // uvoľnenie poľa pointerov
m = NULL; // nastavenie uvoľneného pointera na NULL
```

Druhá vec je, že pointer na pointer vôbec nemusí znamenať dvojrozmerné pole. Môže to byť jednoducho pointer, ktorý ukazuje na iný pointer. Nezabúdajte, že pointer je premenná ako každá iná – je niekde uložená a má svoju adresu:

```c
    int a = 12;
    int *p = &a;
    int **g = &p;
    printf("%p", g);
```

Tento program ukazuje, že pointer `g` ukazuje na pointer `p`, ktorý ukazuje na dáta `a`. 😃 Môžeme tak nielen vidieť, kde je uložená premenná `a`, ale aj kde je uložená premenná `p`.

**Prečo je to dôležité?**

Predstavme si situáciu, kde máme funkcie na vytvorenie a odstránenie jednorozmerného poľa:

```c

int* create_arr(int count)
{
    int *arr = (int*)malloc(count * sizeof(int));
    return arr;
}

void delete_arr(int* arr)
{
   free(arr);
}
```

Tento kód funguje, ale bolo by dobré na konci nastaviť pointer na `NULL`, aby bolo jasné, že už neukazuje na validnú pamäť. Takýto pointer sa nazýva `dangling pointer`:
```c
void delete_arr(int* arr)
{
   free(arr);
   arr = NULL;
}
```
Problém? Táto úprava nemá žiadny efekt. Keďže funkcie pracujú s kópiami parametrov, `arr` sa vnútri funkcie síce nastaví na `NULL`, ale vonku zostane nezmenené. Riešenie? Použiť pointer na pointer:

```c
void delete_arr(int** arr)
{
   free(*arr);
   *arr = NULL;
}

int main()

int *a = create_arr(10);
free(&a); 
```

Teraz odovzdávame adresu pointera, takže môžeme meniť nielen dáta, na ktoré ukazuje, ale aj samotný pointer. Skáčeme po pamäti ako dáma po šachovnici.

### Pointer na funkciu

Dáta majú svoju adresu – ale čo funkcie? Majú aj ony adresy? Áno! A ešte ako sa to dá využiť.

Pointer na funkciu vyzerá takto:

```C
double (*fptr)(int a, int b);
```

Tento pointer ukazuje na funkciu, ktorá má dva `int` parametre a vracia `double`.

Príklad použitia:

```C
void foo(int a) 
{
    printf("%d", a);
}

int main()
{
    void (*show)(int a) = foo; // definovanie pointera
    show(2); // volanie funkcie (dereferencovanie pointera)
}
```

Čo si môžeme všimnúť je to, že názov funkcie tak ako pri poli predstavuje jej adresu. Tým pádom netreba dávať &. A v tomto prípade dereferencia je nič iné ako volanie pôvodnej funkcie. Takže pointer na funkciu nám umožňuje premenovávať si funkcie. Nejaký hlbší význam?

V skutočnosti pointery na funkcie sú veľmi mocné nástroje na tvorenie abstrakných konštrukcií ako sú funkcie vyšších rádov a objektovo-orientované štruktúry. Skúsme najprv tie funkcie vyšších rádov:

Uvažujme dunkciu, čo načíta 2 čísla a vypíše ich súčet:

```c
void vypis_sucet()
{
    int a, b;
    scanf("%d %d", &a, &b);
    int c = a + b;
    printf("%d", c);
}
```

K tejto funkcii snáď nemusíme nič komentovať. Ale ako by to vyzeralo, keby sme chceli spraviť funkciu pre súčin, rozdiel, alebo pre integrál? Nuž museli by sme napísť pre každú operáciu samostatnú funkciu. To by ani tak nebolelo, lebo tá funkcia má 4 riadky, ale skúsme predstierať že táto funkcia má 500 riadkov šialeného kódu. Nebolo by v takom prípade lepšie nejakým spôsobom tú operáciu + - * / nejako prepa3ova+t cez parameter? Čo tak takto?


```c
void vypis_operaciu(int (*fun)(int, int))
{
    int a, b;
    scanf("%d %d", &a, &b);
    int c = fun(a, b);
    printf("%d", c);
}

int sucet(int a, int b)
{
    return a + b;
}

int sucin(int a, int b)
{
    return a * b;
}

int main()
{
    vypis_operaciu(sucet);
    vypis_operaciu(sucin);

}
```

To je nádhera, prosím pozrime si, že pomocou pointerov na funkciu sme oddelili IO logiku od matematickej logiky a vždy keď budeme musieť rozšíriť program stačí napísať iba jednu jednoriadkovú funkciu nie 500 riadkov dlhý kód. tento princíp využíva napr funkcia qsort, ktorá ako parameter prijíma komparačnú funkiu. Oporúčame vám, milí čitatelia si ju pozrieť, všetky pointery, čo má už dávno poznáte.

Druhé použitie je Štrukturálne OOP, inak povedané ako dostať funkcie do štruktúr. Predstavme si, že robíme PC hru, a máme vytvoriť nepriateľov. V našom prípade budú mať život, úroveň útoku a funkciu pozdrav sa. V hre budú dvaja nepriatelia, ninja a dragon, každý sa inak predstaví. 

```c

typedef struct
{
    int health;
    int attack;
    void (*greeting)(void);
} enemy_t;

void greeting_dragon()
{
    printf("I am dragon!\n"); 
}

void greeting_ninja()
{
    printf("I am ninja!\n");
}

int main()
{
    enemy_t dragon = {.health=100, .attack=50, .greeting=greeting_dragon};
    enemy_t ninja  = {.health=30, .attack=10, .greeting=greeting_ninja};

    dragon.greeting();
    ninja.greeting();
}
```

Tu vidíme, že pomocou pointerov na funkcie vieme skoro emulovať objektovo-orientované programovanie a pridávať funkcie k štruktúram elegantne a pekne. 

Aby sme to zhrnuli, pointery na funkcie určite nie sú triviálna vec a využívajú sa najmä pri vyššom progamovaní, ako projektovanie knižníc, API a driverov.

## Záver 

V tomto článku sme si v jednoduchosti predstavili, čo je to operačná pamäť, ako sa do nej ukladajú dáta a že sa v nich vyznať nie je ľahké. Preto boli vymyslené pointery. Mocný nástroj, ktorý nás dokáže v tejto obrovskej pláni viesť a ukazovať nám, kde je čo uložené. Videli sme, že pointerov je viac, každý má svoj zmysel a umožňuje nám robiť iné veci. Niektoré jazyky pointery nemajú, pretože, ako ste videli, vedia byť záhadné. Ale keď im porozumiete, tak viete napísať veľmi rýchly kód.

Dúfame, že po prečítaní tohto článku ste už začali vnímať pointery nie ako svojich nepriateľov, ale ako spoločníkov na dlhých cestách. Tak ako princovia v rozprávkach majú svojich koňov, meče a krásne princezné, my céčkarski programátori máme premenné, funkcie a pointery.










