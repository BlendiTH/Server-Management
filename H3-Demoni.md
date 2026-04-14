## H3 Demoni | Blendi Thaqi 14/04/2026

## Ympäristö

**OS**: Kali GNU/Linux Rolling

**Browser**: Mozilla Firefox 140.8.0esr

**Hardware**: Virtualbox memory used 8 GB

**Processor**: AMD Ryzen 7 7800X3D | 4 cores used

**GPU**: NVIDIA GeForce RTX 5070 Ti 16GB

**Disk**: 80 GB

**Network**: NAT

## Artikkelit ja niiden tiivistelmät

#### Karvinen 2026: [Apache installed with Ansible - quick notes](https://terokarvinen.com/apache-ansible/)
* Palvelimet usein asennetaan "package-file-service" -mallilla, jossa asennetaan ohjelma, kopioidaan sen asetustiedot ja käynnistetään palvelu.
* Ansible-roolissa nämä sitten jaetaan omiin kansioihinsa, kuten: `tasks/`, `files/`, `handlers/`
* Palvelu käynnistyy Ansiblessa vain, jos asetuksissa on tullut kunnon muutos.
* **Oma huomio:** Aika kiinnostavaa, että Ansible ei käynnistä palvelinta turhaan joka ikisellä ajokerralla, vaan vain silloin kun sille on tarvetta, elikkä kun asetuksia on muokattu.
#### Ansible Community Documentation: [Handlers: running operations on change](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html)
* Handlereita käytetään, vain silloin kun halutaan suorittaa jokin tehtävä ja vain, jos kohdekoneella on tehty muutoksia.
* Handlereita voidaan ryhmitellä `listen`-komennolla tietyn aiheen alle, jonka kautta sitten vain yksi kutsu voi käynnistää monta handleria.
* Niitä kutsutaan `notify`-komennolla. Tärkeimpänä, vaikka useampi tehtävä kutsuisi samaa handleria, se ajetaan aina vain sen yhden kerran.
#### Ansiblen sisäänrakennettu dokumentaatio (ansible-doc **service**)
* Moduuli hallitsee kohdekoneiden taustapalveluita (services), ja on yleiskäyttöinen, silllä se tukee automaattisesti useita eri käynnistysjärjestelmiä (`systemd` ja `sysv`)
  * **name:** Määrrittää palvelun nimen, jota hallitaan (kuten: `nginx`). **Pakollinen parametri!**
  * **enabled:** Määrittää sen, että käynnistyykö kyseinen palvelu automaattisesti, aina kun tietokone käynnistetään.
  * **state:** Määrittää halutun pavelun tilan (kuten: `started`, `stopped` tai `restarted`).
  * **Esimerkki:** Palvelun käynnistäminen ja asetus käynnistymään bootissa: `service: name=hhtpd state=started enabled=yes`
* **Oma huomio:** Todella kätevää että Ansible osaa itse päätellä kaiken taustalla että aikooko käyttää `systemctl` vai `service` komentoja, niin ei itse tarvitse kaikkia eri rooleja kirjoitella käsin eri käyttöjärjestelmille.

## Apachen asennus käsin

Tavoitteenamme on asentaa Apache ja tarjoilla sivut omasta kotihakemistosta, josta sitten ajamme nämä komennot terminaalissa.

1. Aloitetaan asentamalla Apache:

```bash
sudo apt update
sudo apt install apache2
```

<img width="703" height="328" alt="image" src="https://github.com/user-attachments/assets/643d3723-40a7-4070-96ea-6ecd0ee0d964" />

2. Luodaan sivusto omille tunnuksille **(ilman root oikeuksia!)**, luodaan samalla sinne yksinkertainen `index.html` tiedosto:
```bash
mkdir -p ~/sivukolme
echo "<h1> Ensimmäinen sivusto H3 tehtävää varten! </h1>" > ~/sivukolme/index.html
```

<img width="700" height="96" alt="image" src="https://github.com/user-attachments/assets/133b9a39-8a7a-436f-ac3b-0b9b4b48454a" />

3. Jotta Apache (jota ajaa `www-data` niminen käyttäjä)[^1]  voi näyttää sivun, annetaan luomille kansioille suoritusoikeus ja tiedostoon lukuoikeus muille käyttäjille:

```bash
chmod 711 ~/    # Kansio, omistajalla on kaikki oikeudet, kun taas muilla vain läpikulku
chmod 755 ~/sivukolme/    # Kansio, omistajalla kaikki, muilla luku ja läpikulku
chmod 644 ~/sivukolme/index.html    # Tiedosto, omistajalla luku ja kirjoitus oikeudet, muilla vain luku
```

