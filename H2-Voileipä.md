## H2 Voileipä | Blendi Thaqi 06/04/2026

## Ympäristö

**OS**: Kali GNU/Linux Rolling

**Browser**: Mozilla Firefox 140.8.0esr

**Hardware**: Virtualbox memory used 8 GB

**Processor**: AMD Ryzen 7 7800X3D | 4 cores used

**GPU**: NVIDIA GeForce RTX 5070 Ti 16GB

**Disk**: 80 GB

**Network**: NAT

## Artikkelit ja niiden tiivistelmät

#### Karvinen 2026: [Sudo without password](https://terokarvinen.com/passwordless-sudo/)
* Yksittäisille käyttäjille tai ryhmille on mahdollista antaa salasanattomat sudo-oikeudet turvallisesti lisäämällä ne `etc/sudoers.d/`-kansioon, vain käyttämällä `visudo`-komentoa.
* *Tärkeänä perussääntönä:* Konfiguraatioiden toimivuus kannattaa **aina** kokeilla sekä varmistaa ensin manuaalisesti ennen sen automatisointia.
* **Oma huomio:** Artikkelissa tuli hyvin esiin kuinka harjoitellessakaan ei saa käyttää huonoa väliaikaista salasanaa (kuten: "1234"), koska jos sen unohtaa vaihtaa myöhemmin, olet luonut koneellesi valtavan tietoturva-aukon .
#### Karvinen 2026: [Passwordless Sudo with Ansible](https://terokarvinen.com/passwordless-sudo-with-ansible/)
* Edellisen tehtävän manuaaliset vaiheet on mahdollista muuttaa automaattiseksi Ansible-rooliksi hyödyntäen Ansiblen perusmoduuleja: `group` eli ryhmän luonti, `user` eli käyttäjän luonti, `authorized_key` eli SSH-avaimen vienti sekä `copy` eli sudoers-säännön kirjoitus
* Kun ajamme tätä automaatiota ensimmäistä kertaa, on tärkeää muistaa että pitää käyttää `--ask-become-password`-valitsinta, jotta Ansible saa tarvittavat oikeuden luoda salasanaton ylläpitäjä.
#### Munroe 2006: [xkcd 149: Sandwitch](https://xkcd.com/149/)
* Hauska sarjakuva, joka tiivistää huumorilla kuinka `sudo` komento toimii Linux-ympäristössä.
* Ilman ylläpitäjän oikeuksia, sarjakuvassa henkilö (eli järjestelmä) kieltäytyy, mutta käyttämällä `sudo` etuliitettä, saamme järjestelmän pakotettua tottelemaan käskyä.
* **Oma kysymys:** Onko Windowsissa olemassa myös tälläistä vastaavaa komentoa?
#### Ansiblen sisäänrakennettu dokumentaatio (ansible-doc)
* Tehokas työkalu jota kannattaa käyttää, on `ansible-doc` jolla saat käyttöohjeet suoraan komentoriviltä, miten eri moduulit toimivat, eikä silloin tarvitse käydä etsimässä selaimesta.
* Esim. `copy` moduulilla voidaan määrittää lähde (`src`) ja kohde (`dest`) kun taas `apt` moduulilla riittää vain paketin nimi ja haluttu tila (esim: `state: present`).
* **Oma huomio:** Moduuleissa, kuten `mode` kannattaa syöttää oikeudet aina lainausmerkeissä (esimerkkinä: "640"), jotta voidaan olla täysin varmoja että Ansible tulkitsee ne varmasti oikein.
## Sudoless: Manuaalisen ylläpitäjän luonti

Aloitetaan tekemällä perusasetukset ja varmistetaan että kaikki toimii kuten pitääkin.

1. Luodaan ensimmäiseksi uusi käyttäjä sekä tarvittava ryhmä:

```bash
sudo adduser jerry    # Luodaan käyttäjä
sudo groupadd sudoless    # Luodaan ryhmä
sudo adduser jerry sudoless    # Lisätään käyttäjä luomaamme ryhmään
```

2. Luodaan käyttäjä, ja muistetaan lisätä sille nimi, niin osaamme erottaa sen muista käyttäjistä:

<img width="419" height="237" alt="image" src="https://github.com/user-attachments/assets/0a683cec-7e6c-4f7b-9fd9-b2a17ebe285f" />

3. Luodaan ryhmä, ja lisätään uusi luomamme käyttäjä luotuun ryhmään:

