# H0 - Sanoma haaviin

Tässä raportissa sieppaan verkkoliikennettä WireShark-sovelluksella, ja analysoin sitä. Raportin aikana puhun sekä kehyksistä että paketeista, jotka muuten ovat kaksi erillistä konseptia mutta WireSharkia käytettäessa näitä termejä käytetään samassa kontekstissa.

Analysointia varten päätin siepata Wi-Fi adapterin liikennettä. Tätä varten olin yhdistänyt Eduroam julkiseen verkkoon, joka on opiskelijoiden käytössä.

## Analyysi

Alhaalla olevassa kuvassa on näkymä, jonka WireShark antaa kaappauksen edetessä. Ensimmäisenä näkyvät tiedot kaapatuista IP-paketeista ovat:

- Järjestysnumero
- Aika joka on kulunut paketin kaappauksen ja kaappauksen aloituksen välillä (Ensimmäinen paketti on napattu heti prosessin käynnistyttyä)
- Lähde IP-osoite, eli mistä IP-osoitteesta paketti on kaappauksen ajankohtana matkalla
- Kohde IP-osoite, eli mikä on IP-paketin kohdelaitteen IP-osoite
- Mitä protokollaa paketin siirtämiseen käytetään
- Kaapattujen tavujen määrä
- Lisätietoja, kuten lähde ja kohde porttien numerot, ja viestin tyyppi

![WireShark kaappauksen näkymä](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/kaappaus.png)

Seuraavaksi valitsen yhden kaapatuista paketeista, ja analysoin siitä saatuja tietoja. Liikun listaa ylhäältä alaspäin järjestyksessä, ja valitsen ne tiedot jotka näen tärkeiksi

- Encapsulation type = Kehys on kapsuloitu Ethernet protokollalla
- Frame Lenght = Kehys on 120 tavua pitkä
- Protocols in frame = Mitä protokollia kehyksessä on käytetty
- Destination = Kehys on matkalla Ciscon reitittimeen, jonka MAC-osoite on 00:13:80:95:d3:c4
- Source = Kehys on viimeksi lähtenyt LiteonTechno verkkolaitteesta jonka MAC-osoite on 14:5a:fc:50:40:33

![Kaapattu-Kehys1](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/kaapattu-paketti1.png)

- Internet Protocol Version 4 = Tässä osiossa on kehyksen ylätunnisteen (header) tiedot, eli millä tiedoilla reitittimet käsittelevät kehystä
- Source Port: Kehys on lähtenyt portista 60259
- Destination Port: Kehys on matkalla porttiin 443
- TLSv1.2 Record Layer: Hypertext Transfer Protocol, eli HTTP
- Content Type: Kehys on sovellus dataa
- Encrypted Application Data: Kehyksen hyötykuorma on enkryptattu jottei sitä pysty lukemaan selkokielellä

![Kaapattu-kehys2](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/kaapattu-paketti2.png)
