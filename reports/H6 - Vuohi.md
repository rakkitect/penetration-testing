# H6 - Vuohi
##### Pentesting course taught by Tero Karvinen @ Haaga-Helia

## x) Lue ja Tiivistä
### Tero Karvinen 2020: Using New Webgoat 2023.4 to Try Web Hacking
- WebGoat tarvitsee Javan Development kitin toimiakseen
- WebGoat pitää konfiguroida johonkin vapaaseen porttiin. Mikäli käynnistyksessä tulee virhe "Port xxxx was already in use", niin porttia käyttävä sovellus pitää joko sulkea, tai valita toinen vapaa portti.

## a) Asenna Webgoat 2023.4
Asennus suoritetaan kohdan 'x' artikkelin mukaisesti. OpenJDK koneellani oli jo asennettuna, mutta minun piti asentaa ja enabloida palomuuri.

OpenJDK ja UFW asennettuna:

![OpenJDK ja UFW versiot]()

Artikkelissa mainitaan WebGoatin uusimman version asentaminen joka on tätä raporttia kirjoittaessa 2023.8, mutta kurssin tehtäviä varten pitää olla versio 2023.4.

    wget https://github.com/WebGoat/WebGoat/releases/download/v2023.4/webgoat-2023.4.jar

WebGoatin käynnistys:

    java -Dfile.encoding=UTF-8 -Dwebgoat.port=8888 -Dwebwolf.port=9090 -jar webgoat-2023.4.jar

![WebGoat asennettuna]()

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

Nyt on aika siis käyttää bruteforcea. Yritin etsiä onko ZAProxyssa mahdollista suorittaa sitä, mutta en löytänyt mitään. Joten generoin listan mahdollisista numeroista, ja aloin syöttämään niitä pitkäjänteisesti.

Hetken ajan jälkeen tajusin että tämähän on hullua. Kävi ilmi että ZAPissa on sisäänrakennettu Fuzzeri jota en vain osannut löytää. Jostain syystä oikeaa arvoa ei löytynyt kuitenkaan fuzzaamalla.

Ohessa käyttämäni fuzzaus parametrit ZAP:in sisällä:

![ZAP FUZZ]()

#### Insecure Direct Object References
Tehtävässä on tarkoitus päästä käsiksi toisen käyttäjän profiiliin. Kirjauduttuani "omilla" tunnuksillani, ZAPissa näkyi palvelimen vastaus profiilin tietojen katsomiseen:

![IDOR Profiili]()

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

Seuraavassa osassa piti saada käyttäjän hash. Tässä hyödynnettiin edellisessä osassa paljastunutta hakemistioa "/access-contro/users". Painamalla "submit" painiketta, sain ZAPiin POST-pyynnön. Tekemällä pyyntöön pienet muokkaukset sain tulostettua käyttäjät:

![Access Control Users]()

Muutokset olivat:
- POST => GET
- URL ```http://127.0.0.1:8888/WebGoat/access-control/user-hash``` => ```http://127.0.0.1:8888/WebGoat/access-control/users```
- Content type = ```Content-Type: application/json; charset=UTF-8```

Lähettämällä tämän ZAPin Manual Request Editorilla, palvelimelta tuli vastauksena kaikkien käyttäjien tiedot, josta napataan Jerryn hash.

Viimeisessä tehtävässä minun olisi tarkoitus luoda uusi tunnus lähettämällä POST pyyntö ```/access-control/users``` hakemistoon. Tässä käyttämäni HTTP-pyyntö.

Vastauksena tuli jatkuvasti ```HTTP/1.1 400 Bad Request
Content-Length: 0
Connection: close```

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



### c) Identity & Auth Failure

#### Authentication Bypasses
#### Insecure Login

### d) Server-side Request Forgery

### e) Client side | Bypass front-end restrictions
## f) Editmenu



# Lähteet
- Karvinen, T. 2020. Try Web Hacking on New Webgoat 2023.4. Luettavissa: https://terokarvinen.com/2023/webgoat-2023-4-ethical-web-hacking/. Luettu: 04.12.2024
- Karvinen, T. 2024. Tunkeutumistestaus H6 - Vuohi | Tehtävänanto. Luettavissa: https://terokarvinen.com/tunkeutumistestaus/#h6-vuohi
- RESTful. HTTP Methods. Luettavissa: https://restfulapi.net/http-methods/. Luettu: 04.12.2024