<img width="268" height="95" alt="image" src="https://github.com/user-attachments/assets/cc664be1-db0a-4870-b893-57781f75d8d5" />

4. Automatisoidaan SSH-kirjautuminen kopioimalla oma julkinen avain uuden käyttäjän tilille, jotta pääsemme ottamaan yhteyden helpoimmin.

```bash
ssh-copy-id jerry@localhost
```

<img width="904" height="217" alt="image" src="https://github.com/user-attachments/assets/d0696c2d-cea4-4f61-aadb-ffc61e7d7071" />

5. Jotta tuo `sudoless`-ryhmä voi suorittaa komentoja ilman minkäänlaista salasanaa, luodaan täysin uusi asetustiedosto ajamalla komento:

```bash
sudo visudo /etc/sudoers.d/sudoless
```

ja lisäämällä tiedostoon seuraava rivi:[^1]

```bash
%sudoless ALL=(ALL) NOPASSWD: ALL
```

6. Kun olemme lisänneet rivin tiedostoon, voimme poistua tallentamalla tiedosto `Ctrl + O` ja poistumalla tiedostosta `Ctrl + X`

<img width="604" height="536" alt="image" src="https://github.com/user-attachments/assets/e9ee22d7-7ad6-4e33-abbb-afd8e8e21874" />

## Salasanaton tunnus Ansible-roolilla

Voimme nyt alkaa automatisoida edellisen kohdan askeleet, jotta voimme tehdä kaiken paljon nopeammin. 

1. Aloitetaan luomalla roolin kansio rakenne:

```bash
mkdir -p ~/tansible/roles/berry/tasks
```

<img width="341" height="52" alt="image" src="https://github.com/user-attachments/assets/4cfe517e-194e-436d-be7e-a956235accfe" />

2. Sitten määritetään kohdekoneet `hosts.ini` tiedostoon:

```ini
localhost

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

<img width="390" height="101" alt="image" src="https://github.com/user-attachments/assets/671dde73-af6e-4029-b2f1-bd8d70517993" />

3. Sitten, luodaan Ansiblen "aivot", eli pääpelikirjan joka yhdistää kohdekoneet ja suoritettavat roolit toisiinsa:

```YAML
- hosts: all
  become: true
  roles:
    - berry
```

<img width="164" height="95" alt="image" src="https://github.com/user-attachments/assets/045af01b-218b-45d4-8aae-f2241e4a9b3c" />

4. Seuraavaksi, kirjoitetaan tehtävät `main.yml` tiedostoon siirtymällä sinne `micro ~/tansible/roles/berry/tasks/main.yml` komennolla:[^2]

```YAML
- group:
    name: "sudoless"
    state: present

- user:
    name: "berry"
    state: present
    groups: ["sudoless", "sudo", "adm"]

- authorized_key:
    user: "berry"
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFx5JvltpmVtYaLJhP4P2kEdYzSiTLyLpNdBgCMA9R10 blendi@kali"

- copy:
    dest: "/etc/sudoers.d/sudoless"
    content: "%sudoless ALL=(ALL) NOPASSWD: ALL\n"
    mode: "0640"
```

<img width="859" height="280" alt="image" src="https://github.com/user-attachments/assets/139e99b1-ff8d-4ee8-a7fd-08a7378c460b" />

5. Olemme valmiita, kokeillaan ajaa automaatio:
```bash
cd ~/tansible

ansible-playbook -i hosts.ini site.yml --ask-become-pass 
```

Saimme seuraavanlaisen tuloksen:

<img width="911" height="443" alt="image" src="https://github.com/user-attachments/assets/7b54c7a0-fe82-4c2e-86a7-7f47f5b91c0f" />

6. Kokeillaan vielä ottamalla yhteys luomaamme käyttäjään, ja kokeilemalla sudon toimivuus:
```bash
ssh berry@localhost

