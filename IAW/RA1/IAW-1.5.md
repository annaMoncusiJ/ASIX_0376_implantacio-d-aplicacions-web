# IAW-1.5 · Repeteix el web amb Docker dins de Debian

**0376 IAW · RA1 · 3 hores**

## Què aconseguiràs?

Instal·laràs Docker dins de Debian i hi desplegaràs el mateix web en un contenidor. **Primer sense cap base de dades**, comprovant només el camí des de l’ordinador; a la darrera classe, un cop creat el contenidor de MariaDB a ASGBD 2.4, li afegiràs la base de dades connectant-lo a aquell contenidor per la xarxa comuna.

## Abans de començar

Reenviament VirtualBox del web Docker: `8081 → 8080`. El contenidor de bases de dades es fa a **ASGBD 2.4**, abans de la darrera classe d’aquesta activitat: fins aleshores el web funciona sense base de dades.
| Servei | Des del teu ordinador | Destinació dins de Debian |
|---|---|---|
| SSH | `127.0.0.1:8022` | `22` |
| Apache tradicional | `http://127.0.0.1:80` | `80` |
| MariaDB tradicional | `127.0.0.1:3306` | `3306` |
| Web Docker | `http://127.0.0.1:8081` | `8080 → contenidor:80` |
| MariaDB Docker | `127.0.0.1:3308` | `3307 → contenidor:3306` |

## Com has de treballar i lliurar