<img width="304" height="139" alt="image" src="https://github.com/user-attachments/assets/7c1df572-742b-46f5-a9c8-0f48dc2208c0" />

4. Voimme nyt luoda Apachelle uuden konfiguraatiotiedoston komennolla `sudo micro /etc/apache2/sites-available/oma.conf` johon sitten lisäämme sivuston asetukset:

```Apache
<VirtualHost *:80>
    ServerName localhost
    DocumentRoot /home/blendi/sivukolme/

    <Directory /home/blendi/sivukolme/>
        Require all granted
    </Directory>
</VirtualHost>
```

<img width="356" height="131" alt="image" src="https://github.com/user-attachments/assets/3718e109-73d0-4616-b204-4fdf36ae04f6" />

5. Ollaan melkein valmiita, mutta ensin meidän pitää ottaa uusi sivu käyttöön, ja poistaa myös Apachen oletussivu käyöstä jolloin voimme jatkaa käynnistämällä palvelimen uudelleen:

```bash
sudo a2ensite oma.conf
sudo a2dissite 000-default.conf
sudo systemctl restart apache2
```

<img width="420" height="296" alt="image" src="https://github.com/user-attachments/assets/c1897394-3b56-4941-8943-28b243ed106d" />

6. Voimme seuraavaksi varmistaa sivuston toimivuus, lataamalla se komentoriviltä, mutta ennen sitä käynnistetään Apache uudelleen varmuuden vuoksi:

```bash
sudo systemctl restart apache2
curl localhost
```

Kuten näkyykin, saimme sen toimimaan! :)

<img width="407" height="80" alt="image" src="https://github.com/user-attachments/assets/8d346b77-b43d-4ecf-bda5-b78931acbf96" />

## Nginxin asennus käsin

Seuraavaksi voimme alkaa asentamaan Nginx, mutta koska kaksi ohjelmaa ei voi kuunnella samaa porttia (eli porttia 80), Apache täytyy sammuttaa ensin, ennen kun aloitamme Nginxin kanssa.

1. Sammutetaan Apache ja estetään sen automaattinen käynnistyminen:

```bash
sudo systemctl stop apache2
sudo systemctl disable apache2
```

<img width="863" height="123" alt="image" src="https://github.com/user-attachments/assets/6aebe6f7-850f-42f4-85b4-7f1321c3d44d" />

2. Voimme nyt aloittaa asentamaan Nginx:

```bash
sudo apt install nginx
```

<img width="730" height="470" alt="image" src="https://github.com/user-attachments/assets/284f4ae8-73b6-4638-bdbe-b952c13d3dac" />

3. Luodaan Nginxille oma asetustiedosto komennolla: `sudo micro /etc/nginx/sites-available/oma_nginx.conf`

_(**HUOM:** Koska loimme Weppisivun `~/publicsite/index.html` edellisessä tehtävässä, sitä ei tässä tehtävässä tarvitse uudelleen luoda!)_

```Nginx
server {
    listen 80;
    server_name localhost;
    root /home/blendi/sivukolme;
    index index.html;
}
```

<img width="295" height="105" alt="image" src="https://github.com/user-attachments/assets/43a5166e-3126-46d6-a4e7-bb011f1b1c77" />

4. Nginxissä sivustot otetaan käyttöön tekemällä niistä linkki `sites-enabled` kansioon, joten tehdään se ja poistetaan samalla Nginxin oletussivu:

```bash
sudo ln -s /etc/nginx/sites-available/oma_nginx.conf /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
```

<img width="674" height="95" alt="image" src="https://github.com/user-attachments/assets/e7c709d5-0ae8-4b6e-a08a-eb72159822bd" />

5. Seuraavaksi, tarkistetaan että Nginxin konfiguraatiossa ei ole minkäänlaisia kirjoitusvirheitä:

```bash
sudo nginx -t
```

<img width="544" height="77" alt="image" src="https://github.com/user-attachments/assets/0e88e8a2-2f7f-44c1-9ef5-12ffa1ffec4b" />

6. Voimme jatkaa testaus osioon, varmistetaan että Nginx toimii ja että se tarjoaa oikean sivun etusivulla ilman minkäänlaisia sudo-oikeuksia. Käynnistetään kuitenkin palvelin ensin uudelleen.

```bash
sudo systemctl restart nginx
curl localhost
```

<img width="417" height="117" alt="image" src="https://github.com/user-attachments/assets/d21e08c4-bf16-4af0-a373-87e5d5bcb1d3" />

## Nginxin asennus Ansiblen avulla

Lopuksi, voimme automatisoida Nginxin asennut hyödyntämällä aiemmin puhutusta "package-file-service" mallilla, Ansiblen avulla. Pidämme kuitenkin weppisivun HTML-tiedostot vieläkin käsin muokattavina!

