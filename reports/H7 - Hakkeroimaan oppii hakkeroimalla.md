# H7 - Hakkeroimaan oppii hakkeroimalla
##### Pentesting course taught by Tero Karvinen @ Haaga-Helia

## x) Lue ja tiivistä
Tehtävänä oli etsiä itsenäisesti kurssin aiheisiin liittyvä JUFO-arvioitu _review_-artikkeli.

- Julkaisija: IEEE
- JUFO-taso: 1
- Valittu artikkeli: [Automated Penetration Testing, A Systematic Review](https://ieeexplore.ieee.org/abstract/document/10278377)

Artikkeli on 8 sivua pitkä, joten tiivistelmä pohjautuu enimmäkseen silmäilyyn.
- Artikkelin päämäärä on kuvata tunkeutumistestauksen elinkaari alusta loppuun, sekä erot manuaalisen ja automatisoidun tunkeutumistestauksen välillä
- Elinkaaren "Tiedonkeruu"-vaiheesta mainitaan, että tiedon löytäminen kohdeorganisaatioista on nykyään helpompaa sosiaalisen median ja erilaisten työkalujen ansiosta. Mainittuja työkaluja ovat CrunchBase-tietokanta, SAM ja GSA eLibrary sekä Whois-tietokanta
- Sisältää tiivistetyn taulukon manuaalisen ja automatisoidun pen-testauksen eroista. Erot ovat erittäin yksinkertaistettuja
- Samoin kuin sovellusten testauksessa, pen-testauksessa on kolme erilaista testaustapaa:
  - Black box penetration testing = Murtautuminen ilman aiempaa tietoa järjestelmästä
  - White box penetration testing = Murtautuminen niin, että hyökkääjä tuntee lähdekoodin
  - Grey box penetration testing = Jonkin verran tietoa järjestelmän toiminnasta ja rakenteesta
- Lyhyt analyysi seitsemästä hyökkäystyökalusta: Nmap, Burpsuite, Wireshark, Metasploit, Nessus, Intruder, Netsparker
- Artikkelin johtopäätös: Automatisoitu ja manuaalinen tunkeutumistestaus täydentävät toisiaan. Tulevaisuudessa on oletettavaa että tekoälymallit yleistyvät alalla

## a) PortSwigger Academy harjoitteluita
###  Username enumeration via different responses

Tarkoituksena on siis tunnistaa olemassa oleva käyttäjätunnus, ja käyttäen brute forcea löytää käyttäjän salasana.

Näillä tiedoilla avasin tietenkin palvelussa "My account" sivun, ja kokeilin kirjautumista saadakseni tutkittavaa liikennettä:

![Login](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-06%20215818.png)

Eli virheellisillä tunnuksilla kirjautuminen antaa vain virheen "Invalid username". Tästä päättelin, että mikäli tunnus **olisi** palvelun tietokannassa, tulisi herjaus vain salasanasta.

Tätä tehtävää varten piti googlettaa kuinka ZAProxylla suoritetaan brute force hyökkäyksiä tunnuksia ja salasanoja kohtaan. ZAProxylla sattuikin olemaan juuri tähän sopiva artikkeli: [PortSwigger Labs: Username Enumeration with ZAP Scripts](https://www.zaproxy.org/blog/2022-04-14-portswigger-lab-username-enumeration-with-zap-scripts/).

![Zest Script](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-06%20220658.png)

![Scripts tab](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-06%20220836.png)

Ohjeessa sanotaan, että vastauksen koko olisi 3011 tavua, mutta ZAProxyssä näkyy sen olevan 3140. Mikäli tätä pitäisi muuttaa, se tehtäisiin tuplaklikkaamalla "Assert - Length..." ja muokkaamalla "Length" parametria halutuksi.

Seuraavaksi luodaan POST-pyynnön ympärille loop, joka suorittaa ns. fuzzauksen. Annetaan muuttujalle nimi kuten "fuzz", ja valitaan tiedosto jossa on lista käyttäjiä. PortSwiggerin labraympäristöön on olemassa oma lista käyttäjiä: https://portswigger.net/web-security/authentication/auth-lab-passwords

![Loop file](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-06%20221324.png)

Jäljellä on vielä fuzz-muuttujan lisääminen itse POST-pyyntöön. Elikkä tuplaklikataan POST, ja lisätään "username=" perään {{fuzz}} (mikäli oma muuttujasi on eri nimellä niin huomioi se tässä):

![User Fuzz](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-06%20221746.png)

Nyt kun ajan scriptin, aukeaa ZAPin alareunan ikkunaan uusi välilehti "Zest Results". Tästä pystyy seuraamaan scriptin tekemiä pyyntöjä:

![Zest Results](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-06%20222239.png)

Ja korostettu pyyntö onkin erityinen: muiden vastauksien koko on 3,140, mutta yksi on 3,142. Avataan vastaus ja nähdään, että käyttäjätunnus ei olekaan invalidi:

![Incorrect password](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-06%20222336.png)

Nyt palapelin ensimmäinen pala on löydetty, käyttäjätunnus on ```af```.

Salasanan löytäminen menee samalla kaavalla:
- Vaihdetaan loopin tiedosto salasana tiedostoon, joka löytyy myös tehtävänannosta
- Muokataan POST-pyynnön body seuraavanlaiseksi: ```username=af&password={{fuzz}}```

Ajoin scriptin uudestaan, ja jälleen yksi vastaus poikkesi muista:

![Salasana löytyi?](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-06%20223851.png)

Pyynnön sisältö oli seuraava: ```username=af&password=princess```, eli nyt kokeillaan manuaalisesti ovatko nämä toimivat tunnukset.

![Solved](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-06%20224053.png)

Palvelun luonteesta riippuen, tämän jälkeen voisi vaihtaa käyttäjän sähköpostiosoitteen palvelussa, salasanan tai löytää mahdollisesti pankkitietoja. Realistisessa tilanteessa millään palvelulla ei kuitenkaan ole näin huonoa suojausta toistuvia kirjautumisyrityksiä kohtaan :)

# Lähteet:
- Bahaa-Eldin, A. M., ElSayad, D., Fayed, Zt. & Saber, V. 2023. Automated Penetration Testing, A Systematic Review. Luettavissa: https://ieeexplore.ieee.org/abstract/document/10278377. Luettu: 11.12.2024
- PortSwigger. Lab: Username enumeration via different responses. Luettavissa: https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses. Luettu: 06.12.2024
- ZAP, 2022. PortSwigger Labs: Username Enumeration with ZAP Scripts. Luettavissa: https://www.zaproxy.org/blog/2022-04-14-portswigger-lab-username-enumeration-with-zap-scripts/. Luettu: 06.12.2024