Desa una còpia d’**aquest mateix document** com `IAW-1.5_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## 1. Instal·la Docker i prepara el web dins de Debian · E1

### Què has de saber abans de començar
Una **imatge** conté el sistema de fitxers i la configuració base per crear un contenidor. Un **contenidor** és una instància en execució, amb processos aïllats que comparteixen el nucli del seu host: en aquest cas, **Debian**, no directament el sistema de l’ordinador.

El Dockerfile descriu com construir la imatge; Compose descriu serveis, ports, xarxes i muntatges. `ports` relaciona un port de Debian amb el port intern del contenidor. `volumes` pot muntar una carpeta o fitxer existent de Debian: això és un **bind mount**, no necessàriament un volum amb nom.

La xarxa comuna permet connectar contenidors entre si: dins d’una xarxa Docker definida per l’usuari, un contenidor en troba un altre pel **nom de servei**. Això és el que farà servir el web, a la darrera classe, per arribar al contenidor de bases de dades.

**Docker s’instal·la dins de Debian**, amb els repositoris APT, i són tres peces: el **dimoni** (`docker`), que és el procés que crea i executa contenidors; el **connector de Compose** (`docker compose`), que llegeix el fitxer YAML; i la **imatge**, que és el motlle. Un paquet instal·lat no vol pas dimoni actiu: comprova-ho. Si el dimoni no s’està executant, els errors seran del tipus *Cannot connect to the Docker daemon* i no tindran res a veure amb el teu fitxer Compose.

La xarxa `asix-lab` s’ha de crear **amb la subnet del laboratori** (172.30.76.0/24, passarel·la 172.30.76.1). Dins d’aquesta xarxa, el contenidor web rep la IP **172.30.76.10** i el de bases de dades la **172.30.76.20**: això és el que permetrà, a la darrera classe, que el web trobi el servei pel nom `db` i que el compte de la base de dades pugui reconèixer aquell origen.

**En aquesta primera part el web no té base de dades.** Només es publica l’HTML i un PHP que no consulta res. Això et deixa comprovar tota la cadena de ports sense dependre de cap servei més; la base de dades s’afegeix a E5.

## Què has de fer

Segueix els passos en ordre des de la màquina virtual Debian, connectat per SSH. Comprova que cada ordre acaba sense errors. En acabar, Docker ha d'estar actiu, la xarxa `asix-lab` ha d'existir i el web ha de quedar preparat per publicar-se al port 8080 de Debian.

### 1. Instal·la Docker Engine i Docker Compose v2 dins de Debian

Comprova que treballes amb Debian 12 (Bookworm) o Debian 13 (Trixie). Abans d'instal·lar la versió oficial, elimina els paquets que poden entrar-hi en conflicte. Aquesta ordre no elimina les imatges, els contenidors ni els volums de `/var/lib/docker`:

```bash
sudo apt remove -y docker.io docker-compose docker-doc docker-buildx podman-docker containerd runc
```

Primer, actualitza els paquets i instal·la les eines necessàries per afegir el repositori oficial de Docker:

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

Afegeix la clau GPG oficial de Docker:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Afegeix el repositori oficial de Docker a les fonts d'APT:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Actualitza els repositoris i instal·la Docker Engine, el connector de Compose i els components necessaris:

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Activa i inicia el servei Docker:

```bash
sudo systemctl enable --now docker
```

### 2. Comprova que la instal·lació és correcta

Executa les ordres següents:

```bash
docker --version
```

```bash
docker compose version
```

```bash
sudo systemctl is-active docker
```

```bash
sudo systemctl is-enabled docker
```

Les dues últimes ordres haurien de retornar, respectivament, `active` i `enabled`.

**Opcional:** per no haver de posar `sudo` a cada ordre, pots afegir el teu usuari al grup `docker`:

```bash
sudo usermod -aG docker $USER
```

Aquesta pertinença concedeix privilegis equivalents a root sobre el sistema. Si l'apliques, has de **tancar la sessió SSH i tornar-hi a entrar** perquè tingui efecte. Fins aleshores, continua utilitzant `sudo docker`.

### 3. Crea la xarxa del laboratori

Abans de crear-la, comprova que no existeix ja:

```bash
sudo docker network inspect asix-lab
```

Si la xarxa encara no existeix, crea-la amb la subnet convinguda:

```bash
sudo docker network create --driver bridge --subnet 172.30.76.0/24 --gateway 172.30.76.1 asix-lab
```

Comprova que la xarxa existeix i revisa'n la configuració:

```bash
sudo docker network ls
sudo docker network inspect asix-lab
```

Si la xarxa ja existia, no la tornis a crear. Comprova que utilitza `172.30.76.0/24` i que la passarel·la és `172.30.76.1`. Si hi ha un conflicte de subnet amb una altra xarxa, no n'inventis una de nova: consulta el professorat perquè la IP fixa del contenidor web depèn d'aquest rang.

### 4. Configura el reenviament de ports de VirtualBox

Afegeix una regla de reenviament de ports a la configuració NAT de VirtualBox per accedir al servidor web Docker. El port 80 de Debian ja l'utilitza l'Apache tradicional; per això Docker publicarà el web al port 8080 de Debian:

- IP de l'amfitrió: `127.0.0.1`

- Port de l'amfitrió: **8081**

- Port del convidat: **8080**

- Protocol: TCP

**No modifiquis la regla SSH existent `8022 → 22`.**

Si el centre no disposa d'accés a Docker Hub, copia el fitxer `asix-web-lab.tar` a Debian i carrega'l:

```bash
sudo docker load -i asix-web-lab.tar
```

**Important:** pregunta al professorat què conté el TAR. Si conté la imatge base `php:8.3-apache-bookworm`, podràs construir `asix-web:lab` sense descarregar-la. Si conté directament la imatge final `asix-web:lab`, no utilitzis `--build` a l'ordre del pas 7: inicia el contenidor amb `up -d`. Per construir sense connexió, el TAR ha d'incloure totes les imatges base necessàries. Si no conté cap d'aquestes imatges, la construcció no es podrà completar sense accés a Docker Hub.

### 5. Prepara la carpeta de treball

Dins de Debian, connectat per SSH, crea la carpeta de treball del projecte. Executa aquestes ordres amb el teu usuari habitual, no com a `root`.

**Copia només els dos fitxers que no necessiten base de dades.** El fitxer `db.php` s'afegirà a l'exercici E5.

```bash
mkdir -p ~/asix-web-docker/public/sense-index
```

```bash
cd ~/asix-web-docker
```

```bash
cp /var/www/asix/public/{index.html,prova.php} public/
```

### 6. Crea els fitxers de configuració de Docker

Dins de `~/asix-web-docker`, crea els fitxers següents amb els noms exactes indicats. Respecta especialment la indentació del YAML: els espais són significatius.

**Dockerfile.web:**

```dockerfile
FROM php:8.3-apache-bookworm
RUN docker-php-ext-install pdo_mysql
COPY asix-docker.conf /etc/apache2/conf-available/asix-lab.conf
RUN a2enconf asix-lab
```

**asix-docker.conf:**

```apache
<Directory /var/www/html>
    Options -Indexes
    AllowOverride None
    Require all granted
    DirectoryIndex index.html
