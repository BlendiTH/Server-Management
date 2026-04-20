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
ansible-playbook -i hosts.ini site.yml
```

<img width="1001" height="264" alt="image" src="https://github.com/user-attachments/assets/797ac04d-3fc3-476d-a484-1ed347610d35" />

## 
