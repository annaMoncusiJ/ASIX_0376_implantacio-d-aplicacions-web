# IAW-1.5 · Repeteix el web amb Docker dins de Debian

**0376 IAW · RA1 · 3 hores**

## Què aconseguiràs?

Instal·laràs Docker dins de Debian i hi desplegaràs el mateix web en un contenidor. **Primer sense cap base de dades**, comprovant només el camí des de l’ordinador; a la darrera classe, un cop creat MariaDB a ASGBD 2.4, integraràs els dos serveis en un únic projecte Docker Compose.

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

**Docker s’instal·la dins de Debian**, amb els repositoris APT, i són tres peces: el **dimoni** (`docker`), que és el procés que crea i executa contenidors; el **connector de Compose** (`docker compose`), que llegeix el fitxer YAML; i la **imatge**, que és el motlle. Un paquet instal·lat no vol pas dimoni actiu: comprova-ho. Si el dimoni no s’està executant, els errors seran del tipus **Cannot connect to the Docker daemon** i no tindran res a veure amb el teu fitxer Compose.

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

**docker-compose.yml:**

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
sudo docker compose --env-file .env.web up -d --build
```

Comprova que el contenidor està en funcionament:

```bash
sudo docker compose --env-file .env.web ps
```

Consulta els registres del servei si necessites detectar algun problema:

```bash
sudo docker compose --env-file .env.web logs
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
sudo docker compose --env-file .env.web up -d
```

```bash
sudo docker compose --env-file .env.web ps
```

```bash
sudo docker compose --env-file .env.web logs --tail=15
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

   sudo docker compose --env-file .env.web up -d
   ```

   No canviïs la regla de VirtualBox: continua reenviant `ordinador:8081 → Debian:8080`.

2. **Comprova el símptoma des de l'ordinador.** Executa `curl -i http://127.0.0.1:8081/prova.php`. La petició no ha d'arribar al contenidor perquè VirtualBox continua enviant-la al port 8080 de Debian, mentre Docker escolta al 8082.

3. **Localitza el tram que funciona.** A Debian, consulta `sudo docker compose --env-file .env.web ps` i executa `curl -i http://127.0.0.1:8082/prova.php`. Anota el port publicat i el codi HTTP obtingut.

4. **Restaura la configuració.** Torna a posar `WEB_PORT=8080` a `.env.web` i aplica el canvi:

   ```bash
   sudo docker compose --env-file .env.web up -d
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
sudo docker compose --env-file .env.web up -d --force-recreate
```

4. Torna a carregar la pàgina des de l'ordinador i comprova que la frase continua apareixent. Explica per què sobreviu a la recreació del contenidor i quin seria el risc si s'eliminés el fitxer de Debian. `public/` és una carpeta de Debian muntada al contenidor en mode només lectura; no és una carpeta de l'ordinador físic ni una còpia de seguretat.

### Escriu la teva resposta aquí
**Quina frase identificativa has afegit a `index.html`?:** …  
**La frase apareixia abans i després de recrear el contenidor? Quin resultat has observat?:** …  
**Quina és la ruta del fitxer a Debian i per què el contingut es conserva en recrear el contenidor?:** …  
**Quina comprovació demostra que Apache serveix el fitxer muntat des de Debian i no una còpia incorporada a la imatge?:** …

## 5. Integra el web i MariaDB en un únic projecte Docker Compose · E5

### Què has de saber abans de començar

A ASGBD-2.4 has creat i inicialitzat un contenidor MariaDB independent. Ara integrarem aquest servei al projecte del web perquè **un únic fitxer `docker-compose.yml` gestioni els dos contenidors**.

No instal·larem una segona base de dades ni en crearem una de nova. Conservarem les dades que ja has introduït a ASGBD-2.4 mitjançant el volum Docker amb nom explícit `asix_mariadb_data`.

Al final tindrem aquesta arquitectura:

```text
Ordinador amfitrió
    |
    | http://127.0.0.1:8081
    v
VirtualBox:8081 -> Debian:8080
                      |
                      v
              Docker Compose
              projecte asix-web
                 /       \
                /         \
        web:80             db:3306
        172.30.76.10       172.30.76.20
                \         /
                 asix-lab
```

Els dos serveis compartiran la xarxa externa `asix-lab`. El web trobarà MariaDB pel nom de servei `db` i el port intern `3306`. El port `3307` només serveix per accedir a MariaDB des de Debian; el `3308` és el reenviament de VirtualBox per accedir-hi des de l'ordinador.

**Abans de començar, comprova que:**

