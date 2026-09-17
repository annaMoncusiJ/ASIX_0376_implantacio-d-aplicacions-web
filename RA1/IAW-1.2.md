# IAW-1.2 · Instal·la Apache/PHP a Debian i accedeix-hi des del teu ordinador
**0376 IAW · RA1 · 4 hores**

## Què aconseguiràs?
Instal·laràs el web a Debian i obriràs HTML/PHP des del navegador de l’ordinador a través de VirtualBox.

## Abans de començar
Debian d’IAW 1.1 amb SSH funcional, sudo i repositoris.

Treballaràs individualment amb una única VM Debian dins de l’ordinador. Els serveis tradicionals i Docker, quan correspongui, s’executen dins d’aquesta mateixa Debian.

1. Al terminal del teu **ordinador**, connecta’t així:
```bash
ssh alumne@127.0.0.1 -p 8022
```
> [!IMPORTANT]
> Tingues en compte que `alumne` és el nom d’usuari de l’exemple. Si el teu usuari de Debian té un altre nom, substitueix-lo a la comanda SSH. `sudo` pot demanar la contrasenya d’aquest usuari; no l’escriguis al lliurament.

2. Quan aparegui la sessió del teu usuari a Debian, executa-hi les comandes de Linux i `sudo`. **Totes les comandes de l’enunciat són dins de Debian, excepte les marcades explícitament «ordinador».**
3. Per obrir pàgines o fer les proves de client, utilitza el navegador/terminal del teu ordinador. Si és PowerShell i `curl` no és el client cURL real, utilitza `curl.exe`.

| Servei | Des del teu ordinador | Destinació dins de Debian |
|---|---|---|
| SSH | 127.0.0.1:8022 | 22 |
| Apache tradicional | http://127.0.0.1:80 | 80 |
| MariaDB tradicional | 127.0.0.1:3306 | 3306 |

**En la infraestructura inicial només està configurat el reenviament SSH 8022→22.** Afegeix els altres reenviaments quan s’indiqui a l’activitat a: VirtualBox → Configuració de la VM → Xarxa → NAT → Avançat → Reenviament de ports. Tots són TCP, amb **IP amfitrió 127.0.0.1**, port amfitrió i port convidat de la taula. Conserva les regles que ja hagis creat en activitats anteriors; no les dupliquis. El professorat t’indicarà si cal especificar la IP convidada.

> `127.0.0.1` identifica l’entorn on executes la comanda: l’ordinador, Debian o un contenidor. Fer servir el mateix número de port, com el 3306 de MariaDB tradicional, no converteix aquests entorns en un de sol. Si el port 3306 de l’ordinador ja està ocupat per un servei local, avisa abans de modificar-lo: no aturis serveis desconeguts ni canviïs ports sense acordar-ho.


Desa el punt de continuació abans de sortir de la sessió SSH. Pots demanar orientació al professorat, però fes personalment les configuracions i les proves. Llegeix la teoria de cada apartat abans d’executar els passos corresponents; és suport per entendre la pràctica, no un qüestionari ni un lliurament addicional.

