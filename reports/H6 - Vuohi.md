# H6 - Vuohi
##### Pentesting course taught by Tero Karvinen @ Haaga-Helia
### Kurssi ja tehtävät: https://terokarvinen.com/tunkeutumistestaus/#h6-vuohi

## x) Lue ja Tiivistä
### Tero Karvinen 2020: Using New Webgoat 2023.4 to Try Web Hacking
- WebGoat tarvitsee Javan Development kitin toimiakseen
- WebGoat pitää konfiguroida johonkin vapaaseen porttiin. Mikäli käynnistyksessä tulee virhe "Port xxxx was already in use", niin porttia käyttävä sovellus pitää joko sulkea, tai valita toinen vapaa portti.

## a) Asenna Webgoat 2023.4
Asennus suoritetaan kohdan 'x' artikkelin mukaisesti. OpenJDK koneellani oli jo asennettuna, mutta minun piti asentaa ja enabloida palomuuri.

OpenJDK ja UFW asennettuna:

![OpenJDK ja UFW versiot](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20102528.png)

Artikkelissa mainitaan WebGoatin uusimman version asentaminen joka on tätä raporttia kirjoittaessa 2023.8, mutta kurssin tehtäviä varten pitää olla versio 2023.4.

    wget https://github.com/WebGoat/WebGoat/releases/download/v2023.4/webgoat-2023.4.jar

WebGoatin käynnistys:

    java -Dfile.encoding=UTF-8 -Dwebgoat.port=8888 -Dwebwolf.port=9090 -jar webgoat-2023.4.jar

![WebGoat asennettuna](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20104210.png)

## WebGoat 2023.4
### b) Broken Access Control
#### Hijack a session
Tehtävässä on tarkoitus kaapata olemassa oleva sessio käyttäen evästeitä. 'hijack_cookie' on eväste jonka arvo pitäisi keksiä/löytää.

ZAProxysta löytyi WebGoat/HijackSession/POST:login()(password,username) elementti, johon käytin Manual Request Editor työkalua. Lähetin useita kirjautumispyyntöjä **ilman** hijack_cookieta, jolloin palvelu lähetti seuraavat vastaukset:

    Set-Cookie: hijack_cookie=6155350480532472052-1733306600732
    Set-Cookie: hijack_cookie=6155350480532472054-1733306601737
    Set-Cookie: hijack_cookie=6155350480532472055-1733306602176
    Set-Cookie: hijack_cookie=6155350480532472057-1733306602558
    Set-Cookie: hijack_cookie=6155350480532472058-1733306602945

Hijack_cookien ensimmäinen osio kasvoi muuten yhdellä, mutta yhdessä kohtaa se hyppäsi numeron yli. Tästä päättelin sen olevan jonkun muun käytössä. Eli ensimmäinen osa on: ```6155350480532472053```. 

Evästeen toinen osa näyttää olevan mahdollisesti jonkinlainen aikaleima joka on ```1733306600732``` ja ```1733306601737``` välillä.

Nyt on aika siis käyttää bruteforcea. Kävi ilmi että ZAPissa on sisäänrakennettu fuzzeri. Jostain syystä oikeaa arvoa ei löytynyt kuitenkaan fuzzaamalla.

Ohessa käyttämäni fuzzaus parametrit ZAP:in sisällä:

![ZAP FUZZ](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20124353.png)

#### Insecure Direct Object References
Tehtävässä on tarkoitus päästä käsiksi toisen käyttäjän profiiliin. Kirjauduttuani "omilla" tunnuksillani, ZAPissa näkyi palvelimen vastaus profiilin tietojen katsomiseen:

![IDOR Profiili](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20130524.png)

Eli attribuutit "role" ja "userId", jotka eivät käyttöliittymässä näy - mutta vaikuttavat melko tärkeiltä.

Omaan profiiliin päästiin vaihtoehtoisesti käyttäen URL:ia ```WebGoat/IDOR/profile/2342384```, ja tästä voi päätellä et myös muiden käyttäjien tunnuksia pääsee tätä kautta löytämään. Fuzzasin ZAPilla pienen rangen sisältä muita käyttäjätunnuksia, ja löytyi käyttäjätunnus "2342388".

