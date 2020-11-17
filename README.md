# Kirjareferaatti

**Tekijä:** Janne Kauppinen, janne.a.kauppinen@student.jyu.fi<br>
**Vuosi:** 2020<br>
**Kurssi:** TIES509

Tässä työssä tehdään yksi tai useampi kirjareferaatti ennaltasovitusta aiheista.

## A Memory Bandwidth-Efficient Hybrid Radix Sort on GPU's

**Tekijät** Elias Stehle, Hans-Arno Jacobsen<br>
**Vuosi:** 2017<br>
**DOI:** 10.1145/3035918.3064043

| Termi           | Suomennos                |
| --------------- | ------------------------ |
| Radix Sort      | Kantalukulajittelu       |
| LSB             | Vähiten merkitsevä bitti |
| MSB             | Eniten merkitsevä bitti  | 

## Johdanto

Tässä artikkelissa esitetään uusi näytönohjaimella suoritettava
kantalukulajittelu algoritmi. Algoritmi kykenee lajittelemaan 2Gb 8 tavuisia
tietuita 50 millisekunnissa, ja toimii myös niin sanotusti out-of-core
algoritmina. Tämä tarkoittaa sitä että algoritmi pystyy järjestämään myös
sellaista dataa joka ei mahdu kerrallaan näytönohjaimen muistiin.

## Kantalukulajittelu (Radix Sort)

Kantalukulajittelu perustuu **k-bittisten** avaimen tulkitsemista
**d-bittisten** kokonaislukujen jonona. Perus idea on se että jaetaan
**k-bittinen** luku riittävän pieniin **d-bittisiin** lukuihin siten että
kantaluku **r = 2^d** ja että avaimet voidaan jakaa tehokkaasti **r**
erillisiin osiin (*engl.  bucket*).

| MSB  |      |      |      |      |      |      | LSB  |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 0001 | 0010 | 0000 | 1010 | 1111 | 1000 | 1010 | 1000 |

**k** = 32, **d** = 4, **r** = 16

Esimerkki 32 bittisestä avaimesta joka on esitetty 4-bittisten lukujen jonona.
Kunkin osa-jonon kantaluku r on 2^4 = 16.

Avaimet voidaan käydä läpi kahdella eri tavalla: eniten merkisevästä
**d-bittisestä** luvusta kohti vähiten merkitsevää lukua (**MSB radix sort**)
tai päin vastaisessa järjestyksessä (**LSB radix sort**).

MSB radix sort aloittaa lajittelun eniten merkitevästä **k-bittisestä** luvusta
ja jakaa avaimet **r** erilliseen osaan arvonsa mukaan. Tämä tehdään niin
sanotun laskentalajittelu (*engl. counting sort*) avulla siten että että
aloitetaan laskemaan **k-bittisten** lukuen esiintyvyyksiä sitä vastaavaan
histogrammiin. Laskentalajittelun jälkeen lasketaan histogrammista alkusummat
(*engl. exclusive prefix sum*) joka antaa jokaiselle luvulle aloitus sijainnin
avainten sijoittelua varten.

Lopuksi avaimet sijoitetaan osiinsa (bucket) **k-bittisen** avaimensa
perusteella. Tätä prosessia jatketaan rekursiivisesti seuraaville
**k-bittisille** luvuille. Lopputuloksena saadaan järjestettyjen lukujen jono.


MSB kantaluku lajittelussa otetaan kustakin avaimesta 8 ensimmäistä bittiä ja
katsotaan mikä luku on kyseessä. Tässä esimerkissä avaimet ovat jaettu
8-bittisiin lukuihin, joten arvo alue on [0-255]. Edellisen esimerkin kohdalla
luku on 00000010 eli desimaalilukuna 2. Tällöin histogrammin arvoa indeksissä 2
korotetaan yhdellä, eli lisätään luvun kaksi ilmentymismäärää yhdellä. Tällä
tavalla käydään avaimet läpi ja korotetaan aina sen histogrammin indeksissä
olevaa arvoa joka vastaa lukua. Histogrammiin lasketaan siis lukujen
esiintyvyyksien lukumäärää. Tämän jälkeen lasketaan alkusummat joista saadaan
tietää mihin kohtaan avain sijoitetaan. 

## Avaimet
0. | 0001 | 1010 | 1000 | 1011 | 1101 | 1000 | 1010 | 1100 |
1. | 0010 | 0110 | 0001 | 0010 | 0110 | 0010 | 1110 | 1000 |
2. | 1101 | 0010 | 0010 | 1111 | 1001 | 1100 | 1010 | 1000 |
3. | 0001 | 0000 | 0000 | 0010 | 1111 | 1000 | 1011 | 1001 |
4. | 0001 | 1111 | 0000 | 1010 | 0011 | 1110 | 1010 | 1000 |
5. | 1100 | 0011 | 0001 | 1011 | 0000 | 1000 | 1110 | 1010 |
6. | 0010 | 0110 | 0000 | 1110 | 1010 | 0100 | 1011 | 0000 |
7. | 0110 | 0110 | 0000 | 1110 | 1010 | 0100 | 1011 | 0000 |

## Histogrammi
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

## Alkusummat (Exclusive prefix sum)
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

## Tulos
| - | - | - | - | - | - | - | - |

Lasketaan histogrammi eli **k-bittisen** luvun esiintymät:

| **0001** | 1010 | 1000 | 1011 | 1101 | 1000 | 1010 | 1100 |

| 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

| **0010** | 0110 | 0001 | 0010 | 0110 | 0010 | 1110 | 1000 |

| 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

| **1101** | 0010 | 0010 | 1111 | 1001 | 1100 | 1010 | 1000 |