## Com has de treballar i lliurar
Desa una còpia d’**aquest mateix document** com `IAW-1.2_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## 1. Identifica què instal·laràs · E1

### Què has de saber abans de començar
Un **servidor web** és un programa que escolta peticions HTTP i retorna recursos, com pàgines, imatges o respostes generades. Apache és aquest programa.
El **navegador** és el client: envia la petició i interpreta el contingut rebut.
**PHP** executa codi al servidor; el navegador rep el resultat, no ha de rebre el codi font.

- **Paquet:** fitxers de programari instal·lats i gestionats per Debian, per exemple `apache2`.
- **Procés:** programa que s’està executant en aquell moment.
- **Servei:** unitat que el sistema pot iniciar, aturar i supervisar; Apache pot utilitzar diversos processos.
- **Port d’escolta:** número que permet adreçar una connexió a un servei, com el TCP 80 d’Apache.

`hostname` identifica la màquina; `ip -br address` resumeix interfícies i adreces; `/etc/os-release` identifica la distribució. La IP NAT de Debian no és el `127.0.0.1` de l’ordinador. VirtualBox relaciona els dos entorns amb el reenviament de ports.

### Què has de fer
1. A la VM Debian, executa les comandes següents i anota la IP NAT que té dins de Debian.
2. Identifica per a què serveixen Apache, PHP i el navegador. 

```bash
hostname
```

```bash
ip -br address
```

```bash
cat /etc/os-release
```

### Escriu la teva resposta aquí
| Element | Funció (una frase) |
|---|---|
| Apache | … |
| PHP | … |
| Navegador | … |

**IP de Debian:** …  

## 2. Instal·la Apache/PHP i configura un lloc · E2

### Què has de saber abans de començar
Debian instal·la programari amb **APT**. `apt update` actualitza la llista de paquets disponibles; **no actualitza tots els paquets instal·lats**. `apt install` instal·la els paquets indicats i les dependències. `sudo` executa aquella comanda amb privilegis administratius.

En aquesta pràctica, `libapache2-mod-php` integra PHP amb Apache; `php-mysql` aporta el connector que s’utilitzarà més endavant per consultar MariaDB. Instal·lar el connector **no instal·la el servidor de bases de dades**.

La configuració del lloc té aquests elements:
- `<VirtualHost *:80>` defineix el lloc per a peticions rebudes al port 80. **No obre per si sol el port:** l’escolta es defineix amb `Listen`.
- `ServerName` identifica el nom previst del lloc. Escriure `asix.test` aquí **no crea DNS ni una entrada al fitxer hosts**. En aquesta pràctica accedim per IP/port.
- `DocumentRoot` és la carpeta publicada per HTTP.
- `<Directory>` estableix les regles d’accés a una carpeta del sistema de fitxers.
- `Options -Indexes` impedeix mostrar un llistat si no hi ha fitxer índex.
- `AllowOverride None` impedeix sobreescriure aquestes opcions amb `.htaccess`.
- `Require all granted` permet l’accés al contingut d’aquest directori, subjecte a la resta de controls.
- `DirectoryIndex` indica el fitxer que s’intenta mostrar en demanar un directori.
- `ErrorLog` i `CustomLog` indiquen els registres d’errors i d’accés.
- `a2ensite` activa un lloc disponible.
- `configtest` comprova la sintaxi
- `reload` torna a carregar la configuració sense equivaler a una aturada completa.
- `enable --now` habilita l’arrencada i inicia ara el servei.

### Què has de fer
1. Afegeix a VirtualBox el reenviament web **TCP, IP amfitrió 127.0.0.1, port amfitrió 80, port convidat 80**. No modifiquis la regla SSH.
| Nom | Protocol | IP amfitrió | Port amfitrió | IP convidat | Port convidat |
   |-----|----------|-------------|---------------|-------------|----------------|
   | `HTTP` | `TCP` | `127.0.0.1` | `80` |  | `80` |
2. Executa a **Debian**:

```bash
sudo apt update
```

```bash
sudo apt install apache2 libapache2-mod-php php-mysql curl -y
```

```bash
sudo systemctl enable --now apache2
```

```bash
systemctl is-active apache2
```

```bash
php --version
```

3. Creem un directori i li assignem els permisos que necessitem
```bash
sudo mkdir -p /var/www/asix/public/sense-index
```
```bash
sudo chmod 0755 /var/www/asix/public/sense-index
```

4. Obre `sudo nano /var/www/asix/public/index.html` i desa:

```html
<!doctype html>
<html lang="ca"><meta charset="utf-8"><title>Prova ASIX</title>
<h1>ASIX_WEB_OK</h1><p>Servidor de proves.</p></html>
```

5. Obre `sudo nano /etc/apache2/sites-available/asix.conf` i desa aquesta configuració:

```apache
<VirtualHost *:80>
    ServerName asix.test
    DocumentRoot /var/www/asix/public
    <Directory /var/www/asix/public>
        Options -Indexes
        AllowOverride None
        Require all granted
        DirectoryIndex index.html
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/asix-error.log
    CustomLog ${APACHE_LOG_DIR}/asix-access.log combined
