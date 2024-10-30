# H1 - Hacker's Journey | Kill Chain #1

## x) Lue/Katso/Kuuntele ja tiivistä

### Herrasmieshakkerit - Suomen kyberpuolustus, vieraana Tuomo Rusila | 0x2e (Herrasmieshakkerit 2023)
- Puolustusvoimat varmistavat Suomen fyysisen turvallisuuden, mutta Suomen tietojärjestelmien turvallisuudesta ei vastaa mikään yksittäinen taho
- Laajempi tietoturvaosaaminen NATO:ssa tulee jäsenmailta, eikä niinkään NATO:n sisältä
- Tuomo Rusilan mukaan hyökkäyskyvykkyys on osa uskottavaa kyberpuolustusta
- Suuri osa Suoman kyberpuolustuksen voimavaroista on reservissä. Lisäksi Puolustusvoimilla on mahdollisuus hyödyntää myös niitä jotka eivät asepalvelusta ole suorittaneet
- Kyber on nykyään merkittävä osa laaja-alaista vaikuttamista, niin hyökkääjän kuin myös puolustajan osalta

### Hutchins et al 2011: Intelligence-Driven Computer Network Defense Informed by Analysis of Adversary Campaigns and Intrusion Kill Chains, chapters Abstract, 3.2 Intrusion Kill Chain. (Amin, Cloppert & Hutchings 2011)
- Tavalliset verkkojen puolustamisessa käytetyt työkalut kuten anti-virus eivät riitä kehittyneitä hyökkääviä osapuolia vastaan
- Hyökkääjien käyttämiin tekniikoihin ja erityisesti "tappoketjuun" tutustumalla, puolustava osapuoli kykenee pienentämään järjestelmän haavoittuvuutta
- Kill chain, eli tappoketju, on prosessi jolla hyökkääjä suunnittelee hyökkäyksen askel askeleelta alusta loppuun. Prosessia sanotaan ketjuksi, koska yhden "linkin" puuttuessa prosessi ei toimi

### € Santos et al: The Art of Hacking (Video Collection): 4.3 Surveying Essential Tools for Active Reconnaissance. (McCoy, Santos, Sternstein & Taylor 2019)
- Aktiivinen tiedustelu sisältää porttiskannausta ja haavoittuvuuksien kartoittamista
- Nmap on suosituin työkalu porttiskannaukseen, sillä se on monipuolinen ja sen käyttöön löytyy paljon ohjeita ja dokumentaatioita
- Avointen porttien lisäksi aktiivisessa tiedustelussa kartoitetaan kohteen käyttämiä verkkopalveluita

### KKO 2003:36. (Finlex 2003)
- Epäilty oli suorittanut porttiskannauksen OP:n osuuskunnan tietojärjestelmiä kohtaan, mutta palomuuri oli estänyt tämän
- Julkisessa verkossa tapahtuva porttiskannaus on rikos, vaikka skannauksessa saatujen tietojen avulla ei ole tarkoitus tunkeutua

