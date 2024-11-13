# H3 - Nuuskija
##### Pentesting course taught by Tero Karvinen @ Haaga-Helia

## x) Lue ja tiivistä
### Popov 2024: Hacktricks: Wireshark tricks (HackTricks s.a)
- Yläreunan työkalupalkin valikoista löytyy työkaluja joiden avulla liikennettä pystyy tarkastella hyvin yksityiskohtaisesti
- Wiresharkin kaappaamaa liikennettä pystyy filtteröimään melko tarkasti eri protokollien perusteella
  - Protokolla kohtaisia filttereitä löytyy WireSharkin dokumentaatiosta: https://www.wireshark.org/docs/dfref/ 
- Pakettien sisältöä voi etsiä *CTRL + F* pikanäppäimellä
- Kaapatuista paketeista saa tietoon host-laitteen nimen hakemalla joko *DHCP* tai *NBNS*

### Karvinen 2023: Find Hidden Web Directories - Fuzz URLs with ffuf
- Verkkopalvelimilla on usein piilotettuja hakemistoja, joihin ei ole olemassa linkkiä
- Fuff on fuzzaus-työkalu joka etsii näitä hakemistoja automaattisesti sille annetun "sanakirjan" avulla
- Vastaavia "sanakirjoja" voi etsiä internetistä, tai tehdä itse oman

## a) Valmiin hyökkäyksen demonstrointi
Valitsin valmiiksi hyökkäykseksi ssh_login scannerin, jolla saa brute forcettua SSH-yhteyden, kunhan vaan löytää sanalistan jossa on toimivat tunnukset. Otin hyökkäyksen käyttöön komennolla ```use auxilary/scanner/ssh/ssh_login```, ja muutin seuraavat asetukset:

      set RHOSTS 192.168.30.103
      set userpass_file /usr/share/metasploit-framework/data/wordlists/piata_ssh_userpass.txt
      set stop_on_success true
      set verbose true
Minulla ei tietenkään ollut niin hyvä tuuri, että olisin saanut heti ensimmäisellä sanalistalla toimivat tunnukset, vaan niitä piti kokeilla muutama. Tähän raporttiin kuitenkin kirjasin sen mikä toimi. Lisäksi asetus ```verbose``` ei ole pakollinen, se tekee prosessista näkyvän mikä mielyttää itseäni.

![SSH_login options](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/SSH_login_asetukset.png)

Lopulta listasta löyty toimivat tunnukset "user:user".

![SSH_user_user_login](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/SSH_user_user_login.png)

Tämän jälkeen vahvistin SSH session olemassa olon, ja päivitin yhteyden meterpreteriin. Turvallista olettaa että "user" on käyttäjä vähillä oikeuksilla, mutta tästä olisi mahdollista levittäytyä verkkoympäristössä horisontaalisesti.

![SSH_login meterpreter](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/SSH_login_meterpreter.png)

## b) Valitun hyökkäyksen lähdekoodin avaus ja arviointi
SSH_login hyökkäyksen "sorsa" eli lähdekoodi aukeaa helpoiten msfconsolessa komennolla ```edit```, jolloin lähdekoodi tulee ruudulle näkyviin. Seuraavaksi omia pohditojani lähdekoodista parhaan osaamiseni mukaisesti.

Heti alussa selviää mitä toimintoja ja asetuksia moduuli käyttää Metasploit Frameworkista.

![SSH_login included](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/SSH_Login_included.png)

Rivillä 92 oleva ```def run_host(ip)``` vaikuttaa olevan hyökkäyksen "liha". Aluksi asetetaan kohde-IP, lähetetään konsoliin viesti 'Starting bruteforce', sekä alustetaan mahdollinen tietokanta? käytettävistä tunnuksista.

![Def run_host](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/SSH_def_run_host.png)

