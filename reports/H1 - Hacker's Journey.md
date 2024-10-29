# H1 - Hacker's Journey | Kill Chain #1

## x) Lue/Katso/Kuuntele ja tiivistä

### Herrasmieshakkerit - Suomen kyberpuolustus, vieraana Tuomo Rusila | 0x2e
### Hutchins et al 2011: Intelligence-Driven Computer Network Defense Informed by Analysis of Adversary Campaigns and Intrusion Kill Chains, chapters Abstract, 3.2 Intrusion Kill Chain.
### Santos et al: The Art of Hacking (Video Collection): 4.3 Surveying Essential Tools for Active Reconnaissance.
### KKO 2003:36.

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
Latasin Metasloitable 2 asennukseen tarvittavat tiedostot [täältä](https://sourceforge.net/projects/metasploitable/). Asennustiedoston lataamisessa kesti itselläni n. 40 minuuttia, ja verkkoyhteyteni on suhteellisen nopea (400Mpbs), joten tähän kannattaa varata

## f) Virtuaaliverkko Kali-koneen ja Metasploit-koneen välille
## g) Metasploit-koneen etsiminen porttiskannauksella
## h) Metasploit-koneen tarkka porttiskannaus

# Lähteet

- Herrasmieshakkerit 2023. Suomen kyberpuolustus, vieraana Tuomo Rusila | 0x2e. Kuunneltavissa: https://open.spotify.com/episode/2z1oltiq7JYOtIAXQr8yHa?si=2071790e4b2d4566
- Kali Org 2024. Kali inside VirtualBox. Luettavissa: https://www.kali.org/docs/virtualization/install-virtualbox-guest-vm/
- Kali Org s.a. Kali installer images. Luettavissa: https://www.kali.org/get-kali/#kali-installer-images
- Sourceforge 2019. Metasploitable. Luettavissa: https://sourceforge.net/projects/metasploitable/
