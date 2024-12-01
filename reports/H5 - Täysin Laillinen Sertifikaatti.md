# H5 - Täysin Laillinen Sertifikaatti
##### Pentesting course taught by Tero Karvinen @ Haaga-Helia

## x) Lue ja Tiivistä

### OWASP 2021: OWASP Top 10:2021

#### A01:2021 - Broken Access Control
- Access Controlin haavoittuvuudet tiivistettynä antavat hyökkääjälle mahdollisuuden päästä käsiksi käyttäjiin tai resursseihin joihin heillä ei pitäisi olla pääsyä
- Joitain tapoja ehkäistä:
  - Deny by default, eli mikäli jotain ei ole erikseen sallittu - se on kielletty
  - Lokiin kirjautuvien virheiden seuraaminen
  - Aseta rajapintoihin "rate limit", eli kuinka usein voi tehdä pyyntöjä. Tämä vähentää automaattisista työkaluista aiheutuvaa riskiä
 
 #### A10:2021 - Server-Side Request Forgery (SSRF)
 - SSRF ymmärtääkseni tarkoittaa selkokielellä: salatun datan pyytämistä palvelimelta väärentämällä URL-osoite. Mikäli webbisovellus ei ole erikseen estänyt tätä, URL-pyyntö pääsee palomuurin ja ACL:n läpi.
 - SSRF ehkäistään OSI-mallin 3. ja 7. tasolla (Network ja Application)

### PortSwigger Academy
- **IDOR** eli Insecure Direct Object References viittaa haavoittuvuuteen, jossa käyttäjä pääsee käsiksi objekteihin - kuten data tai palvelu - esimerkiksi väärentämällä URL-osoitteen tai syöttämällä koodia hakukenttiin
  - Tämä mahdollistaa muiden käyttäjien palvelunsisäisten tietojen vuotamisen, sekä niiden muokkaamisen
 
- **File Path Traversal** on menetelmä jossa hyökkääjä pääsee käsiksi palvelimen tiedostoihin syöttämällä haluamansa tiedostopolun palvelussa jo olemassa olevaan tiedostohakuun

- **Server-side template injection**-menetelmässä käytetään hyväksi "template engine"-palveluiden - kuten WordPress - haavoittuvuutta joka mahdollistaa käyttäjän syötteen ketjuttamisen suoraan templateen sen sijaan että sitä käsiteltäisiin datana.

- **Server-side request forgery** eli **SSRF** käyttää palvelinta käyttävää sovellusta tekemään HTTP-pyynnön sisäisiin resursseihin jotka normaalisti olisivat suojattuja.
  - Yleisiä keinoja SSRF:ää vastaan ovat input filtterit jotka käyttävät joko black- tai whitelistausta, mutta nämä on mahdollista ohittaa

- **Cross-site Scripting, eli XSS** kohdistuu palvelun käyttäjiin, eikä palvelimeen itsessään. Siinä hyökkääjä manipuloi palvelua niin, että se suorittaa haitallista koodia uhreiksi osuvien käyttäjien päädyssä
- XSS kattaa kolme hyökkäystyyppiä:
  - **Reflected XSS**:
    - Haitallinen koodi sisältyy URL-osoitteeseen, ja sivu palauttaa sen suoraan ilman tarkistuksia. Uhrin avatessa URL:n esimerkiksi sähköpostissa olevasta linkistä, koodi suoritetaan.
  - **Stored XSS**:
    - Haitallinen koodi tallennetaan palvelimelle, esimerkiksi kommenttina tai lomaketietona. Kun toinen käyttäjä lataa sivun, tallennettu koodi suoritetaan hänen selaimessaan.
  - **DOM-based XSS**:
    - Samoin kuin Reflected XSS-hyökkäyksessä, koodi suoritetaan suoraan uhrin selaimessa tämän avatessa URL:n, kun verkkosivu käsittelee syötettä JavaScriptin avulla.

## a) Totally Legit Sertificate

### ZAPProxy asennus ja käyttöönotto
Latasin ZAPin Kalin paketin hallinnasta. Käynnistyksen ohessa kysyttiin haluanko että sessio jatkuu taustalla, ja valitsin kyllä. Tällä tavoin minun ei tarvitse tallentaa sessiota jatkuvasti.

![zap-download](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/zaproxy-lataus.png)

![zap-käynnistys ja persist](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/zap-k%C3%A4ynnistys-ja-persist.png)

Menin päivittämään display-asetuksia, mutta painaessani "OK", sain virheen (josta ei nyt satu olemaan kuvankaappausta). Virhe kuitenkin johtui vanhentuneista add-oneista, ja ladattuani niihin päivitykset sain muutettua myös display-asetuksia.

"Process images in HTTP requests/responses"-asetus näyttää kuviin kohdistuvat pyynnöt, ja ajattelin että "Display timestamps on output tabs?"-asetus osoittautuisi hyödylliseksi, sillä kaikki dokumentaatio on hyvästä.

![zap display-asetukset](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/zap-display-asetukset.png)

### CA-sertifikaatin generointi ja käyttöönotto

Sertifikaatin generointi tapahtuu **Options => Network => Server Certificates**. Pystyt valitsemaan kuinka kauan haluat sertifikaatin olevan voimassa, oletuksena se on 365 päivää. Generoidaan se valitsemalla "Generate" ja tämän jälkeen tallennetaan se haluamaasi hakemistoon valitsemalla "Save".

