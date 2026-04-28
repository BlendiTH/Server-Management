## H5 Gitar Hero | Blendi Thaqi 27/04/2026

## Ympäristö

**OS**: Kali GNU/Linux Rolling

**Browser**: Mozilla Firefox 140.8.0esr

**Hardware**: Virtualbox memory used 8 GB

**Processor**: AMD Ryzen 7 7800X3D | 4 cores used

**GPU**: NVIDIA GeForce RTX 5070 Ti 16GB

**Disk**: 80 GB

**Network**: NAT

## Artikkelit ja niiden tiivistelmät

#### Chacon & Straub 2014: [Pro Git, Luku 1.3](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F)
* Git ei tallenna _vain_ tiedostojen muutokset, vaan se myös ottaa projektista "tilannekuvan" **(snapshot)** jokaisella tallennuksella (eli joka kerta kun käytät `git commit` komentoa).
* Gitin toiminnot tehdään päasiassa paikallisesti, joten työtä ja versiointia voi tehdä vaikka lentokoneessa. Nettiä tarvitset vasta kun muutokset pitää lähettää pilveen (esim. GitHubiin) muiden nähtäville, komennolla `git push`.
* Gitillä on kolme tilaa, jossa tiedostot ovat joko **1) Modified** (eli muokattuna, mutta ei tallenettuna), **2) Staged** (eli valmisteltuna seuraavaan tallennukseen) tai **3) Committed** (eli tallennettuna turvallisesti tietokantaan).

#### Komentoketjun selitys: `git add --all && git commit; git pull && git push`
1. `git add -all`: Komento aloittaa etsimällä kaikki _muutetut_ tiedostot ja siirtää ne sitten odotustilaan (äsken opittuun **Staged** tilaan) tallennusta varten.
2. `&& git commit`: Sen jälkeen, komento ottaa odotustilan tiedostoista tilannekuvan ja tallentaa sen paikalliseen tietokantaan (**Committed** tila)
3. `; git pull`: Tässä vaiheessa, komento käy hakemassa pilvestä (esim. GitHubista) muiden tekemät uudet muutokset ja yhdistää ne omiin tiedostoihisi. **Tämä on pakollinen turvatoimi joka täytyy tehdä, ennen omien tietojen vientiä!**
4. `&& git push`: Lopuksi, komento lähettää omat valmiit paikalliset versiot suoraan pilveen.

**Alla oleva kuva tiivistää perusidean visuaalisesti:**

> <img width="2424" height="728" alt="KomentoketjuVisuaalisesti" src="https://github.com/user-attachments/assets/ab5ef184-2b6c-4ed7-92ba-c200c34332e0" />
> <p align="center"><small><i>Itse tehty kaavio</i></small></p>

## Uuden varaston luominen GitHubiin

