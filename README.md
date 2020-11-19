# Kirjareferaatti

**Tekijä:** Janne Kauppinen, janne.a.kauppinen@student.jyu.fi<br>
**Vuosi:** 2020<br>
**Kurssi:** TIES509

Tässä työssä tehdään yksi tai useampi kirjareferaatti ennaltasovitusta aiheista.

## A Memory Bandwidth-Efficient Hybrid Radix Sort on GPU's

**Tekijät** Elias Stehle, Hans-Arno Jacobsen<br>
**Vuosi:** 2017<br>
**DOI:** 10.1145/3035918.3064043

| Termi           | Suomennos                 |
| --------------- | ------------------------- |
| Radix Sort      | Kantalukulajittelu        |
| LSD             | Vähiten merkitsevä numero |
| MSD             | Eniten merkitsevä numero  | 

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
erillisiin osiin (*engl. bucket*).

| MSD  |      |      |      |      |      |      | LSD  |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 0001 | 0010 | 0000 | 1010 | 1111 | 1000 | 1010 | 1000 |

**k** = 32, **d** = 4, **r** = 16

Esimerkki 32 bittisestä avaimesta joka on esitetty 4-bittisten lukujen jonona.
Kunkin osa-jonon kantaluku r on 2^4 = 16 mikä on myös *bucketien* lukumäärä.

Avaimet voidaan käydä läpi kahdella eri tavalla: eniten merkisevästä
**d-bittisestä** luvusta kohti vähiten merkitsevää lukua (**MSD radix sort**)
tai päin vastaisessa järjestyksessä (**LSD radix sort**).

MSD radix sort aloittaa lajittelun eniten merkitevästä **k-bittisestä** luvusta
ja jakaa avaimet **r** erilliseen *bucketiin* arvonsa mukaan. Tämä tehdään niin
sanotun laskentalajittelu (*engl. counting sort*) avulla siten että että
aloitetaan laskemaan **k-bittisten** lukuen esiintyvyydet histogrammiin.
Histogrammin avulla lasketaan alkusummat (*engl. exclusive prefix sum*) joka
antaa jokaiselle luvulle aloitus sijainnin avainten sijoittelua varten.

Lopuksi avaimet sijoitetaan osiinsa (bucket) **k-bittisen** avaimensa
perusteella. Tätä prosessia jatketaan rekursiivisesti seuraaville
**k-bittisille** luvuille. Lopputuloksena saadaan järjestettyjen lukujen jono.

## Esimerkki kantalukulajittelusta (MSD)

Tässä on esimerkki MSD kantalukulajittelusta. Järjestetään 8-bittisiä avaimia
siten että tulkitaan avaimet 4-bitin osissa. 

MSD kantalukulajittelussa lähdetään liikenteeseen eniten merkitsevästä
4-bittisestä luvusta. Kantalukulajittelu ei perustu avainten vertailuun vaan se
perustuu lukujen esiintyvyyksien laskentaan. Lukujen esiintyvyyksien perusteella 
perusteella avaimet tallennetaan kohde taulukkoon. Kunkin vaiheen jälkeen
avaimet ovat järjestyksessä eniten merkitsevien lukujen perusteella ja tätä
prosessia jatketaan kunnes kaikki avaimet ovat järjestetty.

### Avaimet
id  | 4-bit osissa | 10-kantaisessa osissa | Arvo |   
--- | ------------- | ------------------ | ---- |  
a0  | **0001** 1010 |  **1** ,    10 | 26 |
a1  | **0010** 0110 |  **2** ,     6 | 38 |
a2  | **1101** 0010 | **13** ,     2 | 210 |
a3  | **0001** 0000 |  **1** ,     0 | 16 |
a4  | **0001** 1111 |  **1** ,    15 | 31 |
a5  | **1100** 0011 | **12** ,     3 | 195 |
a6  | **0010** 0110 |  **2** ,     6 | 38 |
a7  | **0110** 0110 |  **6** ,     6 | 102 |
a8  | **1001** 1010 |  **9** ,    10 | 154 |
a9  | **0010** 1110 |  **2** ,    14 | 46 |
a10 | **1111** 0000 | **15** ,     0 | 240 |
a11 | **0001** 0001 |  **1** ,     1 | 17 |
a12 | **0001** 0100 |  **1** ,     4 | 20 |
a13 | **1110** 0011 | **14** ,     3 | 227 |
a14 | **0010** 0110 |  **2** ,     6 | 38 |
a15 | **0111** 0110 |  **7** ,     6 | 118 |

