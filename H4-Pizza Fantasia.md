## H4 Pizza Fantasia | Blendi Thaqi 21/04/2026

## Ympäristö

**OS**: Kali GNU/Linux Rolling

**Browser**: Mozilla Firefox 140.8.0esr

**Hardware**: Virtualbox memory used 8 GB

**Processor**: AMD Ryzen 7 7800X3D | 4 cores used

**GPU**: NVIDIA GeForce RTX 5070 Ti 16GB

**Disk**: 80 GB

**Network**: NAT

## Artikkelit ja niiden tiivistelmät

> *Tämän osion tiivistelmät perustuvat Tero Karvisen väitöskirjaan: "**Configuration Management of Distributed Systems over Unreliable and Hostile Networks**"* [^1]

#### Karvinen 2023: [4.12.1 Size and Complexity of Some DSLs (112. Ominaisuuksien määrä.)](https://westminsterresearch.westminster.ac.uk/item/w7vvz/configuration-management-of-distributed-systems-over-unreliable-and-hostile-networks)
* Konfiguraationhallintatyökalujen (kuten kurssilla ennen käytetyn Saltin) kielet ovat kasvaneet niin valtaviksi, että esim. Saltin ohjekirja on jopa kokonaista väitöskirjaa pidempi.
* Yksittäisen työkalun ulkoa opettelu täydellisesti on yhdelle ihmiselle käytännössä mahdotonta.
* **Oma huomio:** Tämän takia, virallisten dokumentaatioiden (kuten kurssilla käytettyä `ansible-doc`) sujuva käyttö ja sen lukeminen on tärkeämpää kuin se, että yrittää muistaa asiat ulkoa.

#### Karvinen 2023: [4.12.2 Use of DSL Functions in Case Configuration (112-115. Mitä oikeasti käytetään.)](https://westminsterresearch.westminster.ac.uk/item/w7vvz/configuration-management-of-distributed-systems-over-unreliable-and-hostile-networks)
* Vaikka näissä työkaluissa on tuhansia erilaisia ominaisuuksia, aidoissa tuotantoympäristöissä pärjätään yleensä vain muutamalla peruskomennolla kuten: `file`, `package`, `exec`, `service`. 
* Taustapalvelimet (eli. **daemonit**) asennetaan melkeinpä aina samalla ja tutula "package-file-service" kaavalla.
* **Oma kysymys:** Jos suurinosa kaikesta työstä hoituu näillä muutamalla peruskomennolla, mihin niitä *todella* harvinaisia (eli niitä, joita oikeasti käytetään todella vähän) muita moduuleja oikeasti edes tarvitaan?

#### Karvinen 2023: [4.12.3.1 Dependencies Between Main Functions (115-117. Tärkeimmät rakennuspalikat.)](https://westminsterresearch.westminster.ac.uk/item/w7vvz/configuration-management-of-distributed-systems-over-unreliable-and-hostile-networks)
* Kaikkien automaatioiden toimintojen välillä on tiukka suoritusjärjestys, ja ne perustuvat lopulta vain kahteen asiaan: Ohjelman ajamiseen: `exec` ja tiedoston muokkaamiseen: `file`.
* Järjestelmälle tärkeimpänä ominaisuutena on, että sen tulee aina olla idempotentti, eli muutoksia tehdään vain jos niille on tarvetta.
* **Oma huomio:** Idempotenssi on todella nerokasta, sillä voit sen avulla ajaa asennusskriptejä vaikka satoja kertoja peräkkäin, ilman pelkoa siitä, että mikään menee rikki.

## Demonin asennus käsin (Fail2Ban)

Valitsin tätä tehtävää varten asennettavaksi demoniksi **Fail2Banin**. Se on _kevyt_ mutta tehokas tietoturvaohjelma, jonka päätavoitteena on suojata palvelinta brute force eli väsytyshyökkäyksiltä. Aloitetaan asentamalla demoni manuaalisesti.

1. Muistetaan aina ensin päivittää pakettilistat, joka jälkeen voimme jatkaa demonin asentamisella:

```bash
sudo apt update
sudo apt install fail2ban
```

<img width="780" height="560" alt="image" src="https://github.com/user-attachments/assets/ca16b9cd-1bfc-42f4-9876-78cab58a35e4" />
<br>