| 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |

| **0001** | 0000 | 0000 | 0010 | 1111 | 1000 | 1011 | 1001 |

| 0 | 2 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |

| **0001** | 1111 | 0000 | 1010 | 0011 | 1110 | 1010 | 1000 |

| 0 | 3 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |

| **1100** | 0011 | 0001 | 1011 | 0000 | 1000 | 1110 | 1010 |

| 0 | 3 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0 |

| **0010** | 0110 | 0000 | 1110 | 1010 | 0100 | 1011 | 0000 |

| 0 | 3 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0 |

| **0110** | 0110 | 0000 | 1110 | 1010 | 0100 | 1011 | 0000 |

| 0 | 3 | 2 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0 |

Kun laskenta on saatu päätökseen, suoritetaan alkusumma laskenta
histogrammista. Alkusummat lasketaan siten, että lasketaan kumulatiivisesti
histogrammin alkioita alkusummataulukkoon. Alkusummataulukosta saadaan tietää
se mihin kohtaan kohdetaulukkoa avain tullaan tallennetaan.

## Histogrammi
| 0 | 3 | 2 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0 |

## Alkusummat (Exclusive prefix sum)
| 0 | 0 | 3 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 |

Tämä vaihetta kutsutaan nimeltään laskentalajittelu (counting sort). Avaimet
lajitellaan nyt eniten merkitsevän 4-bit mukaan järjestykseen käyttäen hyväksi
alkusumma taulukkoa. Alkusumma taulukko kertoo sen mihin kohtaan avain
tallennetaan jonka jälkeen alkusumma taulukon arvoa kasvatetaan yhdellä. Näin
jokainen avain saadaan tallennettua täsmälleen yhteen paikkaan.

a0 :: | **0001** | 1010 | 1000 | 1011 | 1101 | 1000 | 1010 | 1100 | 

prefix sum :: | 0 | 0 | 3 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 |

kohde taulukko :: | a0 | - | - | - | - | - | - | - |

prefix sum :: | 0 | 0+1 | 3 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 |

a1 :: | **0010** | 0110 | 0001 | 0010 | 0110 | 0010 | 1110 | 1000 |

prefix sum :: | 0 | 1 | 3 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 |

kohde taulukko :: | a0 | - | - | a1 | - | - | - | - |

prefix sum :: | 0 | 1 | 3+1 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 |

a2 :: | **1101** | 0010 | 0010 | 1111 | 1001 | 1100 | 1010 | 1000 |

prefix sum :: | 0 | 1 | 4 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 |

kohde taulukko :: | a0 | - | - | a1 | - | - | - | a2 |

prefix sum :: | 0 | 1 | 4 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 7+1 | 8 | 8 |

a3 :: | **0001** | 0000 | 0000 | 0010 | 1111 | 1000 | 1011 | 1001 |

prefix sum :: | 0 | 1 | 4 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 8 | 8 | 8 |

kohde taulukko :: | a0 | a3 | - | a1 | - | - | - | a2 |

prefix sum :: | 0 | 1+1 | 4 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 8 | 8 | 8 |

a4 :: | **0001** | 1111 | 0000 | 1010 | 0011 | 1110 | 1010 | 1000 |

prefix sum :: | 0 | 2 | 4 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 8 | 8 | 8 |

kohde taulukko :: | a0 | a3 | a4 | a1 | - | - | - | a2 |

prefix sum :: | 0 | 2+1 | 4 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 8 | 8 | 8 |

a5 :: | **1100** | 0011 | 0001 | 1011 | 0000 | 1000 | 1110 | 1010 |

prefix sum :: | 0 | 3 | 4 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6 | 8 | 8 | 8 |

kohde taulukko :: | a0 | a3 | a4 | a1 | - | - | a5 | a2 |

prefix sum :: | 0 | 3 | 4 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 6+1 | 8 | 8 | 8 |

a6 :: | **0010** | 0110 | 0000 | 1110 | 1010 | 0100 | 1011 | 0000 |

prefix sum :: | 0 | 3 | 4 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 | 8 |

kohde taulukko :: | a0 | a3 | a4 | a1 | a6 | - | a5 | a2 |

prefix sum :: | 0 | 3 | 4+1 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 | 8 |

a7 :: | **0110** | 0110 | 0000 | 1110 | 1010 | 0100 | 1011 | 0000 |

prefix sum :: | 0 | 3 | 5 | 5 | 5 | 5 | 5 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 | 8 |

kohde taulukko :: | a0 | a3 | a4 | a1 | a6 | a7 | a5 | a2 |

prefix sum :: | 0 | 2 | 5 | 5 | 5 | 5 | 5+1 | 6 | 6 | 6 | 6 | 6 | 7 | 8 | 8 | 8 |

 Tämän
jälkeen tehdään ensimmäinen lajittelu vaihe, eli kaikki avaimet käydään läpi,
32-bittinen luku kopioidaan sen 8-bittisen MSB arvon mukaan uuteen taulukkoon.
Sijainti taulukossa saadaan alkusumma taulukosta. Kun arvo on kopioitu oikealle
paikalle kohde taulukkoa, alkusumman indeksissä olevaa arvoa kasvatetaan
yhdellä. Eli seuraava saman numeron omaava avain kopioidaan yhtä indeksiä
suurempaan paikkaan. Kukin avain tallenttuu siis täsmälleen kerran, ja näin
saatu taulukko on 8 eniten merkitsevän bitin suhteen järjestyksessä. Sama
proseduuri toistetaan ottamalla seuraavat eniten merkitsevät 8-bittiset luvut.
Lopulta kaikki avaimet ovat järjestetty.