- El contenidor MariaDB de ASGBD-2.4 està en funcionament.
- El contenidor MariaDB té la IP `172.30.76.20` i està connectat a `asix-lab`.
- El volum de dades es diu `asix_mariadb_data`.
- Existeix el compte `webapp` amb permís `SELECT` sobre `asix_lab.comprovacio`, restringit a l'origen `172.30.76.10`.
- Coneixes la contrasenya de `webapp`, desada a `~/asix-bd-docker/secrets/webapp_password.txt`.

No continuïs si aquestes condicions no es compleixen. Revisa ASGBD-2.4 abans de modificar el desplegament.

### Què has de fer

Treballa des de Debian. En aquesta activitat farem servir `~/asix-web-docker` com a carpeta del projecte unificat.

**1. Prepara una còpia dels fitxers de MariaDB dins del projecte web.**

Crea una carpeta `db` i copia-hi la configuració, els scripts d'inicialització i els fitxers de secrets creats a ASGBD-2.4:

```bash
mkdir -p ~/asix-web-docker/db
cp -a ~/asix-bd-docker/config ~/asix-web-docker/db/
cp -a ~/asix-bd-docker/init ~/asix-web-docker/db/
cp -a ~/asix-bd-docker/secrets ~/asix-web-docker/db/
```

Comprova l'estructura:

```bash
cd ~/asix-web-docker
find db -maxdepth 2 -type f -print
```

Hauries de trobar els fitxers de configuració, l'SQL d'inicialització i els tres fitxers de contrasenyes. No mostris el contingut dels secrets.

**2. Prepara el fitxer de configuració de la connexió PHP.**

Copia el PHP que fa la consulta:

```bash
cp /var/www/asix/public/db.php public/
```

Comprova que aquesta versió de `db.php` llegeix `/etc/asix/db.ini` i utilitza les claus `host`, `port`, `database`, `user` i `password`.

Crea el fitxer de configuració:

```bash
mkdir -p secrets
chmod 700 secrets
nano secrets/db.ini
```

Escriu-hi el contingut següent i substitueix el valor de la contrasenya pel secret real de `webapp`:

```ini
host = "db"
port = "3306"
database = "asix_lab"
user = "webapp"
password = "ESCRIU_AQUI_EL_SECRET_REAL"
```

No copiïs el secret al document de lliurament.

Protegeix el fitxer perquè només el propietari i el grup el puguin llegir:

```bash
sudo docker run --rm --entrypoint id asix-web:lab www-data
```

Anota el GID que apareix a la sortida. Si és `33`, executa:

```bash
sudo chown root:33 secrets/db.ini
sudo chmod 0640 secrets/db.ini
```

Si el GID és diferent, substitueix `33` pel valor real. Així el procés PHP pot llegir el fitxer a través del grup, però no modificar-lo.

**3. Atura el desplegament independent de MariaDB sense esborrar les dades.**

Entra a la carpeta del projecte de bases de dades i atura'n els contenidors:

```bash
cd ~/asix-bd-docker
sudo docker compose down
```

Aquesta ordre elimina el contenidor MariaDB, però **no elimina el volum `asix_mariadb_data`**. No utilitzis `down -v` ni eliminis el volum manualment.

Comprova que el volum continua existint:

```bash
sudo docker volume inspect asix_mariadb_data
```

Torna al projecte web:

```bash
cd ~/asix-web-docker
```

**4. Crea el fitxer únic `docker-compose.yml`.**

Aquest fitxer substituirà, per al desplegament final, l'ús separat de `docker-compose.yml` i el Compose de MariaDB. Inclourà els dos serveis:

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
      - ./secrets/db.ini:/etc/asix/db.ini:ro
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
  db:
    image: mariadb:11.4
    container_name: mariadb_docker
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD_FILE: /run/secrets/root_password
      MARIADB_DATABASE: asix_lab
      MARIADB_USER: clientlab
      MARIADB_PASSWORD_FILE: /run/secrets/user_password
    secrets:
      - root_password
      - user_password
    ports:
      - "3307:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
      - ./db/config/60-asix.cnf:/etc/mysql/conf.d/60-asix.cnf:ro
      - ./db/init:/docker-entrypoint-initdb.d:ro
    networks:
      lab:
        ipv4_address: 172.30.76.20
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s
volumes:
  mariadb_data:
    external: true
    name: asix_mariadb_data
secrets:
  root_password:
    file: ./db/secrets/root_password.txt
  user_password:
    file: ./db/secrets/user_password.txt
networks:
  lab:
    external: true
    name: asix-lab