2. Käynnistetään seuraavaksi palvelin (`start`), ja laitetaan se käynnistymään myös automaattisesti joka kerta, kun kone käynnistetään uudelleen (`enable`):

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

<img width="988" height="159" alt="image" src="https://github.com/user-attachments/assets/81d6c035-b324-45c2-b17b-92f4b006615e" />
<br>

3. Voimme seuraaavaksi aloittaa sen toimivuuden testauksen komennolla `fail2ban-client`. Komennon avulla kysymme siltä palvelimen nykytilaa (ja mitä se suojaa, oletuksena aina vähintään SSH-yhteyksiä[^2]):

```bash
sudo fail2ban-client status
```

<img width="724" height="268" alt="image" src="https://github.com/user-attachments/assets/8cdd0ea9-2ba3-4616-b0a8-3b637dace8d3" />
<br>
<img width="509" height="186" alt="image" src="https://github.com/user-attachments/assets/e3ff32b6-07cf-4c54-aabb-ac7ae16c964c" />

## Demonin asennuksen automatisointi Ansiblella

Voimme nyt jatkaa Fail2Banin asennuksen siirtämisen Ansiblen hoidettavaksi, käyttäen "package-file-service" mallia.

1. Luodaan roolin kansiorakenne luomalla Ansiblelle uusi `fail2ban` rooli ja sen tarvitsemat alikansiot yhdellä komennolla, viimeisestä tehtävästä opitulla aaltosuljelaajennuksen avulla: [^3]

```bash
mkdir -p ~/tansible/roles/fail2ban/{tasks,handlers,files}
```

<img width="503" height="46" alt="image" src="https://github.com/user-attachments/assets/0b32fbac-eeea-4bf1-81ad-ae6a782c17b7" />
<br>

2. Fail2Banin oletusasetukset ovat sen omassa tiedostossa: `jail.conf`. Mutta ohjelman virallinen suositus on kuitenkin se, että alkuperäistä tiedostoa ei saa muokata, vaan siitä pitää tehdä kopio nimellä: `jail.local`[^4]. Joten, teemme näin kopioimalla tiedoston suoraan Ansiblen `files`-kansioon:

```bash
cp /etc/fail2ban/jail.conf ~/tansible/roles/fail2ban/files/jail.local
```

<img width="587" height="48" alt="image" src="https://github.com/user-attachments/assets/187e3ad8-b724-45d1-86f8-1fea1c23f803" />
<br>

3. Luodaan seuraavaksi handler, joka osaa käynnistää palvelun uudelleen, jos asetuksiin tulee myöhemmin muutoksia:

```bash
micro ~/tansible/roles/fail2ban/handlers/main.yml
```

Kirjoitetaan tiedostoon koodi seuraavanlaisesti:

```YAML
- name: restart fail2ban
  service:
    name: fail2ban
    state: restarted
```

<img width="220" height="94" alt="image" src="https://github.com/user-attachments/assets/a2c46043-54e3-437f-b095-6647e32cb184" />
<br>

4. Nyt voimme jatkaa tehtävien määrittämiseen:

```bash
micro ~/tansible/roles/fail2ban/tasks/main.yml
```

Kirjoitetaan tiedostoon koodi, jonka avulla voimme asentaa paketin, kopioi `jail.local` tiedostomme kohdekoneelle ja varmistaa, että palvelu on päällä

```YAML
- apt:
    name: fail2ban
    state: present

- copy:
    src: "jail.local"
    dest: "/etc/fail2ban/jail.local"
    owner: "root"
    group: "root"
    mode: "0644"
  notify: restart fail2ban

- service:
    name: fail2ban
    state: started
    enabled: yes
```

<img width="330" height="267" alt="image" src="https://github.com/user-attachments/assets/8283bf05-23a7-406a-811f-e15501acd1e2" />
<br>

5. Muistetaan myös lisätä rooli pääpelikirjaamme, jotta Ansible ymmärtää ajaa uuden automaation, lisäämällä sen `site.yml` tiedostoomme:

```YAML
- hosts: all
  become: true
  roles:
    - fail2ban
```