sudo echo "Hei Palvelinten hallinnan kurssilainen! (tai joku muu :D)"
```

Ja kuten näkyykin, saimme kaiken toimimaan täydellisesti! :)

<img width="748" height="213" alt="image" src="https://github.com/user-attachments/assets/542a2360-f142-4e85-adbd-460a5428ded5" />

## Ohjelmien asennus usealle koneelle

Tässä osiossa asennetaan kaksi hyödyllistä komentorivityökalua: `htop` eli tehtävien hallintaa varten oleva työkalu ja `bat` joka on paranneltu "värikkäämpi" versio `cat`-komennosta

1. Muistetaan ennen kaikkea päivittää pakettilistat, jotta voimme olla varmoja että mikään ei ole vanhentunut:

```bash
sudo apt-get update
```

2. Sitten voimme jatkaa luomalla rooli ja tehtävätiedosto:

```bash
mkdir -p ~/tansible/roles/tools/tasks
micro ~/tansible/roles/tools/tasks/main.yml
```

<img width="400" height="100" alt="image" src="https://github.com/user-attachments/assets/06991ee5-8833-4751-b52f-a21a2d36b86c" />

2. Lisätään `main.yml` tiedostoon siihen liittyvät koodit, eli mitä asennamme:

```YAML
- apt:
    name:
      - htop
      - bat
    state: present
```

<img width="206" height="116" alt="image" src="https://github.com/user-attachments/assets/dc40bf86-a446-4eab-8989-3a7c0d6c31ac" />

3. Seuraavaksi käydään lisäämässä puuttuvat roolit `site.yml`:stä:

```bash
micro ~/tansible/site.yml
```

lisätään sinne luomalle rooli, eli `tools`:

<img width="167" height="93" alt="image" src="https://github.com/user-attachments/assets/d863c5a9-bbf0-479a-b2ce-1f1ccaa61fa9" />

4. Voimme nyt ajaa `ansible-playbook -i hosts.ini site.yml -u berry` uudella luodulla käyttäjällämme:

<img width="911" height="481" alt="image" src="https://github.com/user-attachments/assets/431a1e8a-ddd2-480e-a7ae-e61b4950300b" />

Saimme "changed", joten voimme todeta että saimme sen toimimaan! Kokeillaan vielä varmuuden vuoksi komennnot jotka latasimme:

```bash

htop

batcat ~/tansible/site.yml

```

Tässä näemme että `htop` toimii: 

<img width="910" height="589" alt="image" src="https://github.com/user-attachments/assets/1480f078-a536-45e9-b50e-0230785116bc" />

Myös `bat` komento toimii:

<img width="380" height="212" alt="image" src="https://github.com/user-attachments/assets/4f9fd4e5-084d-4914-905b-fb49b492f134" />

## Tiedostojen kirjoittaminen ja niiden oikeudet

Tässä osiossa ideana on luoda `berry`-käyttäjälle kotihakemistoon tiedosto, määrittää sille sisältö ja sitten myös asettaa sille erilaiset käyttöoikeudet'

1. Aloitetaan luomalla kansio roolille ja lisäämällä rooli `site.yml` tiedostoon:

```bash
mkdir -p ~/tansible/roles/uusifile/tasks
```

<img width="369" height="49" alt="image" src="https://github.com/user-attachments/assets/c27a0e1f-d055-47a4-bf35-0fd8d303027d" />

Lisätään se `site.yml` tiedostoon:

<img width="178" height="109" alt="image" src="https://github.com/user-attachments/assets/c4af4bbb-c1d7-4bb0-be2d-f57828dab9ad" />

2. Avataan tiedosto (`micro ~/tansible/roles/uusifile/tasks/main.yml`), ja lisätään sinne koodia, jolla voimme kopioida/luoda tiedostoja kohdekoneelle, tiedoston omistajan ja sen oikeudet:

```YAML
- copy:
    content: |
      Tämän tiedoston pystyy lukemaan vain omistaja!
    dest: "/home/berry/salainen.txt"
    owner: "berry"
    group: "berry"
    mode: "0600"
```

<img width="448" height="128" alt="image" src="https://github.com/user-attachments/assets/f20cac1a-2f21-4237-964d-dba99510505f" />

3. Ajetaan pelikirja ja tarkistetaan tiedoston olemassaolo sekä sen oikeudet:

```bash
ansible-playbook -i hosts.ini site.yml -u berry

