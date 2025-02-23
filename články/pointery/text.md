# Pointery, lístočky na chladničke a Minotaurus

Tak ako v každej rozprávke nájdeme nielen šarmantné princezné, ale aj nešarmantných drakov, tak v jazyku C (C++) okrem krásneho kódu narazíme aj na škaredé pointery. Keď sa s nimi prvýkrát stretneme, častokrát nechápeme čo sa deje a náš program elegantne spadne do priepasti segmentatačných faulov. No čím viac nocí nad nimi preplačeme, tým skôr zistíme, že tieto škaredé káčatká sú v skutočnosti krásne labute. A aby tých bezsenných nocí bolo čo najmenej, pripravili sme pre vás tento článok, v ktorom vám pointery predstavíme tak trochu inak.

<img src="title.jpg" alt="Obr. Labyrint" width="400" style="display: block; margin: auto;">

## Hello world, pointery

Vaše prvé stretnutie s pointermi mohlo vyzerať nejako takto:
```c
int num = 45;
int *ptr = &num; // pointer na num, '*' je označenie pointera, '&' vráti adresu 'num'
```

K tomu pravdepodobne nasledoval takýto popis: 
*Premenná `ptr` je špecifická tým, že neuchováva hodnotu priamo, ale uchováva adresu pamäťového miesta, kde sa táto hodnota nachádza. Operátor `&` vracia adresu premennej `num`, zatiaľ čo operátor `*` označuje, že premenná `ptr` je pointer. Tento pointer umožňuje manipulovať s obsahom premennej `num` prostredníctvom jej adresy v pamäti. To dosiahneme opätovným použitím operátora `*` na premennú `ptr`. Tentoraz však `*` neslúži na označenie pointera, ale na jeho **dereferencovanie** – získanie hodnoty uloženej na danej adrese.*

```c
*ptr = 32;   // teraz je hodnota num = 32, '*'je dereferencovanie pointera
print(*ptr);
```

Ale počkať, naskytá sa tu prirodzená otázka: *Načo nám je nejaká divná premenná `ptr`, keď môžeme hodnotu premennej `num` zmeniť priamo?*
```c
num = 32; 
```
*Why you have to go and make things so complicated?“* - ako spieva kanadská speváčka *Avril Lavigne* vo sojej piesni *Complicated*. Odpoveď na túto zákernú otázku si vyžaduje hlbšie pochopenie toho, čo je to pamäť a ako vlastne funguje. Preto sa musíme vydať na dlhú a únavnú cestu do jej hlbín. 🌊

## Vitajte v pamäti, priatelia  

Priatelia, vitajte v pamäti! Tu sa všetko odohráva, tu sa všetko deje. V prvom rade nám ale dovoľte túto pôvabnú dámu predstaviť jej pravým menom, a to: **operačná pamäť**. Väčšina z nás ju však pozná skôr ako pamäť *RAM (Random Access Memory)*, teda pamäť, ktorá umožňuje prístup na ľubovoľnú jej pozíciu v konštantnom čase. Dôležité je ale pochopiť, že *RAM* nie je názov konkrétnej pamäte, ale iba spôsob ukladania dát. Používať označenie *RAM* je to isté ako povedať, že jazdíme v *benzíne* namiesto toho, že jazdíme v *aute*.

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

Vidíme, že každá premenná zaberá v pamäti určitý počet bajtov, ktoré sú vedľa seba naukladané. Štandartne `int` má 4 bajty, `double` 8 bajtov a `char` 1 bajt (Avšak nemusí to platiť pre všetky počítačové architektúry). Tento princíp platí aj pri zložitejších dátových typoch, ako sú polia a štruktúry. Skúsme sa najprv pozrieť na polia (ale nie také, po ktorých behajú kombajny):

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