<img width="299" height="168" alt="image" src="https://github.com/user-attachments/assets/89a97144-9cd8-4528-b16e-4a64d0820388" />
<br>

6. Lopuksi voimme ajaa pelikirja Ansiblella, siirtymällä ensin oikeaan hakemistoon eli `tansible`:

```bash
cd ~/tansible
ansible-playbook -i hosts.ini site.yml -u berry
```

<img width="1001" height="264" alt="image" src="https://github.com/user-attachments/assets/797ac04d-3fc3-476d-a484-1ed347610d35" />

## Master Copyn muokkaus ja käyttöönotto

Voimme aloittaa automaation toiminnan testaamisen, muuttamalla Fail2Banin asetuksia omalla koneellamme Ansiblen avulla ja pakottamalla ne voimaan kohdepalvelimelle.

1. Ensin, käydään muokkaamassa pääkopiota asetustiedostosta joka on Ansiblen `files` kansiossa. Oletuksena Fail2Ban antaa hyökkääjille 10 minuutin bännit (`bantime = 10m`). Muokataan bänni aikaa nostamalla sitä 1 tuntiin:

```bash
micro ~/tansible/roles/fail2ban/files/jail.local
```

Etsitään tiedostosta rivi jossa lukee `bantime = 10m` joka löytyy **[DEFAULT]** osion alta. Muutetaan se seuraavaksi muotoon `bantime = 60m`

<img width="817" height="75" alt="image" src="https://github.com/user-attachments/assets/c3f8a66b-f037-4d57-aa06-531b68a51b2c" />
<br>

2. Ajetaan pelikirja uudelleen, jotta Ansiblen `copy` moduuli huomaa tiedoston muuttuneen ja kutsuu handlerin käynnistämään palvelun uudelleen:

```bash
ansible-playbook -i hosts.ini site.yml -u berry
```

<img width="996" height="248" alt="image" src="https://github.com/user-attachments/assets/3a718a5c-b534-40fa-ab26-f524d0265c32" />
<br>

3. Tarkistetaan vielä Fail2Banin avulla, kysymällä siltä ottiko se uuden asetuksen käyttöön SSH-palvelun kohdalla. **HUOM:** Fail2Ban ilmoittaa ajat sekunneissa, joten 1h näkyy lukuna **3600**: [^5] 

```bash
sudo fail2ban-client get sshd bantime
```

<img width="340" height="78" alt="image" src="https://github.com/user-attachments/assets/801d4429-3630-42b8-ae49-5cc026fa12f3" />

## Asetusten tuhoaminen ja niiden korjaus

Nyt päästään Puuha pete moodiin. Kokeillaan kuinka kestävä automaatiomme on, tuhoamalla kohdepalvelimen asetukset tarkoituksella ja sitten niiden korjaamisella.

1. Simuloidaan vahinkoa poistamalla Fail2Banin yksi tärkeimmistä tiedostoista, eli sen asetustiedosto `jail.local` ja sammutetaan samalla koko tietoturvapalvelu.

```bash
sudo rm /etc/fail2ban/jail.local
sudo systemctl stop fail2ban
```

<img width="301" height="93" alt="image" src="https://github.com/user-attachments/assets/e48da9ca-842a-48ee-8be6-cec057bad2a5" />
<br>

2. Tarkistetaan vielä tiedoston tilaa, että onko se oikeasti kadonnut.

```bash
ls -l /etc/fail2ban/jail.local
```

<img width="584" height="69" alt="image" src="https://github.com/user-attachments/assets/e285ce80-8878-4618-9678-ff01a5327a43" />
<br>

3. Ajetaan pelikirjamme jälleen kerran, mutta koska asetustiedosto puuttuu sekä palvelin on sammutettu, voimme luottaa Ansibleen, sillä se tunnistaa poikkeamat ja palauttaa kaiken tekemämme Master Copyn pohjalta tismalleen takaisin.

```bash
ansible-playbook -i hosts.ini site.yml -u berry
```

<img width="1001" height="201" alt="image" src="https://github.com/user-attachments/assets/9d19e5da-c52f-4b95-bb39-ad25e91e3e54" />
<br>

4. Varmistetaan vielä että saiko Ansible ongelman korjattua, onko tietoturva oikeasti taas päällä ja ovatko meidän tunnin (3600) bännit taas voimassa.