```

Desa'l com `docker-compose.yml` a `~/asix-web-docker`.

Observa les parts més importants:

- `web` i `db` són dos serveis del mateix projecte Compose.
- Tots dos comparteixen la xarxa externa `asix-lab` i mantenen les IP fixes.
- El nom de servei `db` permet que PHP localitzi MariaDB.
- `depends_on` amb `service_healthy` fa que Compose esperi que MariaDB superi la comprovació de salut abans d'iniciar el web.
- `mariadb_data` és un volum extern: Compose l'utilitza, però no n'assumeix la creació ni l'eliminació.
- Els fitxers de configuració i secrets continuen separats de la imatge i es munten dins dels contenidors.

El servei `db` conserva el volum que ja contenia les dades. Per això, els scripts de `db/init` **no es tornaran a executar**: només s'executen quan MariaDB inicialitza un directori de dades buit.

**5. Comprova la configuració i inicia els dos serveis.**

Des de `~/asix-web-docker`, valida el fitxer:

```bash
sudo docker compose --env-file .env.web config
```

Si no hi ha errors, inicia el projecte:

```bash
sudo docker compose --env-file .env.web up -d --build
```

Comprova els dos serveis:

```bash
sudo docker compose --env-file .env.web ps
```

Consulta els registres si algun servei no està actiu:

```bash
sudo docker compose --env-file .env.web logs --tail=30
```

Comprova que els dos contenidors estan connectats a la xarxa:

```bash
sudo docker network inspect asix-lab
```

Hauries de trobar el web a `172.30.76.10` i MariaDB a `172.30.76.20`.

**6. Comprova la resolució del nom i la connexió a la base de dades.**

Des del servei web, comprova que el nom `db` es resol:

```bash
sudo docker compose --env-file .env.web exec web getent hosts db
```

La resposta ha d'incloure l'adreça `172.30.76.20`.

Ara, des de l'ordinador amfitrió, executa:

```bash
curl -i http://127.0.0.1:8081/db.php
```

El resultat esperat és **HTTP 200** i el text **`ASIX_DB_OK: fila-demo`**.

Si no funciona, comprova per separat:

- Que els dos serveis estiguin actius i saludables.
- Que comparteixin la xarxa `asix-lab`.
- Que `db` es resolgui a `172.30.76.20`.
- Que `secrets/db.ini` estigui muntat i PHP el pugui llegir.
- Que el compte `webapp` tingui la contrasenya correcta i el permís `SELECT`.
- Que la taula `asix_lab.comprovacio` contingui la fila de prova.

**7. Comprova que el projecte unificat es gestiona amb una sola ordre.**

Des de `~/asix-web-docker`, executa:

```bash
sudo docker compose --env-file .env.web ps
```

Aquesta ordre ha de mostrar els serveis `web` i `db`. A partir d'ara, per iniciar, aturar o consultar el projecte, treballa des d'aquesta carpeta i utilitza aquest `docker-compose.yml`.

Per aturar els dos serveis sense eliminar les dades:

```bash
sudo docker compose --env-file .env.web down
```

Per tornar-los a iniciar:

```bash
sudo docker compose --env-file .env.web up -d
```

No utilitzis `down -v`: el volum `asix_mariadb_data` conté les dades de MariaDB.

### Escriu la teva resposta aquí

**Quins dos serveis gestiona el nou `docker-compose.yml`?:** …

**Quina IP té cada servei i quina xarxa comparteixen?:** …

**Quin resultat obtens amb `getent hosts db`?:** …

**Quin resultat HTTP i quin missatge retorna `db.php`?:** …

**Quin volum utilitza MariaDB? Com has comprovat que és el mateix que havies creat a ASGBD-2.4?:** …

**Per què hem aturat el Compose anterior amb `down` i no amb `down -v`?:** …

**Quin avantatge té gestionar web i base de dades des d'un únic projecte Compose?:** …

**Completa la comparació:**

| Aspecte | Manual dins de Debian | Docker dins de Debian | Què es manté? |
|---|---|---|---|
| Instal·lació de components | … | … | … |
| Servei/procés | … | … | … |
| Configuració | … | … | … |
| Camí des de l'ordinador | … | … | … |
| Camí web → BD | … | … | … |
| Persistència/recreació | … | … | … |

**Descriu un avantatge concret i una limitació que hagis observat en aquesta pràctica:** …

## Com es puntua aquesta pràctica

Cadascun dels cinc apartats val **0, 1 o 2 punts**

Aquesta pràctica representa el **15% del RA1**.