Daidalov labyrint, pochádzajúci z gréckych bájí, je jedným z najznámejších symbolov zložitosti a bezvýchodiskovosti. Podľa legendy ho postavil geniálny remeselník Daidalos pre kráľa Minoa. Jeho účel bol v tom,  aby uväznil obávaného Minotaura – napoly človeka, napoly býka. Labyrint bol taký zložitý, že prakticky z neho neexistovala cesta von a ktokoľvek doň vstúpil, už nikdy nenašiel východ. A tu prichádza na scénu Ariadna, krétska princezná, ktorá pomohla hrdinovi Théseovi prekonať tento labyrint. Darovala mu niť, ktorú pri vstupe priviazal a postupne odvíjal. Keď porazil Minotaura, stačilo mu sledovať niť a bezpečne sa tak vrátiť na začiatok. Tento jednoduchý trik premenil beznádejný chaos labyrintu na prehľadnú a zvládnuteľnú štruktúru.

Presne takto fungujú v programovaní pointery. Pamäť môžeme vnímať ako obrovský labyrint bajtov, kde neexistuje žiadne pevné usporiadanie, iba surové údaje roztrúsené v priestore. Bez jasných orientačných bodov by sme sa v nej rýchlo stratili. Pointery sú našou **Ariadninou niťou** – ukazujú na konkrétne miesta v pamäti, umožňujú nám sledovať vzťahy medzi údajmi a efektívne sa pohybovať v tomto zdanlivo chaotickom priestore. Bez nich by sme len blúdili naslepo, dúfajúc, že náhodou trafíme správnu hodnotu. 🔭

### Takže čo je to vlastne pointer?

Vráťme sa na začiatok a definujme si, čo to vlastne pointer je. Pointer je jednoducho premenná, ktorá uchováva celé číslo reprezentujúce pozíciu v pamäti. Štandardne má pointer veľkosť 8 bajtov, aby pokryl všetky možné adresy súčasných 64-bitových systémov; na starších 32-bitových systémoch sa môžeme stretnúť aj s veľkosťou 4 bajty. Ak si zoberieme náš prvý príklad a premietneme ho na tabuľkový model pamäte, vyzeralo by to asi takto:

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
* počet bajtov, na ktoré ukazuje,
* kódovanie dát.

```c
// int = počet bajtov = 4, kódovanie = 'signed integer'
// &a = adresa prvého bajtu premennej a
int *p = &a;
```

Prvá vlastnosť je obsiahnutá v jeho hodnote a zvyšné dve sú skryté v jeho type.

### Polia sú v skutočnosti pointery
Keď už sme sa ako-tak zoznámili s tým, že pointer má podobu čísla, je načase odkryť jeho druhú tvár – tvár poľa:
Každé pole, či už statické alebo dynamické, je v skutočnosti **pointer na jeho prvý prvok v pamäti**. A tajomstvo je odhalené (dramatická hudba) 🎶:

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", *ptr); // 1
```

Čiže si to zhrňme, dátová štruktúra `array` je v jazyku C implementovaná ako **pointer na blok pamäte**. To znamená, že existuje medzi pointermi a poľami určitá dualita: polia sú pointery a pointery môžu byť polia. Ale ešte je tu jedna vec: aby sme mohli s poľami narábať, musíme ich vedieť indexovať. Čo majú pointery?

Pre pointery existuje niečo, ako **pointerovská aritmetika**, ktorá opisuje dve ďalšie možnosti, s ktorými sa dá s pointermi pracovať:

* Posunutie pointera o celé číslo. Napríklad operácia `ptr + 1` znamená posunutie pointera o jeden `int`, čiže o 4 bajty. Keby bol `ptr` typ `double`, tak sa pointer posunie o jeden `double` (8B).
* Odčítanie dvoch pointerov – určenie vzdialenosti medzi nimi: `size_t d = (ptr2 - ptr1)` 

Pomocou pointerovskej aritmetiky bude indexovanie poľa vyzerať takto:

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

To isté platí aj pre dynamické polia, kde je tento mechanizmus explicitnejší. Funkcia `malloc()` v C, alebo operátor `new` v C++ si vypýtajú od operačného systému blok pamäte, ktorého začiatočná adresa sa uloží do lokálneho pointera. Na rozdiel od statických poli je nutné alokovanú pamäť po skončení operácií manuálne uvoľniť.

⚠️ *Dôležité upozornenie: pri `malloc()` treba špecifikovať počet bajtov, nie počet prvkov. Preto je vhodné použiť `sizeof(int)`, aby sa zabránilo nesprávnemu prideľovaniu pamäte. Zároveň je vhodné výsledok pretypovať na `int*`.*

```c
int *arr = (int*)malloc(10 * sizeof(int)) // 10 int prvkov
arr[0] = 1;
arr[1] = 2;
arr[2] = 3;