```bash
sudo fail2ban-client status
sudo fail2ban-client get sshd bantime
```

<img width="346" height="159" alt="image" src="https://github.com/user-attachments/assets/82355053-789e-4145-8838-134c948ed867" />

## Idempotentti

Olemme lähellä loppua, meidän pitää kuitenkin tarkistaa onko tilamme idempotentti. Se siis tarkoitta, että asennusskripti tekee muutoksia **vain silloin, jos tavoitetila poikkeaa nykytilasta.**

1. Viimeisessä tehtävässä näimme, kuinka Ansible juuri korjasi koko palvelimen täydelliseen kuntoon, ajetaan kuitenkin komento taas uudelleen, jotta voimme olla 100% varmoja.

```bash
ansible-playbook -i hosts.ini site.yml -u berry
```

<img width="992" height="206" alt="image" src="https://github.com/user-attachments/assets/b40da4a7-3460-461e-b796-72440308c3eb" />
<br>

> Kuten kuvasta huomataan, tilamme on idempotentti. Ansible tarkisti, onko ohjelma asennettuna, onko asetustiedosto `jail.local` olemassa sekä että se on täysin identtinen ansiblen version kanssa, ja että palvelu on jo päällä. Koska korjattavaa ei ollut, Ansible ei myöskään tehnyt yhtään muutosta: `changed=0`, eikä myöskään tarvinnut käynnistää Fail2Bania turhaan uudelleen.

## Yhteenveto
- **Teoria:** Tutustuimme Karvisen väitöskirjaan, jossa selitettiin kuinka tuotannossa pärjää muutamalla peruskomennolla, ja se että "Package-File-Service" järjestys on automaation peruskivi.

- **Asennus:** Asensimme Fail2banin manuaalisesti

- **Automaatio:** Automatisoimme asennuksen rakentamalla Ansible-rooli, joka ajaa omat tietoturva-asetukset (kuten meidän 1t bännit) `.local` tiedoston kautta ilman että koskimme alkuperäistä `jail.conf` tiedostoa.

- **Tuho ja palautus:** Tuhosimme asetukset täysin `purge` komennolla, mutta saimme sen  automaattisesti Ansiblen avulla takaisin nollasta, jonka Ansible teki itsenäisesti.

- **Idempotenssi:** Todistimme, ettäAnsible tarkistaa nykytilan eikä tee turhia muutoksia, vaan tekee niitä vain jos niille on tarvetta.

## Lähteet
- [Configuration Management of Distributed Systems over Unreliable and Hostile Networks](https://westminsterresearch.westminster.ac.uk/item/w7vvz/configuration-management-of-distributed-systems-over-unreliable-and-hostile-networks)

- [Bemyak: Useful Linux Daemons For Better Pc Experience](https://dev.to/bemyak/useful-linux-daemons-for-better-pc-experience-560)

- [Github Repo: Fail2Ban](https://github.com/fail2ban/fail2ban)

- [Neuralengineer Blog: Fail2Ban: Block brute-force attacks early](https://blog1.neuralengineer.org/fail2ban-block-brute-force-attacks-early-0949ba4653f9)

- [Digitalocean tutorials: "How Fail2Ban Works to Protect Services on a Linux Server"](https://www.digitalocean.com/community/tutorials/how-fail2ban-works-to-protect-services-on-a-linux-server)


[^1]: https://westminsterresearch.westminster.ac.uk/item/w7vvz/configuration-management-of-distributed-systems-over-unreliable-and-hostile-networks
[^2]: https://github.com/fail2ban/fail2ban/wiki/How-fail2ban-works
[^3]: **Miksi käytin {}:** Tämä tulee Bashin komentotulkinnasta "brace expansion". Sen avulla voimme säästää aikaa jos olemme luomassa useita kansioita kerrallaan, esimerkkinä, komento: `mkdir {a, b, c}` luo kansiot a, b ja c vain tuolla yhdellä komennolla.
[^4]: https://wiki.archlinux.org/title/Fail2ban#Configuration
[^5]: https://security.stackexchange.com/questions/271131/fail2ban-syntax-for-setting-bantime-in-a-unit-other-than-seconds