1. Luodaan roolille sille tarvittava kansiorakenne:[^2]

```bash
mkdir -p ~/tansible/roles/nginx_server/{tasks,handlers,files}
```

<img width="548" height="52" alt="image" src="https://github.com/user-attachments/assets/e9633fe0-5ff3-472a-b995-34551cd66720" />

2. Tehdään Nginx-asetustiedoston ns. 'master copy' Ansiblen `files`-kansioon komennolla: `micro ~/tansible/roles/nginx_server/files/oma_nginx.conf` johon kopioimme tuon Nginx-asetustiedoston:

```Nginx
server {
    listen 80;
    server_name localhost;
    root /home/blendi/sivukolme;
    index index.html;
}
```

<img width="262" height="97" alt="image" src="https://github.com/user-attachments/assets/1eb079fd-a509-45b6-a412-e57334e6c119" />

3. Määritetään sen handler komennolla: `micro ~/tansible/roles/nginx_server/handlers/main.yml` joka sitten pystyy käynnistämään Nginxin uudelleen **vain tarvittaessa**:

```YAML
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

<img width="225" height="93" alt="image" src="https://github.com/user-attachments/assets/2e5ca680-fbe6-4b02-9bbc-77a855ffeee5" />

4. Nyt voimme siirtyä automaatiotehtäviin, jotka kirjoitamme itse `tasks/main.yml` -tiedostoon, komennolla: `micro ~/tansible/roles/nginx_server/tasks/main.yml`:

```YAML
- apt:
    name: nginx
    state: present

- copy:
    dest: "/etc/nginx/sites-available/oma_nginx.conf"
    src: "oma_nginx.conf"
    owner: "root"
    group: "root"
    mode: "0644"
  notify: restart nginx

- file:
    src: "/etc/nginx/sites-available/oma_nginx.conf"
    dest: "/etc/nginx/sites-enabled/oma_nginx.conf"
    state: link
  notify: restart nginx

- file:
    path: "/etc/nginx/sites-enabled/default"
    state: absent
  notify: restart nginx
```

<img width="455" height="355" alt="image" src="https://github.com/user-attachments/assets/d494d807-c32f-47d6-b962-87858658aed9" />

5. Jatketaan lisäämällä uusi rooli `nginx_server` pääpelikirjaamme komennolla: `micro ~/tansible/site.yml` ja ajetaan se sen jälkeen hallintakäyttäjällä _(muista ensin siirtyä oikealla hakemistolle!)_:

<img width="173" height="133" alt="image" src="https://github.com/user-attachments/assets/cc3333c8-c609-4802-80f8-ac4632d2d0f1" />

```bash
cd ~/tansible
ansible-playbook -i hosts.ini site.yml -u berry
```

<img width="999" height="526" alt="image" src="https://github.com/user-attachments/assets/694bf4c3-6ffd-4183-ab13-40db8f44f07f" />

## Yhteenveto
- **Palvelimet:** Asensimme Apachen ja Nginxin tarjoilemaan weppisivuja omasta kansiostamme.

- **Automaatio:** Automatisoimme Nginxin asennuksen Ansiblella, hyödyntäen "package-file-service" mallia.

- **Teoria:** Ymmärsimme kuinka tuo "package-file-service" malli toimii, ja kuinka Ansilen handlerit säästävät resursseja tekemällä uudelleenkäynnistyksen vain tarvittaessa.


## Lähteet
[Apache installed with Ansible - quick notes](https://terokarvinen.com/apache-ansible/)

[Handlers: running operations on change](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html)

Ansiblen sisäänrakennettu dokumentaatio (ansible-doc **service**)

[nginx documentation](https://nginx.org/en/docs/)

[Reddit julkaisu Nginx:istä](https://www.reddit.com/r/nginx/comments/mvatwk/what_is_nginx_explain_to_me_like_im_5_because_im/)

[Nginx cheatsheet, Vishnu Chilamakuru](https://dev.to/vishnuchilamakuru/nginx-cheatsheet-24ph)


[^1]: **www-data:** Tarkoittaa käyttäjää, jonka oikeuksillä weppipalvelintoimii, se on hyväksy tietoturvalle koska jos palvelin murretaan, hyökkääjä saa vain rajoitetut oikeudet. Tämän takia annoimme (others) vain oikeuden lukea tiedostoamme!
[^2]: **Miksi käytin {}:** Tämä tulee Bashin komentotulkinnasta "brace expansion". Sen avulla voimme säästää aikaa jos olemme luomassa useita kansioita kerrallaan, esimerkkinä, komento: `mkdir {a, b, c}` luo kansiot a, b ja c vain tuolla yhdellä komennolla.
