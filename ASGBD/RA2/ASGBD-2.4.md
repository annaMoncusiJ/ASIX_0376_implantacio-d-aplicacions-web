# ASGBD-2.4 · Repeteix MariaDB amb Docker

**0377 ASGBD · RA2 · 1 hora**

## Què aconseguiràs?

Ja has instal·lat MariaDB directament a Debian. Ara repetirem el procés, però aquesta vegada utilitzant Docker.

Construiràs una segona instància de MariaDB dins d'un contenidor, que funcionarà simultàniament amb el servidor tradicional.

A més, deixaràs aquesta instància preparada perquè, a l'última classe d'IAW-1.5, el servidor web que has creat pugui connectar-s'hi.

## Abans de començar

El MariaDB tradicional ja està instal·lat i funciona al port 3306. **No l'aturis**, perquè volem tenir les dues instàncies funcionant simultàniament.

A l'activitat IAW-1.5 ja has instal·lat Docker i has creat la xarxa `asix-lab`. També hi has connectat el contenidor web, que té l'adreça `172.30.76.10`.

Ara hi afegirem MariaDB amb l'adreça `172.30.76.20`.

| Servei | Des de l'ordinador | Destinació dins de Debian |
|---|---|---|
| SSH | `127.0.0.1:8022` | `22` |
| Apache tradicional | `http://127.0.0.1:80` | `80` |
| MariaDB tradicional | `127.0.0.1:3306` | `3306` |
| Web Docker | `http://127.0.0.1:8081` | `8080 → contenidor:80` |
| MariaDB Docker | `127.0.0.1:3308` | `3307 → contenidor:3306` |

**Important:** la xarxa `asix-lab` ja existeix perquè l'has creat a IAW-1.5. No la tornis a crear ni en modifiquis la configuració.

El MariaDB Docker haurà de tenir:

- IP interna: `172.30.76.20`.

- Nom de servei Docker: `db`.

- Port intern: `3306`.

- Port publicat a Debian: `3307`.

El reenviament de VirtualBox `3308 → 3307` permetrà accedir-hi des de l'ordinador.

## Com has de treballar i lliurar

Desa una còpia d'aquest mateix document com `ASGBD-2.4_COGNOMS_NOM.md`.

Completa únicament on s'especifica i lliura només aquest fitxer. Enganxa fragments curts, no captures redundants ni logs complets.

Els fitxers de configuració i els secrets es queden a Debian. No lliuris contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

---

## 1. Busca una imatge i prepara el projecte Docker · E1

### Què has de saber abans de començar

Per executar MariaDB amb Docker necessitem una imatge que contingui el servidor de bases de dades.

Com ja hem fet amb NGINX i Apache, no cal instal·lar manualment MariaDB dins del contenidor. Docker descarregarà una imatge que ja conté tot el necessari.