Toisen tunnuksen muuttamiseen etsin apua RESTful API:n [dokumentaatiosta](https://restfulapi.net/http-methods/). Sieltä löytyi seuraava tieto: "Use PUT APIs primarily to update an existing resource.

Tässä vaiheessa törmäsin ongelmiin, sillä mikään mitä yritin ei toiminut.

Muutin methodin GET => PUT, content-type: application/json ja syötin tarvittavat tiedot: ```{"role": 0, "color":"red", "size":"large", "name":"Buffalo Bill", "userId":2342388}```

    PUT http://127.0.0.1:8888/WebGoat/IDOR/profile/2342388 HTTP/1.1
    host: 127.0.0.1:8888
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
    Accept: */*
    Accept-Language: en-US,en;q=0.5
    Content-Type: application/json
    X-Requested-With: XMLHttpRequest
    Referer: http://127.0.0.1:8888/WebGoat/start.mvc
    Cookie: JSESSIONID=_7wjZlerNC8X1clK3osCmyhv1N_0nXnhrNJFB4PO; hijack_cookie=6155350480532472028-1733302370026
    Sec-Fetch-Dest: empty
    Sec-Fetch-Mode: cors
    Sec-Fetch-Site: same-origin
    {"role": 0, "color":"red", "size":"large", "name":"Buffalo Bill", "userId":2342388}
    Connection: close
    content-length: 0

Mutta vastaus oli jatkuvasti:

    HTTP/1.1 400 Bad Request
    Content-Length: 0
    Connection: close

#### Missing Funtion Level Access Control
Tehtävän ensimmäisessä osassa piilotetun dropdown-menun löytää yksinkertaisesti inspect-toolilla.

Seuraavassa osassa piti saada käyttäjän hash. Tässä hyödynnettiin edellisessä osassa paljastunutta hakemistioa "/access-control/users". Painamalla "submit" painiketta, sain ZAPiin POST-pyynnön. Tekemällä pyyntöön pienet muokkaukset sain tulostettua käyttäjät:

![Access Control Users](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20154836.png)

Muutokset olivat:
- POST => GET
- URL ```http://127.0.0.1:8888/WebGoat/access-control/user-hash``` => ```http://127.0.0.1:8888/WebGoat/access-control/users```
- Content type = ```Content-Type: application/json; charset=UTF-8```

Lähettämällä tämän ZAPin Manual Request Editorilla, palvelimelta tuli vastauksena kaikkien käyttäjien tiedot, josta napataan Jerryn hash.

Viimeisessä tehtävässä minun olisi tarkoitus luoda uusi tunnus lähettämällä POST pyyntö ```/access-control/users``` hakemistoon. Tässä käyttämäni HTTP-pyyntö.

    POST http://127.0.0.1:8888/WebGoat/access-control/users HTTP/1.1
    host: 127.0.0.1:8888
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
    Accept: application/json, text/javascript, */*; q=0.01
    Accept-Language: en-US,en;q=0.5
    X-Requested-With: XMLHttpRequest
    Connection: keep-alive
    Referer: http://127.0.0.1:8888/WebGoat/start.mvc
    Cookie: JSESSIONID=FT-MonaP40KDLs8GiPQuKXsnra0URUTRBD4bGbTB
    Content-Type: application/json; charset=UTF-8
    content-length: 0
    { "username": "user1", "password": "password", "admin": true }

Vastauksena tuli jatkuvasti:

    HTTP/1.1 400 Bad Request
    Content-Length: 0
    Connection: close

Virhe ilmeisesti viittaa virheeseen json contentissa. Json tai RESTful API eivät ole tarpeeksi tuttuja jotta pystyisin tämän ratkaisemaan.

### c) Identity & Auth Failure

#### Authentication Bypasses
Kuten tehtävänannossa sanotaan, todennuksen ohitus onnistuu useimmiten kohteessa olevan konfiguraatio/logiikka-virheen ansiosta.

![Security questions](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20221105.png)

En tiedä kysymyksien vastauksia, mutta painetaan submit silti. ZAPin avulla näen, että tämä luo POST-pyynnön josta näkyy seuraava data:

![POST](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20221123.png)

Mitä tapahtuu jos poistetaan ```secQuestion0=&secQuestion1```?

    "lessonCompleted" : false,
    "feedback" : "Not quite, please try again.",
    "output" : null,
    "assignment" : "VerifyAccount",
    "attemptWasMade" : true

Se ei siis toiminut. Seuraavaksi kokeilin muuttaa parametreja: ```secQuestion2=&secQuestion3=&jsEnabled=1&verifyMethod=SEC_QUESTIONS&userId=12309746```

    {
      "lessonCompleted" : true,
      "feedback" : "Congrats, you have successfully verified the account without actually verifying it. You can now change your password!",
      "output" : null,
      "assignment" : "VerifyAccount",
      "attemptWasMade" : true
    }

Läpäisty.

#### Insecure Login
Tämä oli erittäin yksiselitteinen tehtävä.

![Let's try insecure login](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20222501.png)

Katsotaan ZAPissa POST-pyyntöä:

![POST](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20222512.png)

Elikkä tunnus "CaptainJack" ja salasana "BlackPearl".

### d) Server-side Request Forgery

Tehtävänä on URL:ia muuttamalla saada Jerryn kuva palvelimelta.

![Tom](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20222911.png)

![Tom POST](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20222923.png)

Vaihdetaan vain ```url=images%2Ftom.png``` => ```url=images%2Fjerry.png```

Toisessa tehtävässä piti saada palvelin hakemaan tietoa osoitteesta http://ifconfig.pro. Tuttu juttu, painetaan painiketta ja tutkitaan POST-pyyntöä.

![Tehtävänanto](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20223220.png)

![POST](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20223327.png)

Muutetaan URL vastaamaan haluttua osoitetta: url=http://ifconfig.pro

![Rocked the SSRF](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20223959.png)

### e) Client side | Bypass front-end restrictions
#### 1
Tarkoituksena on löytää käyttäjän Neville Bartholomew palkkatiedot, mutta meillä ei ole niihin pääsyä.

![Näkymä](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20224357.png)

Käyttämällä inspect-työkalua näin että yksi User ID puuttuu: 102. Päättelin tämän olevan Nevillen ID. Sitten pitää keksiä miten tämän avulla saadaan hänen tietonsa näkyviin.

![HTML](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20224309.png)

Ensiksi kokeilin HTML-koodin muuttamista inspect-työkalussa, joka toimi teoriassa. User ID: 102 ei olekkaan Nevillen, vaan Moe Stoogen. User ID: 111 taas oli John Wayne.

![Moe Stooge](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20225257.png)

Kokeilin eri numeroita kuten 99 ja 1, mutta nämä eivät tehneet mitään. Lopulta User ID 112 oli oikein:

![Neville](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20225600.png)


#### 2
Seuraavassa osiossa inspectillä löytyy osio joissa listataan koodeja, mutta mikään näistä ei anna "täyttä" alennusta. Mutta kun tarkastelee GET-pyyntöjä joita palvelusta tulee, ja etsii hakusanalla "discount", löytyy tämmöinen:

![get it for free](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20230938.png)

![Ilmainen puhelin](https://github.com/rakkitect/penetration-testing/blob/main/reports/Kuvat/Screenshot%202024-12-04%20230958.png)

## f) Editmenu

Yritin luoda omaa komentoa Paletteroon joka on Micro-tekstieditorin lisäosa, mutta en onnistunut siinä.

[Palettero Github](https://github.com/terokarvinen/palettero).

# Lähteet
- Karvinen, T. 2020. Try Web Hacking on New Webgoat 2023.4. Luettavissa: https://terokarvinen.com/2023/webgoat-2023-4-ethical-web-hacking/. Luettu: 04.12.2024
- Karvinen, T. 2024. Tunkeutumistestaus H6 - Vuohi | Tehtävänanto. Luettavissa: https://terokarvinen.com/tunkeutumistestaus/#h6-vuohi
- Karvinen, T. Palettero. https://github.com/terokarvinen/palettero
- RESTful API s.a. HTTP Methods. Luettavissa: https://restfulapi.net/http-methods/. Luettu: 04.12.2024
