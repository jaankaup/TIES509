| Pvm               | Artikkeli             | Käytetty aika | Kokonais aika |
| ----------------- | --------------------- | ------------- | ------------- |
| 2020.10.31        | Radix Sort            | 2h            | 2h            |
| 2020.11.14        | Radix Sort            | 2h            | 4h            | 
| 2020.11.16        | Radix Sort            | 8h            | 12h           |Lopputuloksena saadaan järjestettyjen lukujen jono.

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

| b0 == 0 | b1 == a0 | b2 == (a0 + a1) | ... | b255 == (a0 + a1 + a2 + ... + 255)] |
| ------- | -------- | --------------- | --- | ----------------------------------- |

 Tämä