Anem a [Docker Hub](https://hub.docker.com/) i busquem MariaDB.

Comprova que existeix una imatge oficial de MariaDB i observa les diferents versions disponibles.

És important seleccionar una versió concreta. Si utilitzem simplement `mariadb`, Docker descarregarà la versió predeterminada i això podria provocar canvis de comportament entre instal·lacions.

### Què has de fer

**1. Crea una carpeta per al projecte:**

```bash
mkdir -p ~/asix-bd-docker
cd ~/asix-bd-docker
```

**2. Crea l'estructura de directoris:**

```bash
mkdir config init secrets
```

El projecte tindrà aquesta estructura:

```text
asix-bd-docker/
├── docker-compose.yml
├── config/
│   └── 60-asix.cnf
├── init/
│   └── 01-dades.sql
└── secrets/
    ├── root_password.txt
    ├── user_password.txt
    └── webapp_password.txt
```

**3. Busca la imatge oficial de MariaDB a Docker Hub.**

Selecciona la versió `11.4` i consulta'n les característiques.

La imatge que utilitzarem serà la última, en aquest cas:

```text
mariadb:11.4
```

**5. Crea el fitxer `docker-compose.yml`:**

```bash
nano docker-compose.yml
```

De moment, no hi escriguis res. El completarem a l'apartat següent.

### Escriu la teva resposta aquí

**Nom de la imatge oficial:** …

**Versió seleccionada:** …

**Per què és millor indicar una versió concreta en lloc d'utilitzar `latest`?:** …

---

## 2. Crea i configura el contenidor MariaDB · E2

### Què has de saber abans de començar

Docker Compose permet definir en un fitxer YAML les característiques d'un o diversos contenidors.

En el nostre cas necessitem indicar:

- La imatge que utilitzarem.

- El nom del contenidor.

- Els ports que volem publicar.

- Els volums que muntarem.

- Els secrets que necessitem.

- La xarxa Docker a la qual pertanyerà.

També necessitem indicar les credencials inicials del servidor.

En lloc d'escriure contrasenyes directament al manifest, utilitzarem fitxers de secrets. Així, les contrasenyes queden separades de la configuració principal del contenidor.

### Què has de fer

**1. Completa el fitxer `docker-compose.yml` que has creat abans.**

```yaml
services:
  db:
    image: mariadb:11.4
    container_name: mariadb_docker
    restart: always
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
      - ./config/60-asix.cnf:/etc/mysql/conf.d/60-asix.cnf:ro
      - ./init:/docker-entrypoint-initdb.d:ro
    networks:
      asix-lab:
        ipv4_address: 172.30.76.20
volumes:
  mariadb_data:
secrets:
  root_password:
    file: ./secrets/root_password.txt
  user_password:
    file: ./secrets/user_password.txt
networks:
  asix-lab:
    external: true
    name: asix-lab
```

Fixa't en alguns detalls:

- `image`: indica la imatge que descarregarem.

- `container_name`: permet identificar el contenidor fàcilment a Portainer.

- `restart`: indica que Docker ha d'intentar reiniciar-lo si s'atura.

- `ports`: publica el port 3306 del contenidor al 3307 de Debian.

- `volumes`: defineix els directoris i fitxers que es munten al contenidor.

- `secrets`: proporciona les contrasenyes mitjançant fitxers.

- `networks`: connecta MariaDB a la xarxa compartida amb el contenidor web.

Observa especialment aquesta part:

```yaml
networks:
  asix-lab:
    ipv4_address: 172.30.76.20
```

Això és imprescindible perquè el contenidor web pugui identificar l'origen de la connexió i perquè l'usuari `webapp` pugui tenir restringit l'accés a la IP del contenidor web.

El nom del servei és `db`. Docker permet que els altres contenidors de la mateixa xarxa el localitzin utilitzant aquest nom.

**2. Crea els fitxers de contrasenyes.**

```bash
nano secrets/root_password.txt
nano secrets/user_password.txt
```

Escriu una contrasenya de pràctiques a cadascun. No utilitzis contrasenyes personals.

Protegeix els fitxers per a que només tu hi tinguis accés:

```bash
chmod 600 secrets/*.txt
```

**3. Comprova que la xarxa existeix.**

```bash
sudo docker network inspect asix-lab
```

Comprova que la subnet és `172.30.76.0/24` i que la passarel·la és `172.30.76.1`.

**4. Comprova la configuració de Compose:**

```bash
sudo docker compose config
```

Aquesta ordre permet comprovar si el fitxer YAML és correcte i si Docker pot interpretar-ne la configuració.

**5. Descarrega la imatge i aixeca el contenidor:**

```bash
sudo docker compose up -d
```

**6. Comprova que s'ha creat correctament:**

```bash
sudo docker compose ps
```

```bash
sudo docker images
```

Obre Portainer i comprova que hi apareix el contenidor `mariadb_docker`.

Si el servidor encara s'està inicialitzant, pots consultar els registres:

```bash
sudo docker compose logs --tail=30 db
```

**7. Comprova la IP del contenidor:**

```bash
sudo docker network inspect asix-lab
```

Busca el contenidor `mariadb_docker` i comprova que té l'adreça `172.30.76.20`.

### Escriu la teva resposta aquí

**Nom del contenidor:** …

**Imatge utilitzada:** …

**Ports publicats:** …

**IP interna del contenidor:** …

**Estat del contenidor:** …

**Quina diferència hi ha entre el port 3307 i el 3306?:** …

---

## 3. Configura MariaDB i prepara l'accés de l'aplicació web · E3

### Què has de saber abans de començar

Igual que quan instal·lem MariaDB tradicionalment, podem modificar diferents paràmetres de configuració del servidor.

En Docker no cal entrar al contenidor i modificar manualment els fitxers interns. Podem preparar un fitxer de configuració a Debian i muntar-lo dins del contenidor.

Això ens permet mantenir la configuració separada de la imatge.

També podem preparar scripts SQL que s'executaran automàticament quan MariaDB inicialitzi un directori de dades nou.

Aquests scripts s'utilitzen habitualment per crear bases de dades, taules i dades inicials.

Finalment, necessitem crear un usuari específic perquè el contenidor web d'IAW-1.5 pugui consultar les dades.

Aquest usuari no serà administrador. Només tindrà permís de lectura sobre la taula que necessitarà l'aplicació.

### Què has de fer

**1. Crea el fitxer de configuració.**

```bash
nano config/60-asix.cnf
```

Introdueix-hi:

```ini
[mariadb]
default_storage_engine=InnoDB
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
max_connections=30
connect_timeout=10
wait_timeout=120
```

Aquest fitxer modifica el comportament del servidor MariaDB.

**2. Prepara un script SQL d'inicialització.**

```bash
nano init/01-dades.sql
```

Introdueix-hi:

```sql
USE asix_lab;
CREATE TABLE comprovacio (
    id INT PRIMARY KEY,
    etiqueta VARCHAR(50)
);
INSERT INTO comprovacio VALUES
(1, 'fila-demo');
```

Aquest script crearà una taula i hi inserirà una dada de prova.

Recorda que els scripts de `/docker-entrypoint-initdb.d` s'executen quan s'inicialitza un directori de dades nou. No s'executen automàticament cada vegada que reiniciem el contenidor.

**3. Torna a iniciar el projecte perquè MariaDB utilitzi els fitxers de configuració i inicialització.**

Com que el contenidor pot haver començat a inicialitzar-se abans de crear aquests fitxers, atura'l i torna'l a aixecar:

```bash
sudo docker compose down
sudo docker compose up -d
```

No utilitzis `down -v`, ja que eliminaria també el volum de dades.

**4. Comprova la configuració efectiva del servidor.**

Podem entrar al client MariaDB directament des del contenidor:

```bash
sudo docker compose exec db mariadb -u root -p
```

Introdueix la contrasenya de root que has creat abans.

Executa:

```sql
SHOW GLOBAL VARIABLES WHERE Variable_name IN (
'default_storage_engine',
'character_set_server',
'collation_server',
'max_connections',
'connect_timeout',
'wait_timeout'
);
```

Comprova que els valors coincideixen amb els que has escrit al fitxer `.cnf`.

**5. Comprova que s'ha creat la base de dades i la taula.**

```sql
SHOW DATABASES;
USE asix_lab;
SHOW TABLES;
SELECT * FROM comprovacio;
```

Hauries d'obtenir la fila `fila-demo`.

**6. Crea l'usuari que utilitzarà l'aplicació web.**

Encara dins del client MariaDB, executa:

```sql
CREATE USER 'webapp'@'172.30.76.10'
IDENTIFIED BY 'CONTRASENYA_DE_PROVA';
GRANT SELECT ON asix_lab.comprovacio
TO 'webapp'@'172.30.76.10';
SHOW GRANTS FOR 'webapp'@'172.30.76.10';
```

Substitueix `CONTRASENYA_DE_PROVA` per una contrasenya que triïs. Utilitza una contrasenya sense cometes simples per evitar problemes amb la sentència SQL.

**Important:** aquesta contrasenya l'hauràs de reutilitzar a l'activitat IAW-1.5, quan creïs el fitxer `secrets/db.ini`.

Desa-la en un fitxer protegit:

```bash
nano secrets/webapp_password.txt
```

Escriu-hi la mateixa contrasenya i protegeix-lo:

```bash
chmod 600 secrets/webapp_password.txt
```

No la lliuris ni la copiïs al document.

Observa els permisos que acabes de concedir: `webapp` només pot fer consultes `SELECT` sobre la taula `comprovacio`. No pot crear, modificar ni eliminar dades.

Surt del client:

```sql
exit;
```

### Escriu la teva resposta aquí

**Paràmetres de configuració verificats:** …

**Base de dades i taula creades:** …

**Resultat de la consulta SELECT:** …

**Compte webapp: origen i permisos concedits, sense contrasenya:** …

**Per què hem restringit l'accés a la IP 172.30.76.10?:** …

**Quan s'executen els scripts d'inicialització?:** …

---

## 4. Comprova l'accés i la persistència de les dades · E4

### Què has de saber abans de començar

Un dels avantatges més importants de Docker és que podem eliminar i recrear contenidors sense haver de perdre necessàriament les dades.

Per aconseguir-ho utilitzem volums.

En el nostre cas hem definit:

```yaml
volumes:
  - mariadb_data:/var/lib/mysql
```

Això indica que el directori on MariaDB desa les seves dades està associat a un volum Docker.

Per tant, encara que eliminem el contenidor, el volum continuarà existint i es podrà tornar a muntar quan creem un contenidor nou.

Cal distingir entre eliminar un contenidor i eliminar un volum. Són dues operacions diferents.

### Què has de fer

**1. Comprova el reenviament de ports des de l'ordinador.**

Des del PowerShell del teu ordinador executa:

```powershell
Test-NetConnection -ComputerName 127.0.0.1 -Port 3308
```

Si tot funciona, hauries d'obtenir:

```text
TcpTestSucceeded : True
```

Això indica que el port és accessible des del teu ordinador.

**2. Comprova la connexió des de Debian.**

Connecta't al MariaDB Docker amb el compte `clientlab`:

```bash
mariadb --protocol=tcp -h 127.0.0.1 -P 3307 -u clientlab -p
```

Executa:

```sql
SELECT USER(), CURRENT_USER();
SELECT * FROM asix_lab.comprovacio;
```

Comprova que pots consultar les dades.

Recorda que aquesta connexió es fa al port 3307 de Debian, no al port 3306 del MariaDB tradicional.

**3. Comprova el volum existent.**

```bash
sudo docker volume ls
```

Busca el volum `asix_mariadb_data`. Com que li hem assignat un nom explícit, no tindrà el prefix automàtic del projecte Compose.

També pots consultar-ne les característiques:

```bash
sudo docker volume inspect NOMBRE_DEL_VOLUM
```

El nom real pot incloure un prefix relacionat amb el nom del projecte Compose.

**4. Elimina i recrea el contenidor.**

Des de la carpeta del projecte:

```bash
sudo docker compose down
```

Aquesta comanda elimina els contenidors del projecte, però conserva el volum.

Torna a aixecar MariaDB:

```bash
sudo docker compose up -d
```

**5. Comprova que les dades continuen existint.**

```bash
sudo docker compose ps
```

Torna a connectar-te:

```bash
mariadb --protocol=tcp -h 127.0.0.1 -P 3307 -u clientlab -p -e "SELECT * FROM asix_lab.comprovacio;"
```

La dada `fila-demo` ha de continuar existint.

**Important:** no executis `docker compose down -v`, ja que aquesta opció elimina també els volums associats al projecte. Tampoc eliminis manualment el volum.

### Escriu la teva resposta aquí

**Resultat de Test-NetConnection des de l'ordinador:** …

**Resultat de USER() i CURRENT_USER():** …

**Resultat de la consulta abans de recrear:** …

**Nom del volum utilitzat:** …

**Resultat de la consulta després de recrear:** …

**Per què no hem perdut les dades?:** …

**Quina diferència hi ha entre eliminar un contenidor i eliminar un volum?:** …

---

## 5. Compara les dues instal·lacions · E5

### Què has de saber abans de començar

Ara ja has treballat amb dues instal·lacions de MariaDB:

- Una instal·lació tradicional a Debian.

- Una instal·lació dins d'un contenidor Docker.

Les dues poden executar el mateix motor de bases de dades, però la manera d'instal·lar-les, configurar-les, iniciar-les i conservar-ne les dades és diferent.

Docker ens permet empaquetar serveis en contenidors i facilita la reproducció d'entorns. Tanmateix, també hem de gestionar correctament les imatges, els volums, les xarxes i els ports.

A més, has deixat preparat el MariaDB Docker perquè el web d'IAW-1.5 s'hi pugui connectar. Recorda que aquesta connexió es farà utilitzant el nom `db` i el port intern `3306`, no el port publicat `3307`.

### Què has de fer

Completa la taula comparant les dues instal·lacions. Utilitza les comprovacions que has fet durant la pràctica.

| Aspecte | MariaDB tradicional | MariaDB Docker |
|---|---|---|
| Instal·lació | ... | ... |
| Gestió del servei | ... | ... |
| Configuració del motor | ... | ... |
| Port d'accés | ... | ... |
| Ubicació de les dades | ... | ... |
| Persistència | ... | ... |
| Eliminació i recreació | ... | ... |

**Un avantatge de treballar amb Docker:** …

**Una limitació o dificultat que has trobat:** …

**Una situació en què utilitzaries Docker en lloc d'una instal·lació tradicional:** …

### Com es puntua aquesta pràctica

Cadascun dels cinc apartats val **0, 1 o 2 punts**.

Aquesta pràctica representa el **10% del RA2**.