</Directory>
```

**compose.web.yml:**

```yaml
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.web
    image: asix-web:lab
    ports:
      - "${WEB_PORT:-8080}:80"
    networks:
      lab:
        ipv4_address: 172.30.76.10
    volumes:
      - ./public:/var/www/html:ro
    restart: unless-stopped
networks:
  lab:
    external: true
    name: asix-lab
```

**.env.web:**

```ini
WEB_PORT=8080
```

### 7. Comprova i engega el projecte

La xarxa `asix-lab` ja està creada al pas 3. La imatge `asix-web:lab` es construeix a partir del `Dockerfile.web`.

Per construir la imatge i iniciar el servei:

```bash
sudo docker compose --env-file .env.web -f compose.web.yml up -d --build
```

Comprova que el contenidor està en funcionament:

```bash
sudo docker compose --env-file .env.web -f compose.web.yml ps
```

Consulta els registres del servei si necessites detectar algun problema:

```bash
sudo docker compose --env-file .env.web -f compose.web.yml logs
```

Des de l'ordinador amfitrió, obre el navegador i accedeix a:

```text
http://127.0.0.1:8081
```

**Important:** la IP `172.30.76.10` és interna a la xarxa Docker. No és una IP que hagis d'introduir al navegador de l'ordinador amfitrió.

En aquesta pràctica encara no hi ha cap muntatge de secrets ni connexió amb una base de dades. Aquests elements s'afegiran a l'exercici E5.

### Escriu la teva resposta aquí

**Quines versions de Docker Engine i Compose has instal·lat? Indica les sortides de versió:** …  

**Quina subnet i quina passarel·la té `asix-lab`? Com ho has comprovat?:** …  

**Quina funció fan Apache, PHP i l'extensió `pdo_mysql` dins la imatge?:** …  

**Per què instal·lem `pdo_mysql` ara si la base de dades no s'incorpora fins a E5?:** …  

**Quin port es publica a Debian (`8080`) i quin port escolta dins del contenidor (`80`)?:** …  

**Per què E1–E4 poden funcionar sense secrets ni comptes de base de dades?:** …  

**Quins canvis has fet respecte de la plantilla? No incloguis cap secret:** …

## 2. Comprova el doble reenviament · E2

### Què has de saber abans de començar

Amb Docker hi ha **dos reenviaments** abans d’arribar al web: VirtualBox porta l’ordinador:8081 a Debian:8080; Docker porta Debian:8080 al contenidor:80. Publicar un port a Docker no crea automàticament la regla de VirtualBox.

Una prova feta dins de Debian al port 8080 comprova el tram Docker; la prova feta a l’ordinador al 8081 comprova la cadena completa. **Encara no hi ha base de dades**, així que el que es comprova ara és només el camí HTTP: si aquesta cadena funciona, el reenviament HTTP està verificat; les fallades futures de la base de dades encara poden ser de xarxa interna, credencials, permisos o configuració. `compose ps` i `logs` mostren informació diferent i complementària: `ps` diu on escolta el contenidor, `logs` diu què li ha passat.

### Què has de fer

Completa les proves en l'ordre indicat. No confonguis la prova local de Debian amb la prova feta des de l'ordinador amfitrió. Des de Debian i situat a `~/asix-web-docker`, inicia el servei i comprova'n l'estat:

```bash
sudo docker compose --env-file .env.web -f compose.web.yml up -d
```

```bash
sudo docker compose --env-file .env.web -f compose.web.yml ps
```

```bash
sudo docker compose --env-file .env.web -f compose.web.yml logs --tail=15
```

Primer, des del terminal de Debian, prova `curl -i http://127.0.0.1:8080/prova.php`. Aquesta prova comprova el port publicat a Debian sense passar per VirtualBox. Després, des del terminal de l'**ordinador amfitrió** (no des de la sessió SSH), executa:

```bash
curl -i http://127.0.0.1:8081/index.html
```

```bash
curl -i http://127.0.0.1:8081/prova.php
```

Les dues peticions fetes des de l'ordinador han de retornar HTTP 200 i mostrar el contingut esperat. Si no és així, revisa `compose ps`, els registres i la regla de VirtualBox abans de continuar. Cadena web: **ordinador:8081 → VirtualBox → Debian:8080 → Docker → web:80**. No demanis `db.php`: encara no existeix al projecte.

### Escriu la teva resposta aquí
| Prova des de l’ordinador | HTTP | Marca real |
|---|---|---|
| index.html | … | … |
| prova.php | … | … |

**Escriu la cadena completa des del navegador fins al port del contenidor, indicant els tres ports:** …  

**Què valida la prova local de Debian? Què valida, addicionalment, la prova des de l'ordinador?:** …

## 3. Detecta un port Docker que no coincideix amb VirtualBox · E3

### Què has de saber abans de començar

El port de l’aplicació dins del contenidor, el publicat a Debian i el reenviat a l’ordinador són valors relacionats però independents. Canviar-ne un pot deixar l’aplicació operativa i, alhora, trencar l’accés des del client.

Per identificar el tram que falla, compara el port declarat al Compose amb el port efectiu de `compose ps`, després compara’l amb el convidat configurat a VirtualBox. La prova local des de Debian permet evitar el primer salt. Restaura el valor acordat, no afegeixis reenviaments improvisats per fer desaparèixer el símptoma sense entendre’l.

### Què has de fer

1. **Provoca el desajustament.** A Debian, obre `.env.web` i canvia `WEB_PORT=8080` per `WEB_PORT=8082`. Desa el fitxer i aplica el canvi:

   ```bash
   nano .env.web
   sudo docker compose --env-file .env.web -f compose.web.yml up -d
   ```
   No canviïs la regla de VirtualBox: continua reenviant `ordinador:8081 → Debian:8080`.
2. **Comprova el símptoma des de l'ordinador.** Executa `curl -i http://127.0.0.1:8081/prova.php`. La petició no ha d'arribar al contenidor perquè VirtualBox continua enviant-la al port 8080 de Debian, mentre Docker escolta al 8082.
3. **Localitza el tram que funciona.** A Debian, consulta `sudo docker compose --env-file .env.web -f compose.web.yml ps` i executa `curl -i http://127.0.0.1:8082/prova.php`. Anota el port publicat i el codi HTTP obtingut.
4. **Restaura la configuració.** Torna a posar `WEB_PORT=8080` a `.env.web` i aplica el canvi:
   ```bash
   sudo docker compose --env-file .env.web -f compose.web.yml up -d
   ```
   Des de l'ordinador, comprova que `http://127.0.0.1:8081/prova.php` torna a funcionar i que `http://127.0.0.1:8081/sense-index/` retorna HTTP 403. Aquest 403 és esperat: el directori existeix però no conté cap fitxer d'índex i Apache té desactivat el llistat de directoris.
