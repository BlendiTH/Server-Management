# H1 Hei Ansiblen maailma | Blendi Thaqi 30/03/2026

## Ympäristö

**OS**: Kali GNU/Linux Rolling
**Browser**: Mozilla Firefox 140.8.0esr
**Hardware** Model: Virtualbox memory used 11 GB
**Processor**: AMD Ryzen 7 7800X3D | 4 cores used
**GPU**: NVIDIA GeForce RTX 5070 Ti 16GB
**Disk**: 80 GB
**Network**: NAT

## Artikkelit ja niiden tiivistelmät

#### Karvinen 2026: [SSH public key - Login without password](https://terokarvinen.com/ssh-public-key-login-without-password/)
* SSH on turvallisin ja selvästi yleisin tapa kirjautua palvelimille sisään. Omien etäyhteyksien lisäksi sitä käyttävät taustalla monet muut työkalut, kuten Git, Ansible ja Rsync.
* Julkisen avaimen todennus tekee SSH:n käytöstä vielä sujuvampaa sekä turvallisempaa, koska salasanaa ei tarvitse syöttää **manuaalisesti** joka kerta.
* Avainpari luodaan `ssh-keygen` -komennolla, josta vain sen julkinen avain (**eli**: `.pub`) kopioiaan kohdekoneen "hyväksyttyjen" avainten listalle käyttämällä `ssh-copy-id` komentoa.
* **Kysymys:** Mitä pitää tehdä, jos käyttäjä hukkaa oman yksityisen avaimensa, ja ei pääse enään käyttämään sitä?
#### Karvinen 2026: [Hello Ansible](https://terokarvinen.com/hello-ansible/)
* **Mikä on Ansible?** Se on konfiguraationhallinta työkalu, jolla pystyt kirjoittamaan infrastruktuuisi koodina, kuten: ohjelmien asennusten automatisointi sekä asetusten tekeminen useille koneille samanaikaisesti. 
* Ansible on "**agentless**", eli se toimii suoraan tavallisen SSH-yhteyden yli ja kohdekoneille ei tarvitse asentaa minkäälaisia erillisiä Ansiblen taustapalveluita.
* **Kysymys:** Miten pystyn varmistamaan, että tehtävä suoritetaan vain tarvittaessa?
## SSH-demonin asennus ja sen testaus

Aloitetaan ensin päivittämällä pakettilistat, jotta voimme olla varmoja että mikään ei puutu ja asentamalla SSH-palvelimen (**OpenSSH**):

```bash
sudo apt-get update
sudo apt-get -y install ssh
```

Sitten, asetetaan SSH-palvelin käynnistymään automaattisesti joka kerta kun järjestelmä käynnistyy ja käynnistetään myös demoni samaan aikaan:

```bash
sudo systemctl enable --now ssh
```

Testataan nyt palvelimen toimintaa, ottamalla SSH-yhteyden omaan koneeseeni (**localhost**). Koska olen jo aiemmin kerran testannut tätä yhteyttä, järjestelmä automaattisesti tunnistautui eikä pyytänyt minkäänlaista sormenjäljen vahvistamista. 
**Muuten olisi tullut tälläinen ilmoitus:** 
```
The authenticity of host 'localhost (::1)' can't be established.
ED25519 key fingerprint is: (Key näkyisi tässä!)
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?)
```

<img width="755" height="314" alt="image" src="https://github.com/user-attachments/assets/1db279c4-ae3f-49f6-8e57-06bb20691902" />

Lopuksi, katkaistaan testiyhteys ja palataan alkuperäiseen terminaali-istuntoon:

<img width="758" height="102" alt="image" src="https://github.com/user-attachments/assets/1b4327c0-89e6-47eb-addb-8f6327ef4d27" />

## SSH-kirjautumisen automatisointi julkisella avaimella

Luodaan ensimmäiseksi uusi avainpari, käyttäen oletusasetuksia (**RSA/Ed25519**) ilman salasanaa:

```bash
ssh-keygen
```

Seuraavaksi, kopioidaan luotu julkinen avain localhostin luotettuihin avaimiin:

```bash
ssh-copy-id localhost
```

SSH-kirjautuminen onnistuu nyt kokonaan ilman salasanaa, mikä on tärkeää, jotta voimme käyttää Ansiblea sujuvasti. Sitä pystymme kokeilemaan komennolla:

```bash
ssh localhost
```
## "Hei Ansible!" SSH:n yli

Asennetaan Ansible, ja tehdään ensimmäinen konfiguraatiomme. Ajamalla seuraavat komennot  voimme päästä alkuun:

1. Asennetaan **Ansible**, **Micro** (tekstieditori) ja **Tree** (näyttää tiedostot ja kansiopuun)

```bash
sudo apt-get update && sudo apt-get install -y ansible micro tree
```

2. Luodaan kansiorakenne ja siirrytään luotuun kansioon

```bash
mkdir -p ~/ansible/roles/hello/tasks
cd ~/ansible
```

3. Seuraavaksi, luodaan tiedosto `hosts.ini` joka kertoo Ansiblelle, mitä koneita hallitaan.

```TOML
localhost

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

4. **Tehdään myös varmuuden vuoksi asetustiedosto (`micro ansible.cfg`), jotta olisi helpompi jatkossa!**

```TOML
[defaults]
inventory = hosts.ini
```

5. Sitten tehdään "pääohje" `site.yml` joka yhdistää kohdekoneet ja roolit.

```yaml
- hosts: all
  roles:
    - hello
```

6. Luodaan varsinainen tehtävä `roles/hello/tasks/main.yml`

```yaml
- copy:
    dest: /tmp/hello-ansible
    content: "Hei Ansible!\n"
```

7. Testataan ja varmistetaan komennoilla: 
```bash
ansible-playbook site.yml
cat /tmp/hello-ansible
```

Kun ajetaan komento `ansible-playbook site.yml`, ensimmäisellä kerralla voi tulla näytölle "**changed=1**" joka on hyvä asia, koska tiedämme että komennot toimivat:

<img width="727" height="240" alt="image" src="https://github.com/user-attachments/assets/af09782d-7081-43a4-b29f-43711139bad1" />

Voimme tämän jälkeen tarkistaa komennolla `cat /tmp/hello-ansible`:

<img width="248" height="59" alt="image" src="https://github.com/user-attachments/assets/5512ccd0-ec6a-444a-a696-90f7a590bc3a" />

## Yhteenveto
* **SSH & Avaimet:** Asensimme SSH-palvelimen ja teimme sen kirjautumista automaattista luomalla SSH-avaimet (kirjautuminen ilman salasanaa).
* **Ansible:** Asensimme Ansiblen, ja loimme vaaditut kansiorakenteet sekä asetustiedostot.
* **Automaatio:** Saimme onnistuneesti `site.yml` ajettua, joka loi tiedoston localhostille!
## Lähteet
[Hello Ansible](https://terokarvinen.com/hello-ansible/), [SSH public key - Login without password](https://terokarvinen.com/ssh-public-key-login-without-password/)
