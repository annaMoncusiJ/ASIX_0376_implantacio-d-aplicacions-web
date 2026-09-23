# Guia Bàsica de Comandes

Aquesta guia recull les comandes essencials per navegar, gestionar fitxers i treballar còmodament en un servidor Debian (o qualsevol sistema basat en Linux).

---

## Índex

1. [Navegació per directoris](#1-navegació-per-directoris)
2. [Llistar contingut](#2-llistar-contingut)
3. [Treballar amb fitxers i directoris](#3-treballar-amb-fitxers-i-directoris)
4. [Veure contingut de fitxers](#4-veure-contingut-de-fitxers)
5. [Cercar fitxers i text](#5-cercar-fitxers-i-text)
6. [Permisos i propietat](#6-permisos-i-propietat)
7. [Informació del sistema](#7-informació-del-sistema)
8. [Gestió de paquets (APT)](#8-gestió-de-paquets-apt)
9. [Xarxa](#9-xarxa)
10. [Serveis (systemd)](#10-serveis-systemd)
11. [Compressió i arxius](#11-compressió-i-arxius)
12. [Dreceres útils de terminal](#12-dreceres-útils-de-terminal)
13. [Comandes d'aplicacions web](#13-comandes-daplicacions-web)
14. [Consells pràctics](#consells-pràctics)
15. [Jerarquia de directoris més important](#jerarquia-de-directoris-més-important)

---

<a id="1-navegació-per-directoris"></a>
## 📁 1. Navegació per directoris

| Comanda | Descripció |
|---------|------------|
| `pwd` | Mostra el directori actual (Print Working Directory) |
| `cd /` | Vés al directori arrel |
| `cd ~` | Vés al directori personal de l'usuari (`/home/usuari`) |
| `cd ..` | Pujar un nivell (directori pare) |
| `cd -` | Tornar al directori anterior |
| `cd /etc/apt` | Anar a un directori concret (camí absolut) |
| `cd Documents` | Anar a un subdirectori (camí relatiu) |

---

<a id="2-llistar-contingut"></a>
## 📋 2. Llistar contingut

| Comanda | Descripció |
|---------|------------|
| `ls` | Llista els fitxers i carpetes del directori actual |
| `ls -la` | Llista TOTS els fitxers (inclosos els ocults) amb detalls (permisos, mida, data) |
| `ls -lh` | Llista amb mides llegibles per humans (KB, MB, GB) |
| `ls -lt` | Ordena per data de modificació (els més recents primer) |
| `ls /var/log` | Llista el contingut d'un directori concret sense moure-te'n |

---

<a id="3-treballar-amb-fitxers-i-directoris"></a>
## 📄 3. Treballar amb fitxers i directoris

### Crear i eliminar

| Comanda | Descripció |
|---------|------------|
| `mkdir nom_carpeta` | Crea un nou directori |
| `mkdir -p projectes/python/src` | Crea una estructura de directoris sencera |
| `touch fitxer.txt` | Crea un fitxer buit o actualitza la seva data |
| `rm fitxer.txt` | Elimina un fitxer |
| `rm -r carpeta/` | Elimina un directori i TOT el seu contingut (⚠️ amb compte!) |
| `rm -i fitxer.txt` | Demana confirmació abans d'eliminar |
| `rmdir carpeta_buida` | Elimina un directori NOMÉS si està buit |

### Copiar i moure

| Comanda | Descripció |
|---------|------------|
| `cp fitxer.txt còpia.txt` | Copia un fitxer |
| `cp -r carpeta_origen/ carpeta_destí/` | Copia un directori sencer recursivament |
| `mv fitxer.txt carpeta/` | Mou un fitxer a un altre directori |
| `mv nom_vell.txt nom_nou.txt` | Reanomena un fitxer |

---

<a id="4-veure-contingut-de-fitxers"></a>
## 🔍 4. Veure contingut de fitxers

| Comanda | Descripció |
|---------|------------|
| `cat fitxer.txt` | Mostra tot el contingut del fitxer per pantalla |
| `less fitxer.txt` | Mostra el contingut paginat (espai per avançar, `q` per sortir) |
| `more fitxer.txt` | Similar a `less`, més bàsic |
| `head -n 20 fitxer.txt` | Mostra les primeres 20 línies |
| `tail -n 20 fitxer.txt` | Mostra les últimes 20 línies |
| `tail -f /var/log/syslog` | Mostra les últimes línies i va mostrant les noves en temps real |
| `nano fitxer.txt` | Editor de text senzill per terminal (Ctrl+O per guardar, Ctrl+X per sortir) |
| `vim fitxer.txt` | Editor més potent (`i` per editar, `:wq` per guardar i sortir) |

---

<a id="5-cercar-fitxers-i-text"></a>
## 🔎 5. Cercar fitxers i text

| Comanda | Descripció |
|---------|------------|
| `find / -name "*.conf"` | Cerca fitxers que acabin en `.conf` des de l'arrel |
| `find /home/usuari -name "*.log"` | Cerca fitxers `.log` dins del directori personal |
| `grep "error" fitxer.log` | Cerca la paraula "error" dins d'un fitxer |
| `grep -r "error" /var/log/` | Cerca "error" de forma recursiva en tots els fitxers de la carpeta |
| `grep -i "error" fitxer.log` | Cerca ignorant majúscules/minúscules |
| `grep -n "error" fitxer.log` | Mostra el número de línia on hi ha la coincidència |
| `which apt` | Mostra el camí complet d'un programa |

---

<a id="6-permisos-i-propietat"></a>
## 🔐 6. Permisos i propietat

| Comanda | Descripció |
|---------|------------|
| `ls -l fitxer.txt` | Mostra els permisos, propietari i grup d'un fitxer |
| `chmod 755 script.sh` | Canvia els permisos (rwxr-xr-x) |
| `chmod +x script.sh` | Afegeix permís d'execució |
| `chown usuari:grup fitxer.txt` | Canvia el propietari i el grup |
| `chown -R usuari:grup carpeta/` | Canvia la propietat recursivament |
| `sudo comanda` | Executa la comanda com a superusuari (root) |
| `su -` | Inicia sessió com a root (`exit` per tornar) |

---

<a id="7-informació-del-sistema"></a>
## ⚙️ 7. Informació del sistema

| Comanda | Descripció |
|---------|------------|
| `uname -a` | Mostra informació del nucli (kernel) |
| `hostnamectl` | Mostra informació del sistema (Debian versió, hostname, kernel) |
| `df -h` | Mostra l'espai en disc usat i disponible (humà llegible) |
| `du -sh carpeta/` | Mostra la mida total d'un directori |
| `free -h` | Mostra l'ús de memòria RAM i swap |
| `top` | Monitor de processos en temps real (premeu `q` per sortir) |
| `htop` | Versió millorada de `top` (cal instal·lar-la: `sudo apt install htop`) |
| `uptime` | Mostra quant de temps fa que el servidor està en marxa |
| `whoami` | Mostra quin usuari ets |
| `id` | Mostra el teu UID, GID i grups |
| `w` | Mostra qui està connectat i què està fent |

---

<a id="8-gestió-de-paquets-apt"></a>
## 📦 8. Gestió de paquets (APT)

| Comanda | Descripció |
|---------|------------|
| `sudo apt update` | Actualitza la llista de paquets disponibles |
| `sudo apt upgrade` | Instal·la les actualitzacions de tots els paquets |
| `sudo apt install nom_paquet` | Instal·la un paquet |
| `sudo apt remove nom_paquet` | Desinstal·la un paquet (manté configuracions) |
| `sudo apt purge nom_paquet` | Desinstal·la un paquet ELIMINANT les configuracions |
| `sudo apt autoremove` | Elimina paquets que ja no calen |
| `apt search paraula` | Cerca un paquet al repositori |
| `apt show nom_paquet` | Mostra informació detallada d'un paquet |
| `dpkg -l` | Llista tots els paquets instal·lats |

---

<a id="9-xarxa"></a>
## 🌐 9. Xarxa

| Comanda | Descripció |
|---------|------------|
| `ip a` | Mostra les interfícies de xarxa i les seves IPs |
| `ip r` | Mostra la taula d'encaminament |
| `ping google.com` | Comprova la connectivitat amb un host |
| `ss -tuln` | Mostra els ports que estan escoltant (TCP/UDP) |
| `curl ifconfig.me` | Mostra la teva IP pública |
| `systemctl restart networking` | Reinicia el servei de xarxa |

---

<a id="10-serveis-systemd"></a>
## 🚀 10. Serveis (systemd)

| Comanda | Descripció |
|---------|------------|
| `systemctl status nginx` | Mostra l'estat d'un servei |
| `sudo systemctl start nginx` | Engega un servei |
| `sudo systemctl stop nginx` | Atura un servei |
| `sudo systemctl restart nginx` | Reinicia un servei |
| `sudo systemctl enable nginx` | Fa que el servei s'inicii amb el sistema |
| `sudo systemctl disable nginx` | Desactiva l'inici automàtic |
| `journalctl -u nginx` | Mostra els logs d'un servei |
| `journalctl -u nginx -f` | Mostra els logs d'un servei en temps real |

---

<a id="11-compressió-i-arxius"></a>
## 🗜️ 11. Compressió i arxius

| Comanda | Descripció |
|---------|------------|
| `tar -cvf arxiu.tar carpeta/` | Crea un arxiu TAR |
| `tar -xvf arxiu.tar` | Extreu un arxiu TAR |
| `tar -czvf arxiu.tar.gz carpeta/` | Crea un TAR comprimit amb gzip |
| `tar -xzvf arxiu.tar.gz` | Extreu un TAR.GZ |
| `zip -r arxiu.zip carpeta/` | Crea un ZIP (cal instal·lar `zip`) |
| `unzip arxiu.zip` | Extreu un ZIP |
| `gzip fitxer.txt` | Comprimeix un fitxer amb gzip |

---

<a id="12-dreceres-útils-de-terminal"></a>
## ⌨️ 12. Dreceres útils de terminal

| Drecera | Acció |
|---------|-------|
| `Ctrl + C` | Interromp / atura el procés actual |
| `Ctrl + D` | Tanca la sessió / surt de la terminal |
| `Ctrl + L` | Neteja la pantalla (equivalent a `clear`) |
| `Ctrl + A` | Mou el cursor al començament de la línia |
| `Ctrl + E` | Mou el cursor al final de la línia |
| `Ctrl + R` | Cerca una comanda anterior a l'historial |
| `↑` / `↓` | Navegar per les comandes anteriors |
| `Tab` | Autocompleta noms de fitxers/carpetes |
| `historial` | Mostra l'historial de comandes |
| `clear` | Neteja la pantalla |
| `exit` | Tanca la sessió SSH / terminal |

---

<a id="13-comandes-daplicacions-web"></a>
## 🌍 13. Comandes útils d'aplicacions web

### Servidors web (Nginx / Apache)

| Comanda | Descripció |
|---------|------------|
| `sudo apt install nginx` | Instal·la Nginx |
| `sudo systemctl start nginx` | Engega Nginx |
| `sudo systemctl reload nginx` | Recarrega la configuració sense tallar connexions |
| `sudo nginx -t` | Comprova que la configuració de Nginx és correcta |
| `sudo a2ensite nom_lloc` | (Apache) Activa un lloc virtual |
| `sudo a2dissite nom_lloc` | (Apache) Desactiva un lloc virtual |
| `sudo a2enmod rewrite` | (Apache) Activa un mòdul (ex. rewrite) |
| `sudo systemctl reload apache2` | (Apache) Recarrega la configuració |

### Certificats SSL (Let's Encrypt)

| Comanda | Descripció |
|---------|------------|
| `sudo apt install certbot python3-certbot-nginx` | Instal·la Certbot per Nginx |
| `sudo certbot --nginx -d exemple.com -d www.exemple.com` | Genera i configura un certificat SSL |
| `sudo certbot renew --dry-run` | Comprova que la renovació automàtica funcionarà |

### Tallafocs (UFW)

| Comanda | Descripció |
|---------|------------|
| `sudo apt install ufw` | Instal·la UFW |
| `sudo ufw allow 22` | Permet el tràfic SSH (port 22) |
| `sudo ufw allow 8022/tcp` | Permet un port SSH personalitzat (ex. 8022) |
| `sudo ufw allow 80/tcp` | Permet HTTP |
| `sudo ufw allow 443/tcp` | Permet HTTPS |
| `sudo ufw enable` | Activa el tallafocs |
| `sudo ufw status` | Mostra les regles actives |
| `sudo ufw delete allow 22` | Elimina una regla |

### Desplegament de codi

| Comanda | Descripció |
|---------|------------|
| `rsync -avz ./local/ usuari@ip:/var/www/app/` | Sincronitza fitxers amb el servidor (molt eficient) |
| `rsync -avz -e "ssh -p 8022" ./local/ usuari@ip:/var/www/app/` | Rsync amb port SSH personalitzat |
| `git clone https://github.com/usuari/repo.git` | Clona un repositori Git |
| `git pull origin main` | Descarrega la darrera versió del codi |
| `sudo chown -R www-data:www-data /var/www/` | Assigna la propietat al servidor web |
| `sudo chmod -R 755 /var/www/` | Ajusta els permisos dels fitxers web |

### Processos i aplicacions en execució

| Comanda | Descripció |
|---------|------------|
| `ps aux \| grep node` | Cerca processos de Node.js en execució |
| `kill PID` | Atura un procés pel seu PID |
| `kill -9 PID` | Força l'aturada d'un procés |
| `nohup python3 app.py &` | Executa una app en segon pla (resisteix tancar la sessió) |
| `sudo apt install nodejs npm` | Instal·la Node.js i npm |
| `npm install` | Instal·la les dependències d'un projecte Node |
| `npm run build` | Construeix una app (React, Vue, Next, etc.) |
| `pip install -r requirements.txt` | Instal·la dependències Python |
| `python3 -m venv venv && source venv/bin/activate` | Crea i activa un entorn virtual Python |

### Diagnosi i logs

| Comanda | Descripció |
|---------|------------|
| `sudo tail -f /var/log/nginx/access.log` | Logs d'accés de Nginx en temps real |
| `sudo tail -f /var/log/nginx/error.log` | Logs d'errors de Nginx |
| `sudo tail -f /var/log/apache2/error.log` | Logs d'errors d'Apache |
| `ss -tuln \| grep :80` | Comprova quin procés escolta al port 80 |
| `curl -I http://localhost` | Comprova la resposta HTTP sense descarregar la pàgina |
| `systemctl list-units --type=service --state=running` | Llista tots els serveis en execució |

---

<a id="consells-pràctics"></a>
## 💡 Consells pràctics

1. **Connectar-se per SSH**:
   ```bash
   # Port per defecte (22)
   ssh usuari@ip_del_servidor

   # Amb un port personalitzat (ex. 8022)
   ssh -p 8022 usuari@ip_del_servidor
   ```

2. **Pujar/baixar fitxers amb SCP**:
   ```bash
   # Enviar un fitxer al servidor
   scp fitxer.txt usuari@ip_servidor:/home/usuari/

   # Descarregar un fitxer del servidor
   scp usuari@ip_servidor:/home/usuari/fitxer.txt ./

   # Amb port personalitzat (majúscula P!)
   scp -P 8022 fitxer.txt usuari@ip_servidor:/home/usuari/
   ```

3. **Ser root amb compte**: Utilitza `sudo` en comptes de treballar directament com a root. Evita `rm -rf /` i comprova sempre la comanda abans de prémer Enter.

4. **Manuals**: Consulta l'ajuda de qualsevol comanda amb:
   ```bash
   man nom_comanda
   nom_comanda --help
   ```

5. **Historial**: Les comandes queden desades a `~/.bash_history`. Pots executar la comanda número 42 amb `!42`.

---

<a id="jerarquia-de-directoris-més-important"></a>
## 📁 Jerarquia de directoris més important

```
/           → Arrel del sistema
├── /home/  → Directoris personals dels usuaris
├── /etc/   → Fitxers de configuració
├── /var/   → Dades variables (logs, bases de dades, webs)
│   ├── /var/log/    → Logs del sistema
│   └── /var/www/    → Contingut web (Apache/Nginx)
├── /tmp/   → Fitxers temporals (s'eliminen al reiniciar)
├── /usr/   → Programes i fitxers del sistema
├── /bin/   → Comandes bàsiques
├── /sbin/  → Comandes d'administració
└── /root/  → Directori personal de l'usuari root
```

---

> ✅ **Recordatori**: Si no saps on ets, executa `pwd`. Si no saps què hi ha, executa `ls -la`. I si alguna cosa va malament, `Ctrl + C` és el teu amic!