Myöhemmin metodissa otetaan käyttöön Metasploit Frameworkin kirjaston ```Metasploit::Framework::LoginScanner::SSH```. Oletan skannauksen alkavan rivillä 118 komennolla ```scanner.scan! do``` joka käynnistää kirjaston sisällä olevat toiminnot murtautumiseen. Lisäksi ```case result.status``` määrittelee kirjautumisyrityksien jälkeiset toimenpiteet riippuen tuloksesta.

![SSH login scanner scan](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/SSH_login_scanner_scan.png)

![SSH login scanner scan 2](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/SSH_login_scanner_scan2.png)

## c) Valitun hyökkäyksen toiminnan selvittäminen verkkosnifferillä

Tätä vaihetta varten skannasin verkkoliikennettä WireSharkilla hyökkäyksen aikana. Olin varmistanut että kumpikaan kone ei ole millään tavalla yhteydessä julkiseen verkkoon, joten WireShark-kaappauksessa ei ole muuta liikennettä kuin näiden kahden koneen välinen liikenne.

Heti ensimmäisenä on turvallista mainita, että brute force hyökkäys todennäköisesti olisi huomattavissa mikäli liikennettä seurataan. Kaappauksessa ilmenee jatkuvalla syötöllä tulevia "Diffie-Hellman Group Exchange Request"-viestejä samasta lähteestä.

![SSH login diffie hellman](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/SSH_login_diffie_hellmann.png)

Lisäksi käyttämällä ***Statistics -> Endpoints*** työkalua, näkyy että kaikki TCP-tason yhteydenotot tulivat samasta IP-osoitteesta, mutta jatkuvasti vaihtuvasta portista:

![TCP Endpoints](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Wireshark_TCP_Endpoints.png)

Muuta mielenkiintoista en kaappauksesta löytänyt, tai tajua etsiä. "Brute Force"-hyökkäys on varsin suoraviivainen eikä kovin kaunis. Samasta osoitteesta jatkuvien kirjautumisyrityksien tulvan luulisi herättävän jonkun huomion, tai jälkikäteen ainakin olevan selvää kuinka järjestelmään on murtauduttu.

## d) Ratkaistaan dirfuz-1

Ensiksi asensin Fuff:in Tero Karvisen ohjeen mukaisesti (Karvinen, T. 2023). Asennuksen jälkeen ajoin työkalun komennolla ```./ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ```.

![FFUF tulos](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/FFUF_tulos.png)

Huomaan heti, että kaikkien ruudulla näkyvien tuloksien koko on 154:

      [Status: 200, Size: 154, Words: 9, Lines: 10, Duration: 0ms]
      [Status: 200, Size: 154, Words: 9, Lines: 10, Duration: 4ms]
      [Status: 200, Size: 154, Words: 9, Lines: 10, Duration: 1ms]

joten mukaillen aiemmin suorittamaani dirfuz-0 harjoitusta, filtteröin vastauksen koon mukaan komennolla ```./ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ -fs 154```.

![FFUF filtteri](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/FFUF_filtteri.png)

Jäljelle jäi vain muutama vaihtoehto, jotka on helppo käydä läpi manuaalisesti. Haasteessa mainitaan että löydettäviä URL:eja on kaksi, Admin-sivu ja versionhallintaan liittyvä sivu. Näillä tiedoilla kokeilin sokkona kahta vaihtoehtoa ja molemmat osuivat nappiin:

![dirfuzt-1 löydetty](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Dirfuzt-1_l%C3%B6ydetty.png)

## e) Hack The Box

# Lähteet

- HackTricks s.a. Generic Methodologies & Resources. Basic Forensic Methodology. Pcap Inspection. Wireshark tricks. Luettavissa: https://book.hacktricks.xyz/generic-methodologies-and-resources/basic-forensic-methodology/pcap-inspection/wireshark-tricks. Luettu: 13.11.2024
- Karvinen, T. 2023. Find Hidden Web Directories. Luettavissa: https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/
