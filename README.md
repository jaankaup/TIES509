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
histogrammiin.  Laskentalajittelun jälkeen lasketaan histogrammista alkusumma
(*engl. exclusive prefix sum*) joka antaa kunkin luvun aloitus sijainnin
seuraavaa vaihetta varten. Histogrammi kertoo sen kuinka monta mitäkin
k-bittistä lukua on yhteensä ja alkusummat kertovat vastaavasti lukua vastaavan
avaimen sijainnin lopullisessa taulukossa johon luku sijoitetaan. Lopuksi
avaimet sijoitetaan osiinsa (bucket) **k-bittisen** avaimensa perusteella. Tätä
prosessia jatketaan rekursiivisesti seuraaville **k-bittisille** luvuille.
Lopputuloksena saadaan järjestettyjen lukujen jono.

Edellisen esimerkin kohdalla histogrammi ja alkusummat ovat GPU
toteutuksissa kumpikin 256 paikkaisia taulukoita.

## Histogrammi
| a0 | a1 | a2 | ... | a255 |
| -- | -- | -- | --- | ---- |

a0 == a1 == ... == a255 == 0

## Alkusummat (Exclusive prefix sum)
| b0 | b1 | b2 | ... | b255 |
| -- | -- | -- | --- | ---- |

MSB kantaluku lajittelussa otetaan avaimesta 8 ensimmäistä bittiä ja katsotaan
mikä luku on kyseessä. Tässä esimerkissä kyseessä on 8-bittinen luku, joten
arvo alue on [0-255]. Edellisen esimerkin kohdalla luku on 00000010 eli
desimaalilukuna 2. Tällöin histogrammin arvoa indeksissä 2 korotetaan yhdellä.
Tällä tavalla käydään avaimet läpi ja korotetaan aina sen histogrammin
indeksissä olevaa arvoa joka vastaa lukua. Tässä siis toisin sanoen suoritetaan
luvun esiintyvyyksien laskentaa. 

Kun laskenta on saatu päätökseen, suoritetaan alkusumma laskenta
histogrammille.  Exclusive prefix sum lasketaan siten, että lasketaan
kumulatiivisesti histogrammin alkioita alkusummataulukkoon. 

| b0 == 0 | b1 == a0 | b2 == (a0 + a1) | ... | b255 == (a0 + a1 + a2 + ... + 254)] |
| ------- | -------- | --------------- | --- | ----------------------------------- |

Tämä vaihetta kutsutaan nimeltään laskentalajittelu (counting sort). Tämän
jälkeen tehdään ensimmäinen lajittelu vaihe, eli kaikki avaimet käydään läpi,
32-bittinen luku kopioidaan sen 8-bittisen MSB arvon mukaan uuteen taulukkoon.
Taulukon sijainti saadaan alkusumma taulukosta, ja kun arvo on kopioitu
oikealle paikalle alkusumman ko. indeksissä olevaa arvoa kasvatetaan yhdellä.
Näin saadaan seuraavan saman numeron indeksi eli sijainti kohdetaulukossa.
