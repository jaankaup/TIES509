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

## Esipuhe

Tämän kirjareferaatin pääasiallinen tarkoitus on esittää artikkelin sisältö
sillä tarkkuudella että lukija ymmärtää ja pystyy itse toteuttamaan artikkelin
mukaisen rinnakkaistetun kantalukulajittelun. Erityisesti GPU:lla toteutettava
kantalukulajittelu on tarpeellista monessa eri kontekstissa ja itse näen
tarpeelliseksi sen että ohjelmoijan on hyvä osata toteuttaa tehokas lajittelu
algoritmi itse.  

## Johdanto

Tässä artikkelissa esitetään uusi näytönohjaimella suoritettava radix sort
algoritmi. Algoritmi kykenee lajittelemaan 2Gb 8 tavuisia tietuita 50
millisekunnissa ja toimii myös niin sanotusti out-of-core algoritmina, eli se
pystyy järjestämään myös dataa joka ei mahdu kerrallaan näytönohjaimen
muistiin. Algoritmi on suunniteltu muistin käytön suhteen tehokkaaksi.

 "Artikkelin kirjoittajat sanovat myös kehittäneensä keinon
vähentääkseen CPU:n ja GPU:n välisen dataliikenteen (PCIe) viivettä."

## Kantalukulajittelu

Kantalukulajittelu perustuu k-bittisten avaimen tulkitsemista d-bittisten
lukujen jonona. Perus idea on se että jaetaan k-bittinen luku riittävän pieniin
d-bittisiin lukuihin siten että kantaluku r = 2^d ja että avaimet voidaan jakaa tehokkaasti 
r erillisiin osiin (engl. bucket). 

| 0001000 | 00001010 | 11111000 | 10101000 |
| ------- | -------- | -------- | -------- |

## Blah

Artikkelin mukaan suurin onglema GPU:lla suoritettaville radix sort
algoritmeille on muistinsiirto-operaatiot. Olemassa olevat radix sort
algoritmit  


 Tämän artikkelin algoritmi puolittaa
tarvittavien muistisiirtojen lukumäärää ja tuo tällä tavalla nopeutusta
suoritusnopeuteen.

## Radix Sort

Tässä kappaleessa on yleistä 