### Escriu la teva resposta aquí
**Quin resultat obtens des de l'ordinador amb el port desajustat (codi HTTP o error)?:** …  
**Quin port publicat mostra `compose ps` i quin resultat dona `curl` a Debian?:** …  
**Quin salt de la cadena fallava i per què?:** …  
**Quin resultat obtens després de restaurar `WEB_PORT=8080` i per què la ruta `sense-index` retorna 403?:** …
## 4. Recrea el web sense perdre els fitxers · E4
### Què has de saber abans de començar
La capa escrivible d’un contenidor no és el lloc adequat per confiar-hi dades que s’han de conservar quan el substitueixes. En aquesta pràctica, les pàgines es troben en una carpeta de Debian que es munta al contenidor. Recrear el contenidor no elimina aquesta carpeta.
L’opció `:ro` fa el muntatge de només lectura **des del contenidor**; no impedeix que editis la carpeta des de Debian amb permisos suficients. Això explica que puguis canviar la pàgina a l’host i veure-la al web. Persistència no equival a còpia de seguretat: si esborres el fitxer host, el muntatge no el recupera.
### Què has de fer
1. A Debian, obre `~/asix-web-docker/public/index.html` i modifica o afegeix una frase identificativa. No eliminis ni alteris la marca `ASIX_WEB_OK`. Desa el fitxer.
2. Des del navegador de l'ordinador amfitrió, visita `http://127.0.0.1:8081/index.html` i comprova que hi apareix la frase nova.
3. Torna a Debian, situa't a `~/asix-web-docker` i força la recreació del contenidor:
```bash

sudo docker compose --env-file .env.web -f compose.web.yml up -d --force-recreate

```
4. Torna a carregar la pàgina des de l'ordinador i comprova que la frase continua apareixent. Explica per què sobreviu a la recreació del contenidor i quin seria el risc si s'eliminés el fitxer de Debian. `public/` és una carpeta de Debian muntada al contenidor en mode només lectura; no és una carpeta de l'ordinador físic ni una còpia de seguretat.
### Escriu la teva resposta aquí
**Quina frase identificativa has afegit a `index.html`?:** …  
**La frase apareixia abans i després de recrear el contenidor? Quin resultat has observat?:** …  
**Quina és la ruta del fitxer a Debian i per què el contingut es conserva en recrear el contenidor?:** …  
**Quina comprovació demostra que Apache serveix el fitxer muntat des de Debian i no una còpia incorporada a la imatge?:** …
## 5. Afegeix la base de dades i connecta el web al contenidor · E5
### Què has de saber abans de començar
A la darrera classe, el contenidor de bases de dades que vas preparar a **ASGBD 2.4** ha d'estar en funcionament i connectat a la xarxa externa `asix-lab`, amb l'àlies DNS `db`. Quan **web** i **db** comparteixen aquesta xarxa Docker, el web pot resoldre el nom `db` i connectar-se al port intern **3306**. La connexió entre contenidors no passa per l'ordinador ni pel reenviament 3308. Abans de començar, comprova a ASGBD 2.4 que el contenidor MariaDB continua actiu i connectat a `asix-lab` amb l'àlies `db`.
Afegir la base de dades vol dir **tres coses**: el fitxer de configuració amb el secret (fora del codi i muntat com a fitxer individual), la línia del muntatge al Compose, i el fitxer `db.php` que fa la consulta. El compte és el d’aplicació del **MariaDB Docker** (`webapp`), restringit a l’origen **172.30.76.10** —la IP fixa del contenidor web— i només amb lectura sobre la taula de comprovació. Per això la IP del web s’ha de mantenir fixada al Compose: si canviés, el compte deixaria de reconèixer aquell origen.
Les dades de prova poden tenir el mateix text que les del servidor tradicional, però els comptes, les contrasenyes i el directori de dades són independents: un resultat correcte no demostra per si sol amb quina instància parles.
La comparació final ha de distingir canvis d’implementació d’invariants: continuen existint HTTP, SQL, permisos, processos i necessitat de verificar. Docker facilita reproduir entorns, però no elimina l’administració ni garanteix seguretat automàtica.
### Què has de fer
A l'última classe, després d'ASGBD 2.4, treballa dins de `~/asix-web-docker`. Abans de modificar el web, comprova que MariaDB està actiu i comparteix la xarxa `asix-lab` amb el nom DNS `db`. Si no es compleix aquesta condició, atura't i revisa l'exercici ASGBD 2.4.
1. Copia al projecte el fitxer que fa la consulta. Abans de continuar, comprova que aquesta versió de `db.php` llegeix la configuració de `/etc/asix/db.ini` i utilitza les claus `host`, `port`, `database`, `user` i `password`. Si la plantilla lliurada a IAW 1.4 utilitza un altre camí o noms de claus, cal mantenir-los coherents entre el PHP i el fitxer de configuració:
```bash

cp /var/www/asix/public/db.php public/

```
2. Crea la carpeta `secrets` i el fitxer `secrets/db.ini`. El fitxer contindrà la configuració per connectar amb el **contenidor MariaDB Docker**, no amb el servidor tradicional. Crea'l amb el teu usuari i edita'l:
   ```bash
   install -d -m 0700 secrets
   nano secrets/db.ini
   ```
   Escriu-hi aquest contingut, substituint la contrasenya pel secret real del compte `webapp` creat a ASGBD 2.4:
   ```ini
   host = "db"
   port = "3306"
   database = "asix_lab"
   user = "webapp"
   password = "ESCRIU_AQUI_EL_SECRET_REAL"
   ```
   No copiïs el secret al document de lliurament. `db:3306` funciona perquè els dos contenidors són dins de la mateixa xarxa Docker. No utilitzis el port 3308 de l'ordinador per connectar els contenidors.
