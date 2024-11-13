# H2 - Social Sploit
##### Pentesting course taught by Tero Karvinen @ Haaga-Helia

## x) Lue ja tiivistä
### Jaswal, N 2020. Mastering Metasploit. Chapter 1: Approaching a Penetration Test Using Metasploit
- Yleistä termistöä Metasploitia käytettäessä:
  - Exploits = Koodinpätkä joka hyödyntää kohteen haavoittuvuutta
  - Payload = Hyötykuorma on koodinpätkä joka suoritetaan kohteen sisällä exploitin jälkeen
  - Auxilary = Moduuleja jotka tarjoavat lisätoimintoja, kuten skannausta, fuzzing-testausta ja liikenteen sieppausta
  - Encoders = Näitä käytetään moduulien piilottamiseen, jotta suojaustyökalut eivät tunnistaisi niitä
  - Meterpreter = Hyötykuorma, joka toimii tietokoneen RAM-muistissa. Tällöin mitään ei tallennu kovalevylle, ja hyökkäyksen tunnistaminen on hankalampaa
 - Metasploit open-source, eli lähdekoodi on avoin. Tämä mahdollistaa Metasploitin jatkuvan kehityksen, sekä omien moduulien luomisen
 - Kerätessä informaatiota kohteesta esim. Nmap:in avulla, on hyvä käyttää Metasploitin sisäistä tietokantaa kerättyjen tietojen säilyttämiseen

## a) Kali & Metasploitable yhteisessä verkossa
Koska tunkeutumistyökaluja käytettäessä on tärkeää, että et vahingossa tunkeudu ulkopuolisiin koneisiin, harjoitukset toteutetaan kahdella virtuaalikoneella jotka eivät saa yhteyttä verkkoon, mutta niiden välille on luotu verkkoyhteys.

Kali:

![Kali koneen verkkoyhteydet](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/kali-network.png)

Metasploitable2:

![Metasploitable2-koneen verkkoyhteydet](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/metasploitable-network.png)

## b) Metasploit msfconsole
Metasploit-työkalua voidaan käyttää Linuxin komentoriviltä komennolla ```msfconsole```.

![Metasploit msfconsole](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/msfconsole.png)

## c) Etsitään Metasploitable porttiskannaamalla
Koska kohteesta kerätty tieto on hyvä tallentaa tietokantaan, varmistan ensin että Metasploitin tietokanta on toiminnassa. Varmistin että Metasploitin käyttämä PostgreSQL-palvelin on käynnissä käyttämällä komentoa ```sudo service postgresql start```, jonka jälkeen alustin tietokannan komennolla ```sudo msfdb init```. Löysin tähän ohjeet Mitesh Shahilta (Shah, M. 2015)

![Tietokannan käynnistys ja alustus](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/database-connection.png)

Seuraavaksi siirryin Metasploitin puolelle, jossa käynnistin porttiskannauksen käyttäen kohdekoneen IP-osoitetta. Varmistin vielä IP-osoitteen oikeaksi selaimen avulla. Skannauksen suoritin komennolla ```db_nmap -sn 192.168.30.103```.

![DB_Nmap](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/db_nmap-metasploit.png)

## d) Metasploitablen perusteellinen porttiskannaus
Skannasin Metasploitablen agressiivisesti komennolla ```db_nmap 192.168.30.103 -A -p-```, tallentaen samalla saadut tiedot tietokantaan.

Suoritin saman skannauksen pelkällä Nmapilla komennolla ```nmap 192.168.30.103 -A -p- -oA foo```, joka tallentaa skannatut tiedot tiedostoon "foo".

## e) Metasploitin tietokannan tarkastelu
Komennoilla ```hosts``` ja ```services``` tulostuu kerätyt tiedot kohteesta. Tulosteessa näkyy hostin IP-osoite, skannattu portti, sen nimi, status (open/closed) ja siitä saatuja tietoja. Tietokannasta pystyy etsimään esim. tiettyä porttia komennolla ```services --port 80```, jolloin näkyy vain portti 80. Komennolla ```services --search``` pystyy etsimään esimerkiksi tiettyä sanaa kuten "MySQL".

![Tietokanta tuloste](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/db_nmap-tuloste.png)
![Tietokanta tuloste 2](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/db_nmap-tuloste2.png)

## f) Metasploitin tietokannan ja Nmapin tiedostojen vertailu
Nmapin skannauksen tulos tallentuu kolmeen tiedostoon, .gnmap, .xml ja .nmap. Näistä hyödyllisin on mielestäni .nmap tiedosto, jossa skannauksen tulokset ovat tallentuneet samaan formaattiin kuin missä ne näkyvät terminaalissa komennon suorittamisen ohella.

Tähän verrattuna Metasploitin tietokanta on paljon helppolukuisempi, mutta Nmapin tiedostoissa on reilusti enemmän tietoa joka on hyödyllistä. Nmapin tiedostoista pystyy etsimään helposti tietoa esim grepillä.

