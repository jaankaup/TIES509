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
| --------------- | -------------------      |
| Radix Sort      | Kantalukulajittelu       |
| LSD             | Vähiten merkitsevä bitti |
| MSD             | Eniten merkitsevä bitti  | 

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
| 00010000 | 00001010 | 11111000 | 10101000 |

**k** = 32, **d** = 8, **r** = 256

Esimerkki 32 bittisestä avaimesta joka on esitetty 8-bittisten lukujen jonona.
Kunkin osa-jonon kantaluku r on 2^8 = 256.

Avaimet voidaan käydä läpi kahdella eri tavalla: eniten merkisevästä
**d-bittisestä** luvusta kohti vähiten merkitsevää lukua (**MSB radix sort**)
tai päin vastaisessa järjestyksessä (**LSB radix sort**).

MSD aloittaa lajittelun eniten merkitevästä **k-bittisestä** luvusta ja jakaa
avaimet **r** erilliseen osaan arvonsa mukaan. Tämä tehdään niin sanotun
laskentalajittelu (*engl. counting sort*) avulla siten että että aloitetaan
laskemaan **k-bittisten** lukuen esiintyvyyksiä sitä vastaavaan histogrammiin.
Laskentalajittelun jälkeen lasketaan histogrammista alkusumma (*engl. exclusive
prefix sum*) joka antaa kunkin luvun aloitus sijainnin seuraavaa vaihetta
varten. Histogrammi kertoo sen kuinka monta mitäkin k-bittistä lukua on
yhteensä ja alkusummat kertovat vastaavasti sen sijainnin lopullisessa
taulukossa johon luku sijoitetaan. Lopuksi avaimet sijoitetaan osiinsa (bucket)
**k-bittisen** avaimensa perusteella. Tätä prosessia jatketaan rekursiivisesti
seuraaville **k-bittisille** luvuille ja lopputuloksena saadaan järjestettyjen
lukujen jono.



## Blah

Artikkelin mukaan suurin onglema GPU:lla suoritettaville radix sort
algoritmeille on muistinsiirto-operaatiot. Olemassa olevat radix sort
algoritmit  


 Tämän artikkelin algoritmi puolittaa
tarvittavien muistisiirtojen lukumäärää ja tuo tällä tavalla nopeutusta
suoritusnopeuteen.

## Radix Sort

Tässä kappaleessa on yleistä 