free(arr); // nezabudnime uvoľniť
```

Môžeme vidieť, že práca s dynamickými poľami je na 99 % rovnaká ako pri poliach statických.
Hlavný rozdiel medzi statickými a dynamickými poliami je v spôsobe ich správy. Statické polia majú pevnú veľkosť určenú pri kompilácii, kým dynamické polia možno alokovať a meniť za behu programu. Navyše, zatiaľ čo statické polia sú implicitne (skryto) spravované a automaticky uvoľňované kompilátorom, dynamické polia vyžadujú manuálne uvoľnenie pamäte programátorom. Žiaľ, programátorom sme my. Keby bol niekto zvedavý, tu je prehľadová tabuľka, ktorá popisuje rozdiel medzi jednotlivými poľami (a lúkami). 

| Vlastnosť          | Statické pole                 |            Dynamické pole            |
|--------------------|-------------------------------|--------------------------------------|
| Alokácia           | Kompilátor (stack/static)     | `new` / `malloc()` (heap)            |
| Veľkosť            | Fixná pri kompilácii          | Možno meniť za behu programu         |
| Je to pointer?     | Áno, implicitne (skryto)      | Áno, explicitne (okato)              |
| Manuálne mazanie?  | Nie                           | Áno (`free()` / `delete[]`)          |

💡 Zaujímavosť: konštrukcia `*(ptr + i) = ptr[i]` nekontroluje dátové typy a keďže plus je komutatívna operácia, daný výraz možno zapísať aj ako: `*(i + ptr) = i[ptr]`. Potom by náš príklad vyzeral takto:

```c
int arr[] = {1, 2, 3};
int *ptr = arr;
printf("%d", 0[ptr]); // 1
printf("%d", 1[ptr]); // 2
printf("%d", 2[ptr]); // 3
```

Síce tento kód je funkčný, ale prosím nerobte to, je to nezdravé. 😃


### Pointery sú vstupno-výstupné parametre
Ak ste si mysleli, že pointery majú iba dve tváre, tak tu máte tretiu - funkčnú tvár. Pri funkciách poznáme niečo ako **prenášanie hodnotou resp. prenášanie odkazom**, čo si pozrieme v nasledujúcom príklade:

```c
void scan_arr(int arr[], int len)
{
    for (int i = 0; i < len; ++i)
        scanf("%d", &arr[i]);
}
```

Funkcia `scan_arr` má dva parametre: **pole `arr` a jeho dĺžku `len`**. Medzi nimi je však zásadný rozdiel (vieme už že pole je vlastne pointer). Keď sa funkcia spustí, pod kapotou si zo svojich parametrov vytvorí svoje vlastné kópie s ktorými pracuje. Takže vo vnútri `scan_arr` vzniknú premenné `arr2` a `len2`.

- **Premenná `len2` nemá okrem rovnakej hodnoty žiadne spojenie s pôvodnou `len`**, čiže keď ju funkcia zmení, nepremietne sa to vo zvyšnom programe. Tomuto hovoríme prenášanie hodnotou.
- **Premenná `arr2` je však kópia pointera `arr`**, čo znamená, že síce `arr2` a `arr` sú síce dve rôzne premenné, ale stále ukazujú na to isté pole (resp. pamäť) a môžu ho meniť. Toto je prenášanie pointerom (odkazom). Pri poliach sa používa natívne, pretože to má aj ekonomický dôvod: **namiesto kopírovania desiatok či stoviek bajtov stačí skopírovať len 8 bajtov!**

Prenášanie pointerom je niečo ako lístoček na chladničke. Keď vám niekto nechá lístoček na chladničke s popisom, kde je večera, tak viete si tú večeru nájsť a narábať s ňou (zjesť ju). Analogicky to funguje aj pri funkciách. Keď pointer vlezie do funkcie, automaticky sa stáva jej  **vstupno-výstupným parametrom**. Upravme si danú funkciu, aby načítala nie len pole ale aj jeho dĺžku a vrátila výsledok cez parametre:

```c
void scan_arr(int *arr, int *len)   // arr[] je to isté ako *arr
{
    scanf("%d", len);  // pri pointeroch nepoužívame `&` pri scanf, lebo ich meno je adresa

    for (int i = 0; i < *len; ++i)
        scanf("%d", &arr[i]);
}
```

Teraz môžeme vidieť, že aj `arr` aj `len` sú pointery, čiže je možné ich hodnoty vo funkcii meniť. Aké pôvabné. 

## Bratracni a sesternice pointerov

Už sme sa zoznámili s jednoduchým jednohviezdičkovým pointerom `*ptr` a všetkými jeho tvárami. Ale v skutočnosti ich existuje celá rodina – niektorí príbuzní sú známejší, iní zas trochu čudáci s vlastnými pravidlami. Všetkých spája spoločná DNA ukazovania na adresy, no každý má svoju špeciálnu úlohu. Aby sme sa v tomto rodinnom strome nestratili, poďme si ich pekne postupne predstaviť.


### Pointer na štruktúru

Pri pointeroch na štruktúru v zásade nemusíme používať nič špeciálne. Stačí nám operátor `*`:

```C
typedef struct
{ 
    int n; // n/d 
    int d;

} fraction_t; // štruktúra zlomok