**k** = 8, **d** = 4, **r** = 16

### Histogrammi (msd lukujen esiintymismäärät)
| 0 | 5 | 4 | 0 | 0 | 0 | 1 | 1 | 0 | 1 | 0 | 0 | 1 | 1 | 1 | 1 |

### Alkusummat (Exclusive prefix sum)
| 0 | 0 | 5 | 9 | 9 | 9 | 9 | 10 | 11 | 11 | 12 | 12 | 12 | 13 | 14 | 15 |

### Tulos
| - | - | - | - | - | - | - | - |

Histogrammiin tallenetaan tieto lukujen lukumääristä. Histogrammista nähdään
myös se kuinka monta avainta tulee olemaan kussakin bucketissa.
Laskentalajittelu suoritetaan rekursiivisesti aina jokaiselle bucketille.

Alkusummat kertovat sen etäisyyden (offset) taulukossa johon avain tallennetaan
kun avaimia aletaan käydä läpi.

Histogrammi kertoo myös sen kuinka monta avainta menee kuhunkin buckettiin.
Tässä esimerkissä ensimmäiseen buckettiin kuuluu kaksi avainta. Alkusummasta nähdään että
ensimmäiseen bucketin aloitusindeksi on 0.

#### Sijoitetaan ensimmäinen avain taulukkoon histogrammin avulla.

a0 :: | **0001** 1010 |

Haetaan histogrammista taulukon indeksi johon avain kopioidaan. Näin avaimet saadaan 
laskettua suoraan oikeaan kohtaan.
| 0 | **0** | 5 | 9 | 9 | 9 | 9 | 10 | 11 | 11 | 12 | 12 | 12 | 13 | 14 | 15 |

Tallennetaan avain taulukkoon.
| **a0** | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |

Kasvatetaan histogrammin arvoa yhdellä. Seuraavan kerran samalla indeksillä saadaan siis seuraava paikka taulukosta. 
Tällä tavoin kaikki avaimet menevät suoraan oikeaan paikkaan. Suoritetaan ns. laskentalajittelu (counting sort).

| 0 | **1** | 5 | 9 | 9 | 9 | 9 | 10 | 11 | 11 | 12 | 12 | 12 | 13 | 14 | 15 |

Sijoitetaan avaimet tulos taulukkoon edellisen esitetyllä tavalla.

a1 :: | **0010** 0110 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|1|**5**|9|9|9|9|10|11|11|12|12|12|13|14|15|
|taulukko      |a0|-|-|-|-|**a5**|-|-|-|-|-|-|-|-|-|-|
|histogrammi + 1|0|1|**6**|9|9|9|9|10|11|11|12|12|12|13|14|15|

a2 :: | **1101** 0010 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|1|6|9|9|9|9|10|11|11|12|12|12|**13**|14|15|
|taulukko      |a0|-|-|-|-|a5|-|-|-|-|-|-|-|**a2**|-|-|
|histogrammi + 1|0|1|6|9|9|9|9|10|11|11|12|12|12|**14**|14|15|

a3 :: | **0001** 0000 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|**1**|6|9|9|9|9|10|11|11|12|12|12|14|14|15|
|taulukko      |a0|**a3**|-|-|-|a5|-|-|-|-|-|-|-|a2|-|-|
|histogrammi + 1|0|**2**|6|9|9|9|9|10|11|11|12|12|12|14|14|15|

a4 ::| **0001** 1111 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|**2**|6|9|9|9|9|10|11|11|12|12|12|14|14|15|
|taulukko      |a0|a3|**a4**|-|-|a5|-|-|-|-|-|-|-|a2|-|-|
|histogrammi + 1|0|**3**|6|9|9|9|9|10|11|11|12|12|12|14|14|15|

a5 ::| **1100** 0011 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|3|6|9|9|9|9|10|11|11|**12**|12|12|14|14|15|
|taulukko      |a0|a3|a4|-|-|a5|-|-|-|-|-|-|a5|a2|-|-|
|histogrammi + 1|0|3|6|9|9|9|9|10|11|11|**13**|12|12|14|14|15|