## g) Metasploitablen vsftpd-palveluun murtautuminen
Seuraten Nipun Jaswalin kirjassa esitettyjä työvaiheita, loin ensimmäiseksi uuden workspacen komennolla ```workspace -a metasploitable```. Kaikki nämä vaiheet eivät välttämättä olisi tarpeellisia tähän harjoitukseen, mutta haluan suorittaa kaiken ns. "oikein"

Tämän jälkeen suoritin komennon ```search vsftpd```, jolla löytyy kaksi mahdollista moduulia. Rivillä 0 oleva on DoS-hyökkäys ja rivillä 1 on ns. backdoor hyökkäys, eli takaovi josta pääsee sisälle järjestelmään.

Komennolla ```use 1``` valitsen backdoor-hyökkäyksen, ja komennolla ```set RHOSTS 192.168.30.103``` asetan kohteeksi Metasploitable-koneen. Tämän jälkeen ajan komennon ```exploit```, joka luo minulle Shell-yhteyden. Komennolla ```whoami``` näen olevani sisällä 'root'-tunnuksilla.

![Vsftpd exploitin alustus](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/exploit-vsftpd.png)

![Vsftpd Shell yhteys](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/exploit_shell-connection.png)

## h) From shell to Meterpreter
Päivitin shell-yhteyden Meterpreter-yhteydeksi asettammalla yhteyden taustalle pikanäppäimellä **Ctrl+Z**, jonka jälkeen ajoin komennon ```sessions -u 1```.

![Shellistä Meterpreteriin](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/shell-meterpreter.png)

## i) Lateral movement
Kansiosta /home löytyy paikalliset käyttäjät. Komennolla ```cat /etc/shadow``` saadaan selville käyttäjien hashatut salasanat. Näitä voidaan yrittää crackata joko sanakirjahyökkäyksellä tai brute forcella. passwd ja shadow tiedostot voidaan ladata hyökkäävälle koneelle 

```ifconfig``` ja ```route``` Näyttävät kohteen verkon rajapinnat, IP-osoitteet ja reititystaulun. Näillä tiedoilla voitaisiin laajentaa hyökkäystä verkon muihin laitteisiin, mikäli niitä on.

![Verkkotiedot](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/metasploitable-ifconfig-ja-route.png)

Ajamalla moduulin "local_exploit_suggester" komennolla ```run post/multi/recon/local_exploit_suggester```, saan kerättyä tietoa mitä muita haavoittuvuuksia voin hyödyntää.

![Local exploit suggester](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/local_exploit_suggester.png)

## j) Murtautuminen toisella tavalla
Tätä tehtävää varten selasin Metasploitable2 walkthrough ohjeita, ja etsin mitä portteja suositellaan murtautumiseen. Päädyin valitsemaan Samban, eli portti 139.

Tunkeutuminen oli varsin samanlainen kuin edellisessä tehtävässä vsftpd:n avulla, asetin RHOST:iksi kohde koneen IP:n, mutta tällä kertaa piti asettaa myös LHOST, eli hyökkäävän koneen IP-osoite. Saatuani yhteyden, päivitin sen taas Meterpreteriksi.

## k) Meterpreterin ominaisuudet
Meterpreterin avulla pystytään ajamaan komentoja jotka paljastavat tietoja hyökkäyksen kohteesta, kuten ```sysinfo```, ```ifconfig``` ja ```arp```.

![Meterpreter ominaisuudet 1](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/meterpreter-ominaisuudet-1.png)

Meterpreterin avulla pystyy myös listaamaan kaikki kohdekoneen käynnissä olevat prosessit komennolla ```ps```, ja joko siirtää yhteyden vähemmän huomattavaan prosessiin komennolla ```migrate``` tai pysäyttämään prosesseja komennolla ```kill```

Kohdekoneelta saa myös ladattua tiedostoja suoraan hyökkäävälle koneelle komennolla ```download <TIEDOSTON POLKU>```

![Download](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/meterpreter-download.png)

## l) Shell-session tallentaminen tekstitiedostoon
Komento ```script -fa log001.txt``` aloittaa käytettyjen komentojen tallenuksen log001.txt-tiedostoon. Tästä tiedostosta pystyy hakemaan jälkikäteen kaikki käytetyt komennot ja saadut tulosteet. Ymmärtääkseni tämä kuitenkin pitää tehdä ennen Metasploitin käynnistämistä jos haluan tallentaa tunkeutumisen aikana käytetyt komennot. Tiedosto tallentuu automaattisesti käyttäjän kotihakemistoon paikallisella koneella. Scriptin ja komentojen tallennuksen voi lopettaa ```exit```-komennolla.

## Lähteet

- Jaswal, N 2020. Mastering Metasploit - Fourth Edition. Chapter 1: Approaching a Penetration Test Using Metasploit. Luettu: 06.11.2024
- Rapid7 s.a. Metasploitable 2 Exploitability Guide. Luettavissa: https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide. Luettu: 06.11.2024
- Shah, Mitesh 2015. How to Fix Metasploit Database Not Connected or Cache Not Built. Luettavissa: https://miteshshah.github.io/linux/kali/how-to-fix-metasploit-database-not-connected-or-cache-not-built/. Luettu: 06.11.2024