int main()
{
    fraction_t f = {2, 3}; // 2/3
    fraction_t *fptr = &f; 

    printf("%d/%d\n", (*fptr).n, (*fptr).d);
}
```

Funkčne je tento zápis úplne v poriadku, ale na súťaž krásy by to asi nebolo. Konštrukcia `(*fptr).n` vyzerá skôr ako test zručnosti na klávesnici než niečo, s čím by sme chceli pracovať každý deň. Našťastie existuje elegantnejší zápis v podobe operátora šípky `->`:

```C
int main()
{
    fraction_t f = {2, 3}; // 2/3
    fraction_t *fptr = &f; 

    printf("%d/%d\n", fptr->n, fptr->d);
}
```

Tak toto už je iná káva ☕! Vidíme, že pointery na štruktúry sa správajú rovnako ako bežné pointery, len s pohodlnejším zápisom (angl. šípka/ručička je pointer 😃).

### Pointer na `void`

No super! Minule sme si povedali, že každý pointer ukazuje na nejaký dátový typ. Lenže `void` žiadnym konkrétnym typom nie je. Pri funkciách ho síce používame, ale iba ako alibi, keď tvrdíme, že nevracajú žiadnu hodnotu: Je to *fantómový* typ. A presne tak funguje aj `void*`.

Predstame si to takto: `void*` síce uchováva adresu nejakých dát, ale vôbec netuší, aké veľké sú alebo aké kódovanie používajú. Je to teda len surová adresa bez ďalších detailov – univerzálny, no slepý ukazovateľ.

```C
int num = 30;
void *ptr = &num;
```

💡 Takýto pointer môže iba existovať – nič iné s ním nespravíme (dereferencovanie, posúvanie). Keby slovenská pieseň od skupiny Elán *Voda, čo ma drží nad vodou* bola o `void` pointeroch, tak by sa namiesto *"Iba ďalej buď, nič viac nemusíš"* spievalo *"Iba ďalej buď, nič viac nemôžeš"*.

```C
void *ptr = &num;
*ptr = 29; // nedefinovaná operácia
```

Načo nám je teda takýto pointer? Ukazuje niekam, ale nevieme, čo tam je. V skutočnosti sú `void*` pointery užitočné pri všeobecných operáciách, ako sú kopírovacie, triediace a vyhľadávacie algoritmy. Umožňujú nám napísať jednu funkciu, ktorá funguje pre rôzne dátové typy rovnako.

Vytvorme si teda funkciu `array_copy`, ktorá skopíruje pole `src` do `dst`, pričom obe polia majú kapacitu `bytes` bajtov:
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

Tu vidíme jednu krásnu zvláštnosť: vo funkcii `array_copy` si vytvoríme dva pointery na `char`: `dst_bytes` a `src_bytes`. Je to preto, že `char` zaberá presne 1 bajt v pamäti, takže `char*` je vlastne *byte* pointer. Potom už len v cykle kopírujeme jednotlivé bajty z jednej pamäte do druhej. V `main` nezabudnime, že `bytes` je počet bajtov, nie počet prvkov! Vďaka tomu môžeme mať jedinú funkciu na všetky dátové typy, lebo bajty sú bajty. Nádherné! 😃

Ale prečo potom `void*` nie je natívne `char*`? Opäť ide o určité alibi voči kompilátoru. Keď špecifikujeme parameter ako `void*`, vypne sa typová kontrola. Vo všeobecnosti je pretypovávanie pointerov nebezpečné a malo by sa používať čo najmenej – pod dohľadom odborníkov (alebo aspoň s veľkou dávkou rešpektu). Deklarovaním premennej ako `void*` kompilátoru hovoríme, že s plným vedomím vstupujeme do *danger zone*. ⚠️


### Pointer na pointer

Toto sa môže zdať ako nepodarený matematický vtip. Pre niektorých je už jeden pointer priveľa, a pointer na pointer znie takmer nemožne. Existujú však dva spôsoby, ako sa na takúto konštrukciu pozerať.

**Dvojrozmerné polia**

Prvý pohľad je pomerne jednoduchý: dvojrozmerné polia. Čo je to dvojrozmerné pole? V podstate pole, ktorého každý prvok nie je jedna hodnota, ale ďalšie pole. A keďže pole je v jazyku C vlastne pointer, dostávame definíciu pointera na pointer a to: **pole pointerov**, pričom každý z nich môže ukazovať na ďalšie pole. To dáva zmysel. Ukážme si preto narábanie s dvojrozmerným dynamickým poľom:

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

**Iba pointer čo ukazuje na druhý pointer**

Druhá vec je, že pointer na pointer vôbec nemusí znamenať dvojrozmerné pole. Môže to byť jednoducho pointer, ktorý ukazuje na iný pointer. Netreba zabúdať, že pointer je premenná ako každá iná – je niekde uložená a má svoju adresu:

```c
int a = 12;
int *p = &a;
int **g = &p;
printf("%p", g);
```

Tento program ukazuje, že pointer `g` ukazuje na pointer `p`, ktorý ukazuje na dáta `a`. 😃 Môžeme tak nielen vidieť, kde je uložená premenná `a`, ale aj kde je uložená premenná `p`.

Prečo je to dôležité? Predstavme si situáciu, kde máme funkcie na vytvorenie (`create_arr()`) a odstránenie (`delete_arr()`) jednorozmerného *dynamického* poľa:

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

Tento kód funguje, ale bolo by dobré pri funkcii `delete_arr` na konci nastaviť pointer `arr` na `NULL`, aby bolo jasné, že už neukazuje na platnú pamäť. Inými slovami, nechceme v programe žiadny `dangling pointer`:
```c
void delete_arr(int* arr)
{
   free(arr);
   arr = NULL;
}
```
Problém? Táto úprava nemá žiadny efekt. Keďže funkcie pracujú len s kópiami parametrov, `arr` sa síce nastaví vo vnútri funkcie na `NULL`, ale vonku zostane nezmenené. Riešenie? Použiť `pointer na pointer`:

```c
void delete_arr(int** arr)
{
   free(*arr);
   *arr = NULL;
}

