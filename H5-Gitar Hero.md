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

> <img width="2372" height="704" alt="KomentoketjuVisuaalisesti" src="https://github.com/user-attachments/assets/19f6b147-d655-4be1-be8d-50d0c75765b7" />
<p align="center"><small><i>Itse tehty kaavio</i></small></p>