## a) Kali Linux asennus
Latasin Kali Linux 2024.3 64-bittisen version [täältä](https://www.kali.org/get-kali/#kali-installer-images). Loin virtuaalikoneen VirtualBoxilla, jossa asetin käyttöjärjestelmäksi Linuxin, ja versioksi Debianin kuten Kalin omassa dokumentaatioissa (Kali Org, 2024). 

Loppuliset speksit näkyvät alhaalla:

![Kali-koneen speksit](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Kali-speksit.png)
  
## b) Kali-virtuaalikoneen irroittaminen verkosta
Irroitan luomani Kali-virtuaalikoneen verkosta menemällä VirtualBoxin asetuksiin, valitsemalla **Network**->**Advanced**->**[ ]Cable Connected**

![Kali verkkoasetukset](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Kali-verkkoasetukset.png)

Tämän jälkeen tarkistin, että virtuaalikoneella ei ole verkkoyhteyttä kokeilemalla pingata Googlen DNS osoitetta 8.8.8.8. Lopputuloksena on odotettu "Network is unreachable".

![Kali ei verkkoa](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Kali-ei_verkkoa.png)

## c) 1000 tavallisimman tcp-portin skannaus
Tehtävänannon sanavalinta "omasta koneestasi" aiheutti hieman hämmennystä, joten päätin porttiskannata luomani Kali-virtuaalikoneen, sekä tätä tehtävää varten Vagrantilla luomani koneen.

Kali-virtuaalikoneen porttiskannauksen lopputulos paljastaa että 1000 tavallisinta tcp-porttia on suljettu, ja tämä johtuu siitä että kone ei ole yhdistettynä verkkoon.

![Kali tcp-scan](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Kali-tcp_scan.png)

Vagrant-virtuaalikoneen porttiskannaus paljastaa yhden avoinna olevan tcp-portin, sillä tässä koneessa on verkkoyhteys tallella. Avoinna oleva portti on portti 22/SSH, koska Vagrant-koneeseen on otettu yhteys SSH:n avulla.

![Vagrant tcp-scan](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Vagrant-tcp_scan.png)

## d) Porttiskannaus demonien asennuksen jälkeen
Tätä tehtävää varten asensin Vagrant-koneelleni kaksi demonia, Apache-palvelimen ja Nginx HTTP-palvelimen. Localhost porttiskannauksen jälkeen erona näkyy, että portin 22 lisäksi avoinna on myös portti 80, jolla on yhteys HTTP-palveluun. Tämä johtuu siitä että sekä Apache ja Nginx käyttävät HTTP-yhteyksiä.

![Vagrant toinen tcp-scan](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/vagrant-tcp_scan2.png)

## e) Metasploitable 2 asennus
Latasin Metasloitable 2 asennukseen tarvittavat tiedostot [täältä](https://sourceforge.net/projects/metasploitable/). Asennustiedoston lataamisessa kesti itselläni n. 40 minuuttia, ja verkkoyhteyteni on suhteellisen nopea (400Mpbs), joten tähän kannattaa varata aikaa.

Latauksen valmistuttua, zip-tiedosto pitää purkaa. Tämän jälkeen luodaan uusi virtuaalikone. Tähän käytin Geeks for Geeksin ohjetta (Geeks for Geeks, 2022). Kovalevyä valittaessa en luonut uutta, vaan valitsin "Use an existing Virtual Hard Disk File", ja etsin purkamamme Metasploit kansion, josta löytyy **Metasploitable.vmdk**-virtuaalinen kovalevy.

Lisäksi asetin jo nyt virtuaalikoneen verkkoasetuksista Adapteri 1:n **Host-only Adapteriksi**, tämä tulee olemaan oleellista seuraavassa tehtävässä.

Virtuaalikoneen lopulliset speksit olivat seuraavat:

![Metasploit-koneen speksit](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Metasploit-specs.png)

Koneen käynnistyksen jälkeen kirjauduin sisään Metasploitin oletustunnuksilla:
- Username: msfadmin
- Password: msfadmin

## f) Virtuaaliverkko Kali-koneen ja Metasploit-koneen välille
### Day 1
Jotta pystyn tunkeutumaan Metasploit-koneelle Kali-koneeltani, näiden kahden koneen välille pitää luoda verkkoyhteys. Ne eivät kuitenkaan saa olla julkisessa verkossa, jotten vahingossa tee mitään laitonta.

Ensimmäisenä menin Kali-koneen verkkoasetuksiin, ja varmistin että Adapteri 1, joka oli yhdistettynä NAT-verkkoon oli kytketty pois päältä. Tämän jälkeen valitsin Adapteri 2:n, jonka asetin **Host-only adapteriksi**, ja varmistin että se on kytketty.

![Kali Host-only](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/kali-host_only.png)

![Metasploitable Host-only](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/metasploit-host_only.png)

Edellisessä tehtävässä asetin Metasploitable-koneen jo valmiiksi Host-only verkkoon, joten tämän hetkisillä tietotaidoillani näiden kahden koneen pitäisi nyt pystyä keskustelemaan keskenään, mutta jostain syystä tämä ei onnistu. Pingatessani Metasploitable-konetta Kali-koneelta, tulee vastaus "ping: connect: Network is unreachable".

Tässä vaiheessa palasin Valkamon artikkeliin virtuaalikoneiden asennuksista (Valkamo 2022). Kokeilin hänellä toiminutta ratkaisua luoda uusi adapteri, ja käytin tätä molemmilla koneilla. Tämäkään ei toiminut.

Lopputulos tässä vaiheessa on se, että mikäli Kali-koneella on yhteys verkkoon, pystyn sillä pingaamaan Metasploit-konetta, vaikka tässä olisi host-only network adapteri. Kun Kali-koneelta poistaa verkkoyhteyden, ei se saa enään yhteyttä Metasploit-koneeseen vaikka molemmat ovat samassa host-only verkossa.

### Day 2

Seuraavana päivänä asensin molemmat koneet uusiksi, seuraten muuten aiempia valintojani mutta ennen ensimmäistä käynnistämistä asetin molemmat koneet samaan Host-only verkkoon. Tämän jälkeen sain muodostettua yhteyden molempien koneiden välille. Mielestäni en tehnyt muita muutoksia asennusprosessin yhteydessä, mutta selkeästi jotain oli ensimmäisellä kerralla mennyt pieleen?

![Kali-Metasploit yhteys](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/kali-metasploit-ping.png)

## g) Metasploit-koneen etsiminen porttiskannauksella
Seuraavaksi yritin etsiä Metasploit-koneen käyttämällä **Nmap**-työkalua. Komennolla ```nmap -sn``` Nmap ei löytänyt yhtään avointa porttia, mutta asettamalla kohteeksi tiedossa olevan IP-osoitteen, eli tässä tapauksessa käytin komentoa ```nmap 192.168.30.103```, sain näkyville kaikki avoinna olevat portit Metasploit-koneesta.

```nmap -sn```-komento etsii saatavilla olevat IP-osoitteet, mutta ei suorita porttiskannausta heti löytämisen jälkeen (Nmap Options Summary).


## h) Metasploit-koneen tarkka porttiskannaus
Yksityiskohtaisempaan porttiskannaukseen käytin komentoa ```nmap 192.168.30.103 -A -p-```. Komennon '-A' ottaa käyttöön agressiivisen skannauksen jolla tunnistetaan tai selvitetään käyttöjärjestelmä, palvelujen versio ja traceroute-polku (Nmap s.a. a). '-p-' skannaa kaikki portit eli 1-65535 (Nmap s.a. b).

Skannauksen tulos oli seuraavanlainen:

![Nmap tarkka skannaus 1](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/nmap-full_scan1.png)
![Nmap tarkka skannaus 2](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/nmap-full_scan2.png)
![Nmap tarkka skannaus 3](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/nmap-full_scan3.png)

Mielestäni hyökkääjälle mielenkiintoisia portteja tästä valikoimasta voisivat olla portit 21 (FTP), 23 (Telnet) ja 3306 (MySQL).
- Port 21/FTP: "Anonymous FTP login allowed" viittaa siihen että koneeseen saa FTP-yhteyden ilman kirjautumista, mikä mahdollistaa hyökkääjälle helpon pääsyn tiedostoihin.
  - Ohjaus- ja datansiirto yhteydet eivät ole salattuja, jolloin hyökkääjä voi kaapata kirjautumistiedot tai tiedonsiirron sisällön
- Port 23/Telnet: Telnet mahdollistaa salaamattoman yhteyden, jolloin kirjautumistiedot voivat vuotaa helposti.
- Port 3306/MySQL: Mahdollisuus käyttää tietokantaa ja hyödyntää tunnettuja haavoittuvuuksia, kuten SQL-injektioita.  
  - Käytetty MySQL-versio on vanha, jolloin siinä saattaa olla tunnettuja haavoittuvuuksia

# Lähteet

- Amin, R., Cloppert, M. & Hutchins, E 2011. Intelligence-Driven Computer Network Defense Informed by Analysis of Adversary Campaigns and Intrusion Kill Chains. Luettavissa: https://lockheedmartin.com/content/dam/lockheed-martin/rms/documents/cyber/LM-White-Paper-Intel-Driven-Defense.pdf. Luettu: 30.10.2024
- Geeks for Geeks 2022. How to install Metasploitable 2. Luettavissa: https://www.geeksforgeeks.org/how-to-install-metasploitable-2-in-virtualbox/. Luettu: 29.10.2024
- Finlex 2023. KKO:2003:36. Luettavissa: https://finlex.fi/fi/oikeus/kko/kko/2003/20030036. Luettu: 30.10.2024
- Hyppönen, M. & Tuominen, T. joulukuu 2023. Suomen kyberpuolustus, vieraana Tuomo Rusila | 0x2e. Herrasmies Hakkerit-podcast. Kuunneltavissa: https://open.spotify.com/episode/2z1oltiq7JYOtIAXQr8yHa?si=2071790e4b2d4566. Kuunneltu 30.10.2024
- Kali Org 2024. Kali inside VirtualBox. Luettavissa: https://www.kali.org/docs/virtualization/install-virtualbox-guest-vm/. Luettu: 29.10.2024
- Kali Org s.a. Kali installer images. Luettavissa: https://www.kali.org/get-kali/#kali-installer-images. Luettu: 29.10.2024
- McCoy, C., Santos, O., Sternstein, J. & Taylor, R. 2019. Art of Hacking - Lesson 4: Active Reconnaissance. Video. Katsottavissa: https://learning.oreilly.com/videos/the-art-of/9780135767849/9780135767849-SPTT_04_00/. Katsottu: 30.10.2024
- Nmap Options Summary
- Nmap s.a. a. A Quick Port Scanning Tutorial. Luettavissa: https://nmap.org/book/port-scanning-tutorial.html#port-scanning-tutorial-nmap2. Luettu: 30.10.2024
- Nmap s.a. b. Command-line Flags. Luettavissa: https://nmap.org/book/port-scanning-options.html. Luettu: 30.10.2024
- Sourceforge 2019. Metasploitable. Luettavissa: https://sourceforge.net/projects/metasploitable/. Luettu: 29.10.2024
- Valkamo 2022. Hacking into a Target using Metasploit. Luettavissa: https://tuomasvalkamo.com/PenTestCourse/week-2/. Luettu: 29.10.2024