int main()
{
    int *a = create_arr(10);
    free(&a);  // & je adresa pointera a, nie adresa dát
}
```

Teraz odovzdávame adresu pointera, takže môžeme meniť nielen dáta, na ktoré ukazuje, ale aj samotný pointer. Skáčeme po pamäti ako dáma po šachovnici.♟️

### Pointer na funkciu

Dáta majú svoju adresu – ale čo funkcie? Majú aj ony adresy? Áno! A to je jedna z najkrajších vlastností jazyka C. Funkcie v skutočnosti nie sú nič iné ako inštrukcie uložené v pamäti, takže aj na ne môžeme odkazovať pomocou pointerov.

A na čo je to dobré? Pointery na funkcie umožňujú dynamicky voliť, ktorá funkcia sa zavolá – napríklad pri implementácii `callbackov`, teda funkcií, ktoré môžu byť parametrom iných funkcií, alebo pri ukladaní funkcií do štruktúr, čo sa využíva napríklad v polymorfizme alebo pri tvorbe tabuliek funkcií.

Pointer na funkciu vyzerá takto:

```C
double (*fptr)(int a, int b);
// double (*fptr)(int, int);  // alternatívne
```

Tento pointer ukazuje na funkciu, ktorá má dva parametre typu `int` a vracia hodnotu s typom `double`. Inými slovami, `fptr` je premenná, ktorá môže uchovávať adresu ľubovoľnej funkcie s takýmto rozhraním. Všimnime si, že pri parametroch nemusíme špecifikovať ich názov, ale je to užitočné napríklad pri tvorbe aplikačných rozhraní.

Príklad použitia:

```C
void foo(int a) 
{
    printf("%d", a);
}

