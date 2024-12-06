# H7 - Hakkeroimaan oppii hakkeroimalla
####

## b) PortSwigger Academy harjoitteluita
Päädyin valitsemaan labran: Username enumeration via different responses

Tarkoituksena on siis tunnistaa olemassa oleva käyttäjätunnus, ja käyttäen brute forcea löytää käyttäjän salasana.

Näillä tiedoilla avasin tietenkin palvelussa "My account" sivun, ja kokeilin kirjautumista saadakseni tutkittavaa liikennettä:

![Login]()

Eli virheellisillä tunnuksilla kirjautuminen antaa vain virheen "Invalid username". Tästä päättelin, että mikäli tunnus **olisi** palvelun tietokannassa, tulisi herjaus vain salasanasta.

Tätä tehtävää varten piti googlettaa kuinka ZAProxylla suoritetaan brute force hyökkäyksiä tunnuksia ja salasanoja kohtaan. ZAProxylla sattuikin olemaan juuri tähän sopiva artikkeli: [PortSwigger Labs: Username Enumeration with ZAP Scripts](https://www.zaproxy.org/blog/2022-04-14-portswigger-lab-username-enumeration-with-zap-scripts/).

![Zest Script]()

![Scripts tab]()

Ohjeessa sanotaan, että vastauksen koko olisi 3011 tavua, mutta ZAProxyssä näkyy sen olevan 3140. Mikäli tätä pitäisi muuttaa, se tehtäisiin tuplaklikkaamalla "Assert - Length..." ja muokkaamalla "Length" parametria halutuksi.

Seuraavaksi luodaan POST-pyynnön ympärille loop, joka suorittaa ns. fuzzauksen. Annetaan muuttujalle nimi kuten "fuzz", ja valitaan tiedosto jossa on lista käyttäjiä. PortSwiggerin labraympäristöön on olemassa oma lista käyttäjiä: https://portswigger.net/web-security/authentication/auth-lab-passwords

Jäljellä on vielä fuzz-muuttujan lisääminen itse POST-pyyntöön. Elikkä tuplaklikataan POST, ja lisätään "username=" perään {{fuzz}} (mikäli oma muuttujasi on eri nimellä niin huomioi se tässä):

![User Fuzz]()

Nyt kun ajan scriptin, aukeaa ZAPin alareunan ikkunaan uusi välilehti "Zest Results". Tästä pystyy seuraamaan scriptin tekemiä pyyntöjä:

![Zest Results]()

Ja korostettu pyyntö onkin erityinen: muiden vastauksien koko on 3,140, mutta yksi on 3,142. Avataan vastaus ja nähdään, että käyttäjätunnus ei olekaan invalidi:

![Incorrect password]()

Nyt palapelin ensimmäinen pala on löydetty, käyttäjätunnus on ```af```.

Salasanan löytäminen menee samalla kaavalla:
- Vaihdetaan loopin tiedosto salasana tiedostoon, joka löytyy myös tehtävänannosta
- Muokataan POST-pyynnön body seuraavanlaiseksi: ```username=af&password={{fuzz}}```

Ajoin scriptin uudestaan, ja jälleen yksi vastaus poikkesi muista:

![Salasana löytyi?]()

Pyynnön sisältö oli seuraava: ```username=af&password=princess```, eli nyt kokeillaan manuaalisesti ovatko nämä toimivat tunnukset.

![Solved]()

Palvelun luonteesta riippuen, tämän jälkeen voisi vaihtaa käyttäjän sähköpostiosoitteen palvelussa, salasanan tai löytää mahdollisesti pankkitietoja. Realistisessa tilanteessa millään palvelulla ei kuitenkaan ole näin huonoa suojausta toistuvia kirjautumisyrityksiä kohtaan :)

# Lähteet:
- ZAP, 2022. PortSwigger Labs: Username Enumeration with ZAP Scripts. Luettavissa: https://www.zaproxy.org/blog/2022-04-14-portswigger-lab-username-enumeration-with-zap-scripts/. Luettu: 06.12.2024
- PortSwigger. https://portswigger.net/web-security/authentication/auth-lab-passwords