![zap-certificate](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/zap-certificate.png)

Itse käytän Linuxilla Firefoxia, jossa sertifikaatti otetaan käyttöön menemällä **Settings => Privacy & Security => Certificates => View Certificates => Authorities => Import**. Etsin generoidun sertifikaatin ja painan "Open".

![Zap certificate käytössä](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/zap-certificate-k%C3%A4yt%C3%B6ss%C3%A4.png)

Seuraavaksi ZAP piti asettaa Firefoxin proxyksi. Varmistin ZAPProxyn asetuksista **Network => Local Servers/Proxies** mitä porttia localhost käyttää, ja käytin näitä tietoja Firefoxin proxy-asetuksien muuttamiseen.

![zap localhost proxy](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/zap-localhost-proxy.png)

![firefox proxy asetukset](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/firefox-proxy-settings.png)

Käynnistin varmuuden vuoksi selaimen uudestaan, ja ZAPProxyyn alkoi ilmestymään hakupyyntöjä. Varmuuden vuoksi käytin tähän Metasploitable2 virtuaalikoneen ja oman koneeni välistä yhteyttä.

![zap GET pyynnöt](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/zap-get.png)

## b) Kettumaista | Asenna FoxyProxy

FoxyProxy Standard on selaimen lisäosa, jonka avulla ZAP Proxy ymmärtääkseni seuraa vain FoxyProxyyn asetettuja verkkosivuja. Asennuksen löysin [FoxyProxyn sivuilta](https://getfoxyproxy.org/downloads/). Sivulta löytyy linkit ja ohjeet jokaiselle selaimelle. Itse käytän FireFoxia.

FoxyProxyn asetuksissa asetan proxyyn ohjattaviksi verkkosivuiksi localhostin, PortSwigger Academyn Labsit ja Metasploitable2-koneeni.

![Foxyproxy patterns](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/foxyproxy-patterns.png)

PortSwigger labrojen URL:in käytön huomasin Tommi Salon [raportista](https://github.com/TommiSalo02/penetration-testing-course/blob/main/h5-T%C3%A4ysin-Laillinen-Sertifikaatti.md), ja asetin sen FoxyProxyyn jo tässä vaiheessa. Relevantiksi se tulee vasta seuraavissa tehtävissä.

# PortSwigger Labrat

## c) Insecure Direct Object References

Tehtävänä oli etsiä käyttäjän 'Carlos' salasana. Labran alustana on jonkinlainen verkkokauppa, jonka etusivulla on vaihtoehtoina "Home", "My account" ja "Live chat". Avasin ensin chat-palvelun. Viestin kirjoittaminen ei tuottanut tulosta ZAP proxyn puolella, mutta "View Transcript" suoritti latauksen: 2.txt

Lataus näkyi ZAPin puolella pyyntönä:

    GET https://0a2f006c045da3e782292947002d00af.web-security-academy.net/download-transcript/2.txt HTTP/1.1

Palvelin antoi vastauksen:

    HTTP/1.1 302 Found
    Location: /download-transcript/2.txt

Elikkä meillä on tiedossa URL mistä chat-historia ladataan, ja että ne nimetään yksinkertaisesti numerojärjestyksessä. Saisinko ladattua tiedoston 1.txt kirjoittamalla hauksi?
    
    GET https://0a2f006c045da3e782292947002d00af.web-security-academy.net/download-transcript/1.txt HTTP/1.1

![PortSwigger IDOR](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/PortSwigger-IDOR.png)

Avaamalla tiedoston näen Carlosin chat-historian jossa hän kysyy salasanaansa chatissa.

![PortSwigger Carlos salasana](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/PortSwigger-Carlos-salasana.png)

Kirjautuminen näillä tiedoilla:

![PortSwigger IDOR Ratkaisu](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/PortSwigger-IDOR-ratkaistu.png)

# Lähteet
- FoxyProxy. s.a. Downloads. Luettavissa: https://getfoxyproxy.org/downloads/. Luettu: 01.12.2024
- Karvinen, T. 2024. Tunkeutumistestaus - Täysin Laillinen Sertifikaatti. Luettavissa: https://terokarvinen.com/tunkeutumistestaus/#h5-taysin-laillinen-sertifikaatti
- OWASP org. 2021. A01:2021 - Broken Access Control. Luettavissa: https://owasp.org/Top10/A01_2021-Broken_Access_Control/. Luettu: 29.11.2024
- OWASP org. 2021. A10:2021 - Server-Side Request Forgery (SSRF). Luettavissa: https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/. Luettu: 29.11.2024
- PortSwigger Academy. s.a. Cross-site scripting. Luettavissa: https://portswigger.net/web-security/cross-site-scripting. Luettu: 29.11.2024
- PortSwigger Academy. s.a. File Path Traversal. Luettavissa: https://portswigger.net/web-security/file-path-traversal. Luettu: 29.11.2024
- PortSwigger Academy. s.a. Insecure direct object references (IDOR). Luettavissa: https://portswigger.net/web-security/access-control/idor. Luettu: 29.11.2024
- PortSwigger Academy. s.a. Server-side request forgery (SSRF). Luettavissa: https://portswigger.net/web-security/ssrf. Luettu: 29.11.2024
- PortSwigger Academy. s.a. Server-side template injection. Luettavissa: https://portswigger.net/web-security/server-side-template-injection. Luettu: 29.11.2024