ls -l /home/berry/salainen.txt
```

<img width="916" height="526" alt="image" src="https://github.com/user-attachments/assets/10844288-4d11-498b-b78c-77fadf95c153" />

Tästä huomaammekin, että oikeukset toimivat, ja jopa minä en pysty katsomaan mitä tiedostossa on, ilman sudo komentoa:

<img width="533" height="65" alt="image" src="https://github.com/user-attachments/assets/f46576e2-8389-4109-9303-73aa81f0c3f6" />

<img width="591" height="66" alt="image" src="https://github.com/user-attachments/assets/acb449df-2751-4e43-9955-7d5494672f4f" />

Käyttämällä `sudo`:a ennen komentoja, voimme saada sen toimimaan:

<img width="538" height="194" alt="image" src="https://github.com/user-attachments/assets/4f7e92d8-894c-4286-8ad0-510d9474d9a0" />

4. Asetimme tiedostolle oikeudet muodossa `0600`, symbolisessa muodossa se kirjoitetaan näin: `-rw-------` joka tarkoittaa:
	
	- **1. merkki:** kertoo onko hakemisto vai ei (onko d kirjainta? ei.)
	- **Seuraavat kolme merkkiä:** (`rw-`) ovat omistajan oikeudet.
	- **Keskimmäiset kolme merkkiä:** (`---`) ovat ryhmän oikeudet.
	- **Viimeiset kolme merkkiä:** (`---`) ovat muiden käyttäjien oikeudet.
	
	Omistaja pystyy tässä tilanteessa vain lukea ja kirjoittaa, ryhmä ja muut käyttäjät eivät pysty tekemään yhtään mitään, koska heille ei ole annettu oikeuksia.

## Järjestelmän kellonajat kuntoon

Varmistetaan, että koneen aikavyöhyke on asetettu Helsinkiin. Tämä on todella tärkeää jotta saamme lokitiedostojen aikaleimat täsmäämään.

1. Luodaan kansio roolille ja lisätään se suoraan `site.yml` tiedostoon:

```bash
mkdir -p ~/tansible/roles/clock/tasks
```

<img width="179" height="140" alt="image" src="https://github.com/user-attachments/assets/c8815b53-05d9-4570-ba2c-606ae4659439" />

2. Seuraavaksi luodaan ja muokataan tehtävätiedostoa `main.yml` komennolla `micro ~/tansible/roles/clock/tasks/main.yml` ja lisätään sinne tämä pätkä koodia jolla pystymme määrittämään aikavyöhykkeen:

```YAML
- timezone:
    name: Europe/Helsinki
```

<img width="225" height="63" alt="image" src="https://github.com/user-attachments/assets/66bb3d85-f8db-44ba-bff5-3e67f5907e65" />

3. Viimeiseksi, ajetaan pelikirja ja katsotaan saimmeko aikavyöhykkeen vaihtumaan:
```bash
ansible-playbook -i hosts.ini site.yml -u berry
```

<img width="909" height="190" alt="image" src="https://github.com/user-attachments/assets/f57bc778-4a29-4647-9253-428ad4f98a2d" />

Saatiin muutos tiedostoon, kokeillaan vielä aikavyöhykettä komennolla `timedatectl`:

<img width="467" height="162" alt="image" src="https://github.com/user-attachments/assets/78b4ede9-3ce6-4775-adcf-750bf27abe6b" />

Saimme kaiken toimimaan! Time zone rivillä tulostui Helsinki, joka olikin meidän tavoitteena.

## Yhteenveto
* Saimme luotua hallintakäyttäjä `berry`, jolle lisättiin SSH-avain ja konfiguroitiin salasanaton sudo-oikeus
* Asensimme `apt`-moduulilla pari ohjelmaa: `htop` ja `bat` (jota käytetään komennolla `batcat`)
* Saimme luotua uudelle hallintakäyttäjälle, meidän käyttäjältä tiedosto `/home/berry/salainen.txt` ja asetettiin sille `0600` oikeudet.
* Asetettiin järjetelmän aikavyöhykkeeksi **Europe/Helsinki**
## Lähteet
[Passwordless Sudo](https://terokarvinen.com/passwordless-sudo/), [Passwordless Sudo with Ansible](https://terokarvinen.com/passwordless-sudo-with-ansible/), [xkcd 149: Sandwitch](https://xkcd.com/149/)

[^1]: Säännön purku / kuinka sääntö toimii: `%sudoless` (eli, kohderyhmä, % tarkoittaa ryhmää), `ALL=` (eli, kaikilla koneilla), `(ALL)` (Eli, kenen tahansa oikeuksilla), `NOPASSWD:` (Ohittaa salasanan kokonaan, eikä kysy sitä), `ALL` (Sallii kaikkien komentojen suorittamisen)
[^2]: `key: ""` kohtaan voit löytää oman julkisen avaimen käyttämällä seuraavaa komentoa, josta voit sitten kopioida oikean avaimen (**eli se joka alkaa `ssh-ed25519` tai `ssh-rsa`**): `cat ~/.ssh/id_ed25519.pub` tai `cat ~/.ssh/id_rsa.pub`