int main()
{
    void (*show)(int a) = foo; // definovanie pointera
    show(25); // volanie funkcie (dereferencovanie pointera)
}
```

Čo si môžeme všimnúť je to, že názov funkcie, podobne ako názov poľa, predstavuje aj jej adresu. Preto nemusíme používať operátor `&`. Volanie funkcie cez pointer je zároveň ekvivalentom jej priameho volania – dereferencovanie pointera vlastne znamená vykonanie pôvodnej funkcie.


**Implementácia callbalckov alebo funkcií ako parametre**

Pozrime sa teraz na prvé praktické využitie – implementáciu `callbacku`. Uvažujme funkciu, ktorá načíta dve čísla z klávesnice a vypíše ich súčet:

```c
void vypis_sucet()
{
    int a, b;
    scanf("%d %d", &a, &b);
    int c = a + b;
    printf("%d", c);
}
```

Na tejto funkcii nie je nič zvláštne – jednoducho prečíta dve čísla a vypočíta ich súčet. Ale čo ak by sme chceli podobnú funkciu pre rozdiel, súčin alebo dokonca integrál? Museli by sme písať samostatnú verziu pre každú operáciu. Pri štyroch riadkoch kódu by to možno nebolo také hrozné, ale čo ak by táto funkcia mala 500 riadkov komplikovanej logiky? Neexistuje spôsob, ako namiesto duplikovania kódu nejako prepašovať operáciu (`+`, `-`, `*`, `/`) cez parameter?

Čo keby sme to skúsili takto:

```c
void vypis_operaciu(int (*fun)(int, int))
{
    int a, b;
    scanf("%d %d", &a, &b);
    int c = fun(a, b);     // volanie funkcie fun
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

To je nádhera! Vidíme, že pomocou pointerov na funkciu sme elegantne oddelili vstupno-výstupnú logiku (`printf()`, `scanf()`) od matematickej logiky (`+`, `*`). A čo je ešte lepšie – ak budeme chcieť v budúcnosti rozšíriť program o ďalšie operácie, stačí nám napísať len jednu jedoduchú funkciu namiesto 500 riadkov nového kódu, v ktorom sa trikrát pomýlime.

Tento princíp sa využíva aj v štandardnej funkcii `qsort()`, ktorá ako parameter prijíma komparačnú funkciu. Odporúčame vám, milí čitatelia, aby ste sa na ňu pozreli – všetky pointery, ktoré tam nájdete, už dávno poznáte!


**Štrukturálne OOP: Ako dostať funkcie do štruktúr**

Druhým praktickým využitím pointerov na funkcie je takzvané **"štrukturálne (fejkové) objektovo-orientované programovanie"**. Inými slovami, ako môžeme dostať funkcie do štruktúr?

Predstavme si, že pracujeme na PC hre a potrebujeme vytvoriť rôznych nepriateľov. Každý nepriateľ bude mať určitú hodnotu života `health`, úroveň útoku `attack` a unikátny spôsob predstavenia sa `greeting()`. V našom prípade budeme mať dvoch nepriateľov – `ninju` a `draka` – pričom každý sa ohlási iným spôsobom.

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

Tu vidíme, že pomocou pointerov na funkcie dokážeme takmer emulovať *objektovo-orientované programovanie v jazyku C*. Umožňujú nám elegantne a čisto pridávať funkcie k štruktúram, čím zlepšujú organizáciu kódu a jeho rozšíriteľnosť.

Na záver môžeme povedať, že pointery na funkcie určite nepatria medzi úplne triviálne koncepty. Ich skutočná sila sa prejavuje najmä pri pokročilejšom programovaní – napríklad pri návrhu knižníc, API rozhraní, alebo dokonca ovládačov (driverov). Ak ich pochopíte a ovládnete, otvoria sa vám dvere k mnohým pokročilým technikám v jazyku C.

## Záver 

A zazvonil zvoniec a pointerom je koniec? Ani náhodou! Naša cesta sa práve začala. V tomto článku sme si jednoducho predstavili, čo je to operačná pamäť, ako sa do nej ukladajú dáta a prečo nie je jednoduché sa v nej vyznať. Práve preto vznikli pointery – mocný nástroj, ktorý nám pomáha orientovať sa v obrovskej pamäťovej krajine ukazujúc, kde je čo uložené.

Videli sme, že pointerov je mnoho a každý má svoj príbeh a svoj účel. Niektoré programovacie jazyky pointery nemajú, pretože, ako ste sami zistili, vedia byť poriadne záhadné. Ale keď im porozumiete, odomknú vám cestu k efektívnemu a mimoriadne rýchlemu kódu. Váš program bude bežať ako ako vetrom poháňaný tátoš.

Dúfame, že po prečítaní tohto článku už pointery nevnímate ako nepriateľov, ale ako spoľahlivých spoločníkov na vašej programátorskej ceste. Tak ako rytieri v rozprávkach majú svoje meče, kone a neodolateľné princezné, my, céčkarski programátori, máme premenné, funkcie a samozrejme pointery.

## Užitočná literatúra
Ak vás pointery a jazyk C zaujali, odporúčame knihu: **Programovací jazyk C – Brian W. Kernighan, Dennis M. Ritchie**. Toto je klasika, ktorú by mal mať každý céčkarsky programátor v knižnici. Nájdete v nej nielen základy, ale aj pokročilé techniky, ktoré vám pomôžu pochopiť, prečo je C taký mocný (a občas nemilosrdný) jazyk.