3. Comprova quin GID té l'usuari `www-data` dins de la imatge PHP:
   ```bash
   sudo docker run --rm --entrypoint id asix-web:lab www-data
   ```
   Utilitza el GID que apareix a la sortida. Si és `33`, executa:
   ```bash
   sudo chown root:33 secrets/db.ini
   sudo chmod 0640 secrets/db.ini
   ```
   Si el GID és diferent, substitueix `33` pel valor real. Així el procés PHP pot llegir el fitxer a través del grup, però no modificar-lo. El secret es munta com a fitxer individual i en mode només lectura.
4. **Afegeix el muntatge** al `compose.web.yml`, dins de `volumes`:
```yaml

      - ./secrets/db.ini:/etc/asix/db.ini:ro

```
5. Abans de recrear el web, comprova que el contenidor `web` resol el nom `db` a través de la xarxa compartida:
   ```bash
   sudo docker compose --env-file .env.web -f compose.web.yml exec web getent hosts db
   ```
   Si no apareix una adreça IP, revisa que MariaDB estigui connectat a `asix-lab` i que tingui l'àlies `db`. Quan la resolució funcioni, recrea el web i comprova la consulta des de l'ordinador:
```bash

sudo docker compose --env-file .env.web -f compose.web.yml up -d --force-recreate

```

```bash

curl -i http://127.0.0.1:8081/db.php

```

El resultat esperat és **HTTP 200** i el text **ASIX_DB_OK: fila-demo**. Si obtens HTTP 503 o un altre error, comprova per separat el muntatge del fitxer, els permisos de lectura, la resolució DNS `db`, l'estat de MariaDB, el nom de la base de dades i les credencials. No copiïs cap contrasenya ni cap contingut del fitxer secret a la resposta.

6. Completa la comparació i lliura el document. Als últims 30 min, qüestionari individual del RA1; no hi ha examen pràctic addicional.

### Escriu la teva resposta aquí

**Quin resultat HTTP i quin missatge retorna `db.php` en consultar `db:3306`? No incloguis el secret:** …  

**Quin compte d'aplicació s'utilitza i des de quina IP d'origen? Per què aquesta IP s'ha de mantenir fixa?:** …
| Aspecte | Manual dins de Debian | Docker dins de Debian | Què es manté? |
|---|---|---|---|
| Instal·lació de components | … | … | … |
| Servei/procés | … | … | … |
| Configuració | … | … | … |
| Camí des de l’ordinador | … | … | … |
| Camí web → BD | … | … | … |
| Persistència/recreació | … | … | … |

**Descriu un avantatge concret i una limitació que hagis observat en aquesta pràctica:** …

## Com es puntua aquesta pràctica

Cadascun dels cinc apartats val **0, 1 o 2 punts**

Aquesta pràctica representa el **15% del RA1**.