</VirtualHost>
```

6. Activa el lloc, valida la sintaxi i recarrega Apache. **Si configtest mostra un error, corregeix-lo abans de recarregar.** La desactivació de 000-default afecta només el lloc predeterminat del laboratori.

```bash
sudo chmod 0644 /var/www/asix/public/index.html
```

```bash
sudo a2ensite asix.conf
```

```bash
sudo a2dissite 000-default.conf
```

```bash
sudo apachectl configtest
```

```bash
sudo systemctl reload apache2
```

```bash
systemctl is-active apache2
```

```bash
sudo ss -ltnp
```

```bash
sudo apachectl -M
```

Has d’obtenir `Syntax OK`, servei `active` i escolta al port 80. Una versió diferent de PHP no és un error si és la versió indicada per al laboratori.

### Escriu la teva resposta aquí
**DocumentRoot configurat:** …  
**Mòdul PHP identificat:** …
```text
Enganxa aquí Syntax OK, estat del servei, línia del port i primera línia de php --version.
```
**Si has canviat alguna directiva respecte del model, indica quina i per què:** …

## 3. Publica PHP i comprova’l des de l’ordinador · E3

### Què has de saber abans de començar
En una **resposta estàtica**, Apache llegeix el fitxer HTML i en retorna el contingut. En una **resposta dinàmica**, PHP s’executa dins del servidor i genera la resposta abans d’enviar-la. El navegador rep HTTP en tots dos casos.

`curl -i` mostra les capçaleres HTTP i el cos. El codi **200** indica que la petició s’ha atès correctament, però per demostrar la funcionalitat també has de comprovar el contingut. Per això es demanen marques i una data generada per PHP.

VS Code Remote SSH permet editar i treballar amb fitxers de Debian des de l’ordinador. El terminal remot executa a Debian; un terminal local de l’ordinador és un entorn diferent. La connexió SSH utilitza el port 8022 de l’ordinador (22 de la màquina que redirigim amb virtual box), mentre que el navegador usa el 80 del web.

### Què has de fer
1. Ara editarem el projecte, es pot fer servir un editor específic, però per ara amb `nano` en tenim prou. No executis l’editor com a root: edita una còpia anomenada `prova.php` al teu directori personal (`~/prova.php`).
2. Pública-la al directori web executant el següent comandament (això crearà una còpia de `prova.php` a `/var/www/asix/public/prova.php` amb permisos `0644` de cop):

```bash
sudo install -m 0644 ~/prova.php /var/www/asix/public/prova.php
```

El fitxer ha de contenir exactament aquest codi, per ara no cal entendre'l, ho treballarem més endavant:

```php
<?php
header('Content-Type: text/plain; charset=utf-8');
echo "ASIX_PHP_OK\n";
echo date(DATE_ATOM) . "\n";
```
3. Des de l'**ordinador amfitrió**, executa segons el teu sistema operatiu:

<details>
<summary><b>Opcions per a Linux</b></summary>

```bash
curl -i http://127.0.0.1:80/index.html
```

```bash
curl -i http://127.0.0.1:80/prova.php
```
</details>

<details>
<summary><b>Opcions per a Windows</b></summary>

*Nota: S'utilitza `curl.exe` per evitar que PowerShell confongui el comandament amb `Invoke-WebRequest`.*

```powershell
curl.exe -i http://127.0.0.1:80/index.html
```

```powershell
curl.exe -i http://127.0.0.1:80/prova.php
```
</details>

Les dues respostes han de tenir codi **200**. La primera ha de mostrar `ASIX_WEB_OK`; la segona, `ASIX_PHP_OK` i la data calculada al servidor. Si veus el codi font PHP, el processament PHP no està ben configurat.

4. Entra a les següents url des del teu navegador:
    - 127.0.0.1
    - 127.0.0.1/index.html
    - 127.0.0.1/prova.php

### Escriu la teva resposta aquí
**Client des d’on proves i entorn integrat utilitzat:** …
| URL | Codi HTTP observat | Text retornat |
|---|---|---|
| /index.html | … | … |
| /prova.php | … | … |

**Per què la data de prova.php es calcula al servidor i no al navegador?**
...

**Quina diferència hi ha quan al navegador accedim a 127.0.0.1 i a 127.0.0.1/index.html?**
...

## 4. Comprova que no es mostren directoris ni secrets · E4

### Què has de saber abans de començar
Els permisos Linux determinen qui pot llegir, escriure i executar o travessar un recurs. Es divideixen en tres categories: propietari (usuari que el crea), grup (conjunt d'usuaris) i altres (qualsevol altre usuari o servei com el servidor web Apache).

Els valors es llegeixen sumant els codis d'accés per a cada categoria d'usuari: `r=4` (lectura), `w=2` (escriptura), `x=1` (execució o travessa). 

| Permís | Valor | En un Fitxer | En un Directori (Carpeta) |
| :---: | :---: | :--- | :--- |
| **`r`** | **4** | Permet llegir o obrir el fitxer per veure'n el contingut. | Permet llistar els fitxers que hi ha a dins (`ls`). |
| **`w`** | **2** | Permet modificar o esborrar el contingut del fitxer. | Permet crear, moure o eliminar fitxers a l'interior. |
| **`x`** | **1** | Permet **executar** el fitxer si és un script o binari. | Permet **travessar** el directori (entrar-hi amb `cd` o accedir als seus fitxers). |

En un **directori**, el permís `x` permet travessar-lo per accedir als fitxers que conté; si falta aquest permís, ningú podrà accedir al seu contingut encara que els fitxers interns siguin públics. En un fitxer, `x` permet executar-lo com un programa, la qual cosa no és el mateix que permetre’n la lectura.

Per exemple, **0644** és lectura/escriptura per al propietari (`4+2=6`) i només lectura (`4`) per al grup i els altres; és la configuració estàndard per a fitxers web. El valor **0755** afegeix travessa/execució (`4+2+1=7` i `4+1=5`), habitual en directoris públics per permetre que Apache pugui entrar a buscar les pàgines. 

El zero (`0`) inicial indica que no hi ha cap permís especial actiu (com SUID, SGID o Sticky Bit), assegurant que es neteja qualsevol configuració especial prèvia. Són valors orientatius per a contingut no secret, no permisos universals.

Un **403** és una resposta HTTP que denega l’accés. Pot provenir d’una regla d’Apache o de permisos del sistema. `chmod 777` ampliaria innecessàriament qui pot modificar el contingut. Un secret necessita permisos restrictius i, a més, quedar fora del directori publicat.

### Què has de fer
1. Comprova que `sense-index` existeix i és buit.
2. Des de l’**ordinador**, accedeix a `127.0.0.1:80/sense-index`. Has d’obtenir **403**: Apache no ha de llistar el directori.
>[!TIP]
> Accedint des del navegador no podem veure sempre el codi que ens ha tornat. Per poder-lo veure sense fer curl hem de fer clic dret -> inspeccionar -> Network i recarregar la pàgina

3. Revisa la directiva `Options -Indexes` dins el fitxer de configuració  i els permisos del lloc (amb `ls -l` es poden veure els permisos d'un lloc). No apliquis `chmod 777` per fer desaparèixer errors.
4. No publiquis cap contrasenya ni fitxer privat dins de `/var/www/asix/public`.

### Escriu la teva resposta aquí
**Resultat HTTP de /sense-index/:** …  
**Directiva que evita el llistat:** …  
**Per què 777 no és una solució correcta?** …  
**On desaries un fitxer privat de configuració perquè Apache no el publiqui?** …

## 5. Relaciona una petició amb el log i documenta · E5

### Què has de saber abans de començar
El **registre d’accés** anota peticions rebudes: origen, URL, moment i resultat, segons el format configurat. El **registre d’errors** recull problemes del servidor o de l’aplicació. No totes les respostes d’error HTTP generen necessàriament una línia al registre d’errors.

Un **404** indica que el servidor HTTP ha respost però no ha trobat el recurs demanat; no demostra que Apache estigui aturat. Per diagnosticar, relaciona l’hora i la URL que has demanat amb la línia del registre.

Un procediment reproduïble explica **què fas, on ho fas i com saps que ha funcionat**. Una llista de comandes sense ordre, context ni resultats esperats no és suficient per lliurar el servei a un altre tècnic.

### Què has de fer
1. Des de l’**ordinador**, demana `http://127.0.0.1:80/no-existeix`. És normal obtenir **404**.
2. A Debian, consulta:

```bash
sudo tail -n 15 /var/log/apache2/asix-access.log
```

```bash
sudo tail -n 15 /var/log/apache2/asix-error.log
```

3. Localitza la petició 404 al registre d’accés. El registre d’error pot no tenir cap missatge per aquest 404.
4. Canvia una frase d’index.html, torna a obrir la pàgina i comprova el canvi.
5. Escriu un procediment de màxim **12 passos curts** que permeti a un altre tècnic repetir la instal·lació i les proves. Conserva Debian operativa per a l’activitat següent.

### Escriu la teva resposta aquí
**Fragment del registre amb la petició 404:**
```text
…
```
**Quina frase has canviat i quin resultat has comprovat?** …  
**Procediment (màxim 12 passos):**
1. …


## Com es puntua aquesta pràctica
Cadascun dels cinc apartats val **0, 1 o 2 punts**: 2 si és complet i verificat; 1 si hi ha una part correcta però falta una comprovació o justificació; 0 si no hi ha resposta o és incorrecta. Total: **10 punts**.

| Apartat | Què es comprova |
|---|---|
| E1 | Components i recorregut client/servidor. |
| E2 | Instal·lació, configuració, servei i port. |
| E3 | Execució real d’HTML/PHP des de l’ordinador i entorn integrat. |
| E4 | Restricció d’accés i permisos. |
| E5 | Registre interpretat i procediment reproduïble. |

Aquesta pràctica representa el **10% del RA1**. 