a6 ::| **0010** 0110 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|3|**6**|9|9|9|9|10|11|11|13|12|12|14|14|15|
|taulukko      |a0|a3|a4|-|-|a5|**a6**|-|-|-|-|-|a5|a2|-|-|
|histogrammi + 1|0|3|**7**|9|9|9|9|10|11|11|13|12|12|14|14|15|

a7 ::| **0110** 0110 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|3|7|9|9|9|**9**|10|11|11|13|12|12|14|14|15|
|taulukko      |a0|a3|a4|-|-|a5|a6|-|-|**a7**|-|-|a5|a2|-|-|
|histogrammi + 1|0|3|7|9|9|9|**10**|10|11|11|13|12|12|14|14|15|

a8 ::| **1001** 1010 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|3|7|9|9|9|10|10|11|**11**|13|12|12|14|14|15|
|taulukko      |a0|a3|a4|-|-|a5|a6|-|-|a7|-|**a8**|a5|a2|-|-|
|histogrammi + 1|0|3|7|9|9|9|10|10|11|**12**|13|12|12|14|14|15|

a9 ::| **0010** 1110 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|3|**7**|9|9|9|10|10|11|12|13|12|12|14|14|15|
|taulukko      |a0|a3|a4|-|-|a5|a6|**a9**|-|a7|-|a8|a5|a2|-|-|
|histogrammi + 1|0|3|**8**|9|9|9|10|10|11|12|13|12|12|14|14|15|

a10 ::| **1111** 0000 |

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|3|8|9|9|9|10|10|11|12|13|12|12|14|14|**15**|
|taulukko      |a0|a3|a4|-|-|a5|a6|a9|-|a7|-|a8|a5|a2|-|**a10**|
|histogrammi + 1|0|3|8|9|9|9|10|10|11|12|13|12|12|14|14|**16**|

a11 :: |**0001** 0001|

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|**3**|8|9|9|9|10|10|11|12|13|12|12|14|14|16|
|taulukko      |a0|a3|a4|**a11**|-|a5|a6|a9|-|a7|-|a8|a5|a2|-|a10|
|histogrammi + 1|0|**4**|8|9|9|9|10|10|11|12|13|12|12|14|14|16|

a12 :: |**0001** 0100|

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|**4**|8|9|9|9|10|10|11|12|13|12|12|14|14|16|
|taulukko      |a0|a3|a4|a11|**a12**|a5|a6|a9|-|a7|-|a8|a5|a2|-|a10|
|histogrammi + 1|0|**5**|8|9|9|9|10|10|11|12|13|12|12|14|14|16|

a13 :: |**1110** 0011|

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|5|8|9|9|9|10|10|11|12|13|12|12|14|**14**|16|
|taulukko      |a0|a3|a4|a11|a12|a5|a6|a9|-|a7|-|a8|a5|a2|**a13**|a10|
|histogrammi + 1|0|5|8|9|9|9|10|10|11|12|13|12|12|14|**15**|16|

a14 :: |**0010** 0110|

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|5|**8**|9|9|9|10|10|11|12|13|12|12|14|15|16|
|taulukko      |a0|a3|a4|a11|a12|a5|a6|a9|**a14**|a7|-|a8|a5|a2|a13|a10|
|histogrammi + 1|0|5|**9**|9|9|9|10|10|11|12|13|12|12|14|15|16|

a15 :: |**0111** 0110|

|taulukon indeksi|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|
|----------------|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|
|histogrammi   |0|5|9|9|9|9|10|**10**|11|12|13|12|12|14|15|16|
|taulukko      |a0|a3|a4|a11|a12|a5|a6|a9|a14|a7|**a15**|a8|a5|a2|a13|a10|
|histogrammi + 1|0|5|9|9|9|9|10|**11**|11|12|13|12|12|14|15|16|

Alkuperäisestä histogrammista saadaan bucketien indeksit ja siihen kuuluvien avainten lukumäärät.

|bucket indeksi|1|2|6|7|9|12|13|14|15|
|------------ |---|---|---|---|---|----|----|----|----|
|taulukko     |a0 , a3 , a4 , a11 , a12|a5 , a6 , a9 , a14|a7|a15|a8|a5|a2|a13|a10|

