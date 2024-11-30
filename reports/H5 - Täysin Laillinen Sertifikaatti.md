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

## a) Totally Legit Sertificate | ZAPProxy asennus ja käyttöönotto

Latasin ZAPin Kalin paketin hallinnasta. Käynnistyksen ohessa kysyttiin haluanko että sessio jatkuu taustalla, ja valitsin kyllä. Tällä tavoin minun ei tarvitse tallentaa sessiota jatkuvasti.

Menin päivittämään display-asetuksia, mutta painaessani "OK", sain virheen (josta ei nyt satu olemaan kuvankaappausta). Virhe kuitenkin johtui vanhentuneista add-oneista, ja ladattuani niihin päivitykset sain muutettua myös display-asetuksia.

"Process images in HTTP requests/responses"-asetus näyttää kuviin kohdistuvat pyynnöt, ja ajattelin että "Display timestamps on output tabs?"-asetus osoittautuisi hyödylliseksi, sillä kaikki dokumentaatio on hyvästä.

![zap-download]()

![zap-käynnistys ja persist]

![zap display-asetukset]

# Lähteet
- Karvinen, T. 2024. Tunkeutumistestaus - Täysin Laillinen Sertifikaatti. Luettavissa: https://terokarvinen.com/tunkeutumistestaus/#h5-taysin-laillinen-sertifikaatti
- OWASP org. 2021. A01:2021 - Broken Access Control. Luettavissa: https://owasp.org/Top10/A01_2021-Broken_Access_Control/. Luettu: 29.11.2024
- OWASP org. 2021. A10:2021 - Server-Side Request Forgery (SSRF). Luettavissa: https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/. Luettu: 29.11.2024
- PortSwigger Academy. s.a. Cross-site scripting. Luettavissa: https://portswigger.net/web-security/cross-site-scripting. Luettu: 29.11.2024
- PortSwigger Academy. s.a. File Path Traversal. Luettavissa: https://portswigger.net/web-security/file-path-traversal. Luettu: 29.11.2024
- PortSwigger Academy. s.a. Insecure direct object references (IDOR). Luettavissa: https://portswigger.net/web-security/access-control/idor. Luettu: 29.11.2024
- PortSwigger Academy. s.a. Server-side request forgery (SSRF). Luettavissa: https://portswigger.net/web-security/ssrf. Luettu: 29.11.2024
- PortSwigger Academy. s.a. Server-side template injection. Luettavissa: https://portswigger.net/web-security/server-side-template-injection. Luettu: 29.11.2024
