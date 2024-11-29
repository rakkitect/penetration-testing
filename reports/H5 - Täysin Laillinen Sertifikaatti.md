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


# Lähteet
- Karvinen, T. 2024. Tunkeutumistestaus - Täysin Laillinen Sertifikaatti. Luettavissa: https://terokarvinen.com/tunkeutumistestaus/#h5-taysin-laillinen-sertifikaatti
- OWASP org. 2021. A01:2021 - Broken Access Control. Luettavissa: https://owasp.org/Top10/A01_2021-Broken_Access_Control/. Luettu: 29.11.2024
- OWASP org. 2021. A10:2021 - Server-Side Request Forgery (SSRF). Luettavissa: https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/. Luettu: 29.11.2024