1. Aloitetaan kirjautumalla sisään [GitHubiin](https://github.com/login):

> <img width="402" height="339" alt="image" src="https://github.com/user-attachments/assets/cde8d7b9-76fb-4e9b-a010-1c2f6451e817" />
<br>

2. Kun olet kirjautunut sisään, painamalla yläkulmassa olevaa **"+"** kuvaketta, voimme luoda uuden varaston valitsemalla **"New repository"**:

> <img width="1273" height="515" alt="image" src="https://github.com/user-attachments/assets/59336b42-e7a5-4ce2-b24b-31760560d644" />
<br>

3. Sitten, jatketaan täyttämällä varaston tiedot. Tehtävänannon mukaan, sekä nimessä että kuvauksessa tulee olemaan sana **"sunshine"** joten, nimetään varasto samalla tavalla:
- **Repository name:** sunshine
- **Description:** sunshine with a chance of bugs
- **Visibility:** Public (Valitsemme Public, jotta kuka tahansa pääsee katsomaan varastoa ja sen tiedostoja)
- **Add a README file:** On (README kertoo heti mitä repo sisältää ja mihin se on tarkoitettu, saamme tämän myötä myös ensimmäisen valmiin tiedoston, jota voimme jatkossa sitten kloonata ja muokata.)
- **Choose a license:** Valitaan pudotusvalikosta **GNU General Public License v3.0** (Voit itse valita haluatko lisenssin vai et, itse ainakin valitsen aina tämän saman).

> <img width="774" height="617" alt="image" src="https://github.com/user-attachments/assets/30e6fd8d-8d5d-4105-b660-ecc23d3d0f9a" />
<br>

4. Olemme valmiita julkaisemaan varastomme, sen pystymme tehdä painamalla oikealla alakulmassa olevaa vihreää **Create repository** -nappia.

> <img width="146" height="47" alt="image" src="https://github.com/user-attachments/assets/3c8a87d2-71ba-4227-b734-1861536104b8" />
<br>

## SSH-avaimen luonti ja sen vieminen GitHubiin

Luodaan SSH-avain tässä välissä, jotta Linux-koneemme voi jutella GitHubin kanssa turvallisesti, meidän täytyy antaa niille avaimet. **HUOM. Jos olet jo aikaisemmin luonut SSH-avaimen (esim. aiemmissa harjoituksissa), sinun ei silloin tarvitse tehdä uutta, ja voit hypätä suoraan kohtaan 2.**

1. Luodaan avain, kirjoita alla oleva komento terminaaliin:

```bash
ssh-keygen
```
_Tässä kohtaa, muista painaa Enteriä jokaiseen kysymykseen mitä näytölle tulee (noin 3 kertaa tulee kysyttyä asioita), kunnes ohjelma piirtää sinulle pinenen ASCII-taideteoksen ruudulle._

2. Jatketaan lukemalla julkinen avain, tulostamalla se ruudulle seuraavalla komennolla:

```bash
cat ~/.ssh/id_ed25519.pub
```
_HUOM. Jos avaimesi on eri niminen, sen pystyy löytämään ja tulostamaan seuraavalla komennolla: `cat ~/.ssh/id*.pub`_

<img width="746" height="127" alt="image" src="https://github.com/user-attachments/assets/bf55ea99-9d16-4a7a-9fa0-c75c0b14c3f3" />
<br>

3. Seuraavaksi, kopioi avain maalaamalla hiirellä koko ruudulle tulostunut teksti (eli se joka yleensä alkaa sanalla ``ssh-rsa` tai ``ssh-ed25519` ja päättyy koneesi / käyttäjäsi nimeen.

4. Sitten voimme jatkaa viemään avain GitHubiin:

- Githubissa, klikkaa oikeasta yläkulmasta profiilikuvaasi ja valitse pudotusvalikosta **Settings**.
> <img width="269" height="431" alt="image" src="https://github.com/user-attachments/assets/caa7ea71-ee18-451e-a26b-aa9dda7eeb75" />
<br>

- Sitten, valitaan vasemmasta valikosta **SSH and GPG keys**.


> <img width="305" height="308" alt="image" src="https://github.com/user-attachments/assets/2e65ea6e-10c8-4ea6-b572-2fdf58138028" />
<br>

- Painetaan seuraavaksi vihreää **New SSH key** -nappia.

> <img width="109" height="39" alt="image" src="https://github.com/user-attachments/assets/77c87ad8-08c0-473f-82dc-2b76c67139d2" />
<br>

- Lopuksi, annetaan avaimelle jokin kiva nimi, ja liitetään äsken kopioitu teksi isoon **"Key"** -laatikkoon ja sen jälkeen voimme painaa **Add SSH key** nappia.

> <img width="763" height="499" alt="image" src="https://github.com/user-attachments/assets/ca03f625-0d42-49b3-9b32-40b554bc4d08" />
<br>

## Kloonaus ja muutosten tekeminen

Voimme nyt jatkaa yhdistämällä pilven omaan koneeseen.

1. Aloitamme ensin menemällä luodun varastomme **"sunshine"** pääsivulle GitHubissa, jotta pystymme kloonaamaan varaston koneellemme, tarvitsemme sen kloonausosoitteen. Paina vihreää **<> Code** -nappia, ja valitse välilehti **SSH** (tärkeää valitä tämä, sillä muut voivat joskus olla toimimatta) ja kopioi siinä näkyvä osoite (esim: `git@github.com:BlendiTH/sunshine.git`):

> <img width="409" height="342" alt="image" src="https://github.com/user-attachments/assets/776b23de-5cd5-4562-bd8c-2052f8f9972e" />
<br>

2. Kloonataan varasto omalle koneelle kirjoittamalla terminaaliin `git clone` ja liittämällä sen perään äsken kopioima osoite. Esimerkiksi:
_HUOM. Tässä kohtaa kannattaa tehdä uusi hakemisto, ennen kuin alkaa kopioimaan varastoa omalle koneelle, tässä esimerkissä käytän kansiota `code`_

```bash
git clone git@github.com:BlendiTH/sunshine.git
```
_Jos se kysyy että "Are you sure you want to continue connecting?", kirjoita `yes` ja paina enter jotta pääset jatkamaan._

<img width="570" height="150" alt="image" src="https://github.com/user-attachments/assets/1624d3c3-db11-475b-aba2-7dfa2cd93ec2" />
<br>

3. Jatketaan menemällä sisään kansioon:

```bash
cd ~/code/sunshine
```

4. Jotta voimme aloittaa tekemään yhtikäs mitään, meidän täytyy kertoa Gitille kuka olemme, sillä Git vaatii, että nimesi sekä sähköpostisi ovat tiedossa ennen ensimmäistäkään tallennusta, ajetaan seuraavat kaksi komentoa (muista pistää omat tietosi!):

```bash
git config --global user.name "Blendi Thaqi"
git config --global user.email "BlendiTH@proton.me"
```

<img width="454" height="97" alt="image" src="https://github.com/user-attachments/assets/ebdc1689-7511-4940-a07a-77e26f1dde20" />
<br>

5. Nyt voimme jatkaa tekemään jonkinlaisen muutoksen, tehdään vaikka uusi tiedosto seuraavalla komennolla:

```bash
micro sunshines.txt
```

<img width="140" height="43" alt="image" src="https://github.com/user-attachments/assets/52ce8da1-dbd4-459e-9186-27cd9b5470d5" />
<br>

6. Voimme jatkaa käyttämään aiemmin opittua komentoketjua työntämällä nämä muutokset takaisin GitHubiin:

```bash
git add --all && git commit -m "Lisättiin uusi sunshines.txt tiedosto" ; git pull && git push
```

<img width="785" height="228" alt="image" src="https://github.com/user-attachments/assets/6ff05105-25b7-4e5b-965e-397c62d28800" />
<br>

7. Käydään vielä tarkistamassa tulokset menemällä selaimella takaisin GitHub-varaston sivulle ja päivittämällä sivu:

> <img width="934" height="537" alt="image" src="https://github.com/user-attachments/assets/c0a096da-25ce-4451-aa34-7971f373c3a9" />

*Kuten huomaamme, saimme `sunshines.txt` näkyville ja GitHubiin!*

## Tyhmän muutoksen tuhoaminen, Doh!

Tässä vaiheessa kokeilllaan, kuinka Git pystyy pelastamaan meidän omilta virheiltämme, jos emme ol vielä ehtineet niitä tallentamaan (commit).

1. Tehdään **"tyhmä"** muutos lisäämällä aiemmin luotun tiedostoon jotakin turhaa:

```bash
micro sunshines.txt
```

<img width="444" height="43" alt="image" src="https://github.com/user-attachments/assets/ea988fdf-1e27-4409-ba8d-0ee0581155d6" />
<br>

2. Voimme tarkistaa tilanteen komennolla `git status`, josta näemme että Git huomasi tiedoston muuttuneen, mutta se on **punaisena** (eli ei vielä add/commit-vaiheessa). Voimme myös tarkistaa tiedoston sisällön komennolla `cat sunshines.txt`:

<img width="580" height="255" alt="image" src="https://github.com/user-attachments/assets/dccf512f-0c00-487b-92c6-f56cdca1c01e" />
<br>

3. Tuhotaan huonot muutokset! Palautetaan projektin tila takaisin edelliseen tallennettuun versioon (commit) seuraavalla komennolla:

<img width="488" height="57" alt="image" src="https://github.com/user-attachments/assets/d8ddb4cc-7fc4-4f52-a7f2-65c3b37a271e" />
<br>

4. Tarkistetaan vielä lopputulos, josta voimme huomata että huono koodirivi katosi kokonaan ja tiedosto palasi täsmälleen siihen tilaan, jossa se oli edellisen commitin kohdalla.

```bash
cat sunshines.txt
```

<img width="286" height="64" alt="image" src="https://github.com/user-attachments/assets/b0165fa1-b589-4dd0-aa4e-b1c13a1a30e9" />
<br>

## Lokien tarkistelu

Seuraavaksi käydään tarkistamassa miltä projektin historia näyttää "konepellin alla".

1. Ajetaan lokikomento jonka avulla näemme historian ja tarkan eron siitä, mitä tiedostoissa on muuttunut:

```bash
git log --patch
```

<img width="711" height="590" alt="image" src="https://github.com/user-attachments/assets/8a844c02-bba5-4258-821b-df1e0e08a039" />

*Pääset katsomaan lisää tietoa kelaamalla alaspäin (tai ylöspäin) nuolinäppäimillä ja pääset pois painamalla `q`.*

<br>

2. Lokin tulkinta hieman tarkemmin
- **Commit-rivi:** Tuo pitkä merkkijono (esim. `commit da0040f...`) on SHA-1 -tarkistussumma, joka on version yksilöllinen sormenjälki.
- **Author-rivi:** Tässä näkyy simppelisti kuka on tehnyt muutoksen, sekä hänen sähköpostinsa.
- **Date-rivi:** Milloin tallennus on tehty.
- **Viesti:** Commit viesti, eli yleensä jokin lyhyt kirjoitamasi selite kun on käytetty commit komentoa.
- **Diff (eli muutokset):** Alhaalla näkyy vihreällä plusmerkillä rivit, jotka on lisätty, ja punaisella miinusmerkillä rivit, jotka on poistettu (kuvassa näkyy vain vihreitä plussia koska me ei olla poistettu mitään).

## Ansible-kansion versionhallinta

Infrastruktuuri koodin (kuten tässä kurssissa käytetyn Ansiblen pelikirjojen) versionti on koko automaation tärkein tuki ja turva. Laitetaan aiemmin tekemämme Ansible-kansio Gitin piiriin.

1. Siirrytään ensin luotuun Ansible kansioomme:

```bash
cd ~/tansible
```

2. Käynnistetään Git-versionhallinta tässä kansiossa käyttämällä seuraavaa komentoa:
_Tämä sitten luo piilotetun `.git` -kansion, joka alkaa seuraamaan kaikkia `tansible`-kansion tiedostoja!_

```bash
git init
```

<img width="660" height="267" alt="image" src="https://github.com/user-attachments/assets/34c631dc-1ea3-40fe-ad33-a0e4900a3c0c" />
<br>

3. Tehdään jokin muutos vaikka lisäämällä pieni kommentti pääpelikirjaamme:

```bash
micro site.yml
```

<img width="400" height="201" alt="image" src="https://github.com/user-attachments/assets/b8143efc-e12c-4207-9052-4358c63f0d81" />
<br>

4. Ajetaan ansible testiksi, jolla pystymme varmistamaan että kaikki yhä toimii muutoksen jälkeen:

```bash
ansible-playbook -i hosts.ini site.yml -u berry
```

<img width="395" height="59" alt="image" src="https://github.com/user-attachments/assets/958d7a4b-03d0-4254-847d-eb4550e9c765" />
<br>

5. Tallennetaan uusi versio, koska kuten äsken huomasimme Ansible-ajo meni läpi ilman virheitä:

```bash
git add --all && git commit -m "Lisättiin kommentti site.yml ja testattiin Ansiblen toimivuus"
```

<img width="794" height="265" alt="image" src="https://github.com/user-attachments/assets/bd5908ba-b40e-4515-b7bb-aae9ae66b888" />

## Yhteenveto
* **Teoria:** Opiskeltiin Gitin perusteet (tilannekuvat, paikallisuus) ja ymmärsimme työnkulun `add -> commit -> pull -> push` rakenteen kivalla kuvalla.
* **GitHub ja SSH:** Luotiin uusi repository ja asennettiin GitHubiin SSH-avaimet turvallista ja salasanoista vapaata yhteyttä varten.
* **Käytäntö ja peruutus:** Kloonasimme varastin, työnsimme muutoksia pilveen ja opimme kuinka tuhoamme virheet lopullisesti ``git reset --hard` komennolla.
* **Ansible:** Toimme aiemmin käytetyn Ansible-infrastruktuurikansion Gitin piiriin turvataksemme automaatiokoodimme.

## Lähteet
* [Pro Git, 2nd Edition (Chacon & Straub)](https://git-scm.com/book/en/v2)
* [GeeksforGeeks: Git Cheat Sheet](https://www.geeksforgeeks.org/git/git-cheat-sheet/)
* [Atlassian: Git Glossary & Commands](https://www.atlassian.com/git/glossary#commands)
* [ExplainShell: Git-komentoketjun purku](https://explainshell.com/explain?cmd=git+add+--all+%26%26+git+commit%3B+git+pull+%26%26+git+push)
* [A Grip on Git](https://agripongit.vincenttunru.com/)
* [DEV Community: Understanding Git through images](https://dev.to/nopenoshishi/understanding-git-through-images-4an1)