Luvut ovat nyt 4:n eniten merkitsevän bitin mukaan järjestyksessä.
Kantalukulajittelut täytyy vielä suorittaa ensimmäiselle ja toiselle
bucketille. Laskentalajittelu etenee siten rekursiivisesti kunnes ali bucketeja
ei enää synny. Tällöin avaimet ovat järjestetty.

## Algoritmin vakaus/epävakaus

LSD radix sort alkoittaa lastentalajittelun vähiten merkitsevästä luvusta ja
etenee kohti eniten merkitsevää lukua. Tällä tavalla toteutettuna algoritmin on
oltava vakaa. Tämä tarkoittaa sitä että alkuperäisten, saman luvun omaavat
avainten täytyy säilyttää keskenäisen järestyksen myös laskentalajittelun
jälkeen. Eli jos on kaksi saman luvun omaavaa avainta, täytyy alkuperäinen
järjestys ottaa huomioon.

Tässä artikkelissa esitetty algoritmi on MSD radix sort, ja se on luonteeltaan
epävakaa. Toisin sanoen kahden saman luvun omaavaa avainten alkuperäisellä
järjestyksellä ei ole merkitystä. Tämä mahdollistaa GPU:lla säieryhmien välisen
atomisten jaetunmuistin käytön. Tämä mahdollistaa myös 8-bittisten lukujen
käytön laskentalajittelussa. Ethan Steelen esityksessä
(https://slideplayer.com/slide/15106242/ käy ilmi että rinnakkaistetussa LSD
lajittelussa hyödynnetään säikeiden privaatteja histogrammeja joista sitten
koostetaan lopullinen histogrammi.  Tämä tarkoittaa sitä että histogrammin
kasvaessa rekistereiden lukumäärä kasvaa eksponentiaalisesti. Tästä syystä LSD
radis sortit ovat rajautuneen 5-bitin lukuihin mikä puolestaan lisää
muistioperaatioiden määrää.

## Laskentalajittelu ja paikallinen lajittelu

Tämän artikkelin algoritmi toimii kahdella eri tavalla. Suuremmille määrille
avaimia suoritetaan laskentalajittelu. Tämä on kuitenkin tehotonta kun
buckettien määrää kasvaa. Tästä syystä kun lajiteltavan bucketin avaimet
mahtuvat jaettuun muistiin, ne lajitellaan paikallisella lajittelulla (*engl.
local sort*). Tekijät eivät mainitse mitä lajittelualgoritmia he käyttävät
paikallisessa lajittelussa, mutta esimerkiksi bitonic sort on ainakin tähän
referaatin kirjoittajan mielestä hyvä GPU:lle sopiva lajittelualgoritmi
pienempien avainmäärien lajitteluun. 

Laskentalajittelu jakaa bucketit ali-buckteihin, ja kun järjestettävä data on
riittävän pieni, se järjestetään paikallisella lajittelulla. Algoritmi alkaa
laskentalajittelulla joka jakaa järjestettävän data eniten merkitsevän luvun
mukaan **r = 2^k** ali-bucketeihin siten että jokainen bucketiin kuuluva avain
omaa saman eniten merkitsevän luvun. Ali-bucketien kohdalla otetaan seuraavaksi
eniten merkitsevät luvut laskentalajitteluun. Kukin ali-bucket jaetaan jälleen
uusiin ali-bucketeihin tai järjestetään paikallisella lajittelulla. 

Lajittelu on valmis kun kaikki vähiten merkitsevän luvun bucketit ovat laskettu
ja/tai data on järjestetty paikallisella lajittelulla. 

Paikallinen lajittelu voidaan tehdä jaetussa muistissa, mutta laskentalajittelu
tarvitsee ylimääräisen taulukon jonne hajautetut avaimet kirjoitetaan. Tässä
algoritmissa käytetään kahta samankokoista taulukkoa ns. kaksoispuskurointina.
Edellisen vaiheen tulokset tallenetaan toiseen taulukkoon, jota sitten
käytetään seuraavan vaiheen syöttönä. Näin pyritään kierrättämään samoja
taulukoita algoritmin suorituksen aikana. Nämä kaksi taulukkoa vaikuttavat
toisiinsa, ja viimeisimpänä lajittelun kohteena oleva taulukko sisältää koko
algoritmin lopputuloksen. 

## Merkintöjä


