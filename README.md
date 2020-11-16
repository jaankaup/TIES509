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
**d-bittisten** lukujen jonona. Perus idea on se että jaetaan **k-bittinen**
luku riittävän pieniin **d-bittisiin** lukuihin siten että kantaluku **r =
2^d** ja että avaimet voidaan jakaa tehokkaasti **r** erillisiin osiin (*engl.
bucket*).

| MSB      |          |          | LSB      |
| -------- | -------- | -------- | -------- |
| 00000010 | 00001010 | 11111000 | 10101000 |

**k** = 32, **d** = 8, **r** = 256

Esimerkki 32 bittisestä avaimesta joka on esitetty 8-bittisten lukujen jonona.
Kunkin osa-jonon kantaluku r on 2^8 = 256.

Avaimet voidaan käydä läpi kahdella eri tavalla: eniten merkisevästä
**d-bittisestä** luvusta kohti vähiten merkitsevää lukua (**MSB radix sort**)
tai päin vastaisessa järjestyksessä (**LSB radix sort**).

MSB radix sort aloittaa lajittelun eniten merkitevästä **k-bittisestä** luvusta
ja jakaa avaimet **r** erilliseen osaan arvonsa mukaan. Tämä tehdään niin
sanotun laskentalajittelu (*engl. counting sort*) avulla siten että että
aloitetaan laskemaan **k-bittisten** lukuen esiintyvyyksiä sitä vastaavaan
histogrammiin. Laskentalajittelun jälkeen lasketaan histogrammista alkusummat
(*engl. exclusive prefix sum*) joka antaa kunkin luvun aloitus sijainnin
lajittelua varten. Histogrammi pitää sisällään kunkin luvun k-bittisen luvun
esiintymismäärät. Alkusummat kertovat vastaavasti lukua vastaavan avaimen
sijainnin lopullisessa tietorakenteessa johon avain tallennetaan. Tietorakenne
on GPU:ssa taulukko. Lopuksi avaimet sijoitetaan osiinsa (bucket)
**k-bittisen** avaimensa perusteella. Tätä prosessia jatketaan rekursiivisesti
seuraaville **k-bittisille** luvuille. Lopputuloksena saadaan järjestettyjen
lukujen jono.

Edellisen esimerkin mukainen histogrammi ja alkusummat ovat GPU
toteutuksissa kumpikin 256 paikkaisia taulukoita.

## Histogrammi
| a0 | a1 | a2 | ... | a255 |
| -- | -- | -- | --- | ---- |

a0 == a1 == ... == a255 == 0

## Alkusummat (Exclusive prefix sum)
| b0 | b1 | b2 | ... | b255 |
| -- | -- | -- | --- | ---- |

MSB kantaluku lajittelussa otetaan kustakin avaimesta 8 ensimmäistä bittiä ja
katsotaan mikä luku on kyseessä. Tässä esimerkissä avaimet ovat jaettu
8-bittisiin lukuihin, joten arvo alue on [0-255]. Edellisen esimerkin kohdalla
luku on 00000010 eli desimaalilukuna 2. Tällöin histogrammin arvoa indeksissä 2
korotetaan yhdellä, eli lisätään luvun kaksi ilmentymismäärää yhdellä. Tällä
tavalla käydään avaimet läpi ja korotetaan aina sen histogrammin indeksissä
olevaa arvoa joka vastaa lukua. Histogrammiin lasketaan siis lukujen
esiintyvyyksien lukumäärää. 

Kun laskenta on saatu päätökseen, suoritetaan alkusumma laskenta
histogrammista. Alkusummat lasketaan siten, että lasketaan kumulatiivisesti
histogrammin alkioita alkusummataulukkoon. Alkusummataulukosta saadaan tietää
se mihin kohtaan kohdetaulukkoa avain tullaan tallennetaan.

| b0 == 0 | b1 == a0 | b2 == (a0 + a1) | ... | b255 == (a0 + a1 + a2 + ... + 254)] |
| ------- | -------- | --------------- | --- | ----------------------------------- |

Tämä vaihetta kutsutaan nimeltään laskentalajittelu (counting sort). Tämän
jälkeen tehdään ensimmäinen lajittelu vaihe, eli kaikki avaimet käydään läpi,
32-bittinen luku kopioidaan sen 8-bittisen MSB arvon mukaan uuteen taulukkoon.
Sijainti taulukossa saadaan alkusumma taulukosta. Kun arvo on kopioitu oikealle
paikalle kohde taulukkoa, alkusumman indeksissä olevaa arvoa kasvatetaan
yhdellä. Eli seuraava saman numeron omaava avain kopioidaan yhtä indeksiä
suurempaan paikkaan. Kukin avain tallenttuu siis täsmälleen kerran, ja näin
saatu taulukko on 8 eniten merkitsevän bitin suhteen järjestyksessä. Sama
proseduuri toistetaan ottamalla seuraavat eniten merkitsevät 8-bittiset luvut.
Lopulta kaikki avaimet ovat järjestetty.
