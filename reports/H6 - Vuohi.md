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
#### Insecure Direct Object References
#### Missing Funtion Level Access Control

### c) Identity & Auth Failure
#### Authentication Bypasses
#### Insecure Login

### d) Server-side Request Forgery

### e) Client side | Bypass front-end restrictions
## f) Editmenu



# Lähteet
- Karvinen, T. 2020. Try Web Hacking on New Webgoat 2023.4. Luettavissa: https://terokarvinen.com/2023/webgoat-2023-4-ethical-web-hacking/. Luettu: 04.12.2024
- Karvinen, T. 2024. Tunkeutumistestaus H6 - Vuohi | Tehtävänanto. Luettavissa: https://terokarvinen.com/tunkeutumistestaus/#h6-vuohi
