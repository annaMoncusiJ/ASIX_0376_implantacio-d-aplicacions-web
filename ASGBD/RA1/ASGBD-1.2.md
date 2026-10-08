# ASGBD-1.2 · Instal·la MariaDB a la mateixa Debian
**0377 ASGBD · RA1 · 2 hores**

## Què aconseguiràs?
Instal·laràs el SGBD a la mateixa VM on ja tens Apache. Comprovaràs el funcionament local sense obrir encara l’accés des de l’ordinador.

## Abans de començar
VM Debian amb l'activitat ASGBD-1.1 feta. No creïs una segona VM ni aturis Apache.
Consulta opcional dels apunts: 4.1, instal·lació de 4.3.2 adaptada i inici 6.1.

Treballaràs individualment amb una única VM Debian dins de l’ordinador. Els serveis tradicionals i Docker, quan correspongui, s’executen dins d’aquesta mateixa Debian.

- Al terminal del teu **ordinador**, connecta’t així:
```bash
ssh alumne@127.0.0.1 -p 8022
```
> [!IMPORTANT]
> Tingues en compte que `alumne` és el nom d’usuari de l’exemple. Si el teu usuari de Debian té un altre nom, substitueix-lo a la comanda SSH. `sudo` pot demanar la contrasenya d’aquest usuari; no l’escriguis al lliurament.

## Com has de treballar i lliurar
Desa una còpia d’**aquest mateix document** com `ASGBD-1.2_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## 1. Comprova requisits i instal·la els paquets · E1

### Què has de saber abans de començar
Instal·lar un SGBD implica posar a disposició executables, biblioteques, eines client i fitxers de servei/configuració. En Debian, el gestor de paquets resol dependències i registra versions. El **paquet servidor** i el **paquet client** tenen funcions diferents.

### Què has de fer
1. Instal·la servidor i client:

```bash
sudo apt update
```

```bash
sudo apt install mariadb-server mariadb-client
```

```bash
dpkg-query -W mariadb-server mariadb-client
```

2. Si la instal·lació falla, conserva l’error i consulta’l abans de repetir o canviar repositoris. No continuïs afirmant que el servei està instal·lat si no ho has verificat.

### Escriu la teva resposta aquí
**SO i recursos revisats:** …
```text
Enganxa les dues línies de versions de paquets instal·lats.
```
**Per a què serveix el paquet servidor i per a què el client?** …

## 2. Comprova servei i accés local · E2

### Què has de saber abans de començar
Que el paquet estigui instal·lat no garanteix que el servidor hagi iniciat ni que accepti consultes. Per verificar-lo comprovem **servei, escolta i accés SQL**. El directori `datadir` conté les dades del servidor: no és una carpeta temporal que es pugui esborrar per «netejar» una instal·lació.

Una connexió local pot usar un **socket Unix**, mecanisme de comunicació entre processos de la mateixa màquina. Amb autenticació per socket, el SGBD pot utilitzar la identitat de l’usuari del sistema per autoritzar l’accés. Això és diferent d’una connexió TCP amb usuari/contrasenya i no implica accés remot lliure.

### Què has de fer
Executa a Debian:

```bash
sudo systemctl enable --now mariadb
```

```bash
systemctl is-active mariadb
```

```bash
sudo ss -ltnp
```

```bash
sudo mariadb -e "SELECT VERSION() AS versio, @@datadir AS directori_dades; SELECT 1;"
```

Has d’obtenir servei `active`, una versió, un directori de dades i el valor 1. En aquest entorn, `sudo mariadb` pot entrar amb autenticació local per socket: això **no** significa que els equips remots puguin entrar sense contrasenya.

### Escriu la teva resposta aquí
**Estat del servei:** …  
**Adreça/port d’escolta observats:** …
```text
Versió, datadir i resultat de SELECT 1.
```
**Com t’has autenticat localment?** …

## 3. Crea una dada i consulta-la després de reconnectar · E3

### Què has de saber abans de començar
SQL permet donar instruccions al SGBD. En l’exemple, `CREATE DATABASE` i `CREATE TABLE` defineixen estructures; `INSERT` introdueix una fila; `SELECT` consulta dades. La clau primària `id` identifica cada fila de manera única.

El guió usa dades fictícies i una fila coneguda per poder verificar resultats. Tornar a connectar abans de consultar comprova que no només has escrit la instrucció o vist un text al terminal: la dada existeix al servidor. Aquesta prova no substitueix una estratègia de còpies de seguretat ni una prova de recuperació de desastres, que no es desenvolupen aquí.

### Què has de fer
1. Entra amb `sudo mariadb` i executa aquest SQL. Només crea dades fictícies de laboratori:

```sql
-- Només dades fictícies del laboratori. No executar en un servidor de producció.
CREATE DATABASE IF NOT EXISTS asix_lab CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE TABLE IF NOT EXISTS asix_lab.comprovacio (
 id INT PRIMARY KEY,
 etiqueta VARCHAR(40) NOT NULL
) ENGINE=InnoDB;
INSERT INTO asix_lab.comprovacio (id, etiqueta) VALUES (1, 'fila-demo')
ON DUPLICATE KEY UPDATE etiqueta=VALUES(etiqueta);
SELECT id, etiqueta FROM asix_lab.comprovacio;
```

2. Surt del client amb `exit`.
3. Obre una connexió nova i consulta:

```bash
sudo mariadb -e "SELECT id, etiqueta FROM asix_lab.comprovacio WHERE id=1;"
```

Has de veure **1** i **fila-demo**. El fitxer SQL escrit però no executat no demostra que la dada existeixi.

### Escriu la teva resposta aquí
**Resultat de la consulta després de reconnectar:**
```text
…
```
**Què demostra aquesta comprovació?** …

## 4. Resol un error senzill d’instal·lació · E4

### Què has de saber abans de començar
APT i MariaDB són sistemes diferents. Si APT no localitza un paquet, el problema és anterior a l’execució del SGBD: pot ser el nom, l’índex de paquets o el repositori. En el microcas es treballa una errada de nom controlada.

La diagnosi ha de ser proporcional a l’evidència. Comprovar el nom correcte és més adequat que canviar repositoris o reinstal·lar Debian sense justificació. Després de corregir, verifica que el servei que ja funcionava no s’ha vist afectat. Documentar una incidència petita també permet demostrar una metodologia correcta.

### Què has de fer
Aquesta és una prova intencionada. Executa `sudo apt install mariadb-servre` i llegeix el missatge. No has de modificar repositoris ni eliminar el servidor que ja funciona.
1. Identifica què no troba apt.
2. Contrasta el nom amb `apt-cache policy mariadb-server`.
3. Escriu la comanda corregida. Si el paquet correcte ja està instal·lat, no cal reinstal·lar-lo.
4. Torna a comprovar que MariaDB continua actiu.

### Escriu la teva resposta aquí
**Missatge d’error observat:** …  
**Causa:** …  
**Comanda amb el nom corregit:** …  
**Estat final del servei:** …

## 5. Localitza configuració i logs i documenta · E5

### Què has de saber abans de començar
La configuració del servidor pot estar repartida entre un fitxer principal i directoris inclosos. Cal saber **quins fitxers es llegeixen** i en quin ordre; editar un fitxer que no es carrega no canviarà el servei.

Els registres poden anar al journal del sistema, a fitxers o a altres destinacions segons la instal·lació. `journalctl -u mariadb` filtra missatges associats al servei; `log_error` ajuda a identificar la configuració del registre del SGBD. Un valor buit no justifica inventar una ruta. Tampoc són equivalents el registre d’errors, el de consultes i els registres interns de recuperació.

### Què has de fer
1. Inspecciona `/etc/mysql/my.cnf` i els directoris que inclou. Anota quin fitxer conté la configuració del servidor a la teva VM.
2. Executa:

```bash
sudo mariadb -e "SHOW VARIABLES LIKE 'log_error';"
```

```bash
sudo journalctl -u mariadb -n 15 --no-pager
```

3. Si `log_error` conté una ruta, comprova que el fitxer existeix i llegeix-ne un fragment. Si és buit o el registre és al journal, anota el mecanisme observat: no inventis una ruta.
4. Escriu un procediment de màxim **10 passos** des de la comprovació inicial fins a la consulta final per tal d'instal·lar i configurar mariaDB. No copiïs tot l’historial del terminal.

### Escriu la teva resposta aquí
**Fitxer/directori de configuració:** …  
**Ubicació dels logs:** …
```text
Fragment breu del registre que has consultat.
```
**Procediment (màxim 10 passos):**
1. …
2. …

## Com es puntua aquesta pràctica
Cadascun dels cinc apartats val **0, 1 o 2 punts**: 2 si és complet i verificat; 1 si hi ha una part correcta però falta una comprovació o justificació; 0 si no hi ha resposta o és incorrecta. Total: **10 punts**.

Aquesta pràctica representa el **25% del RA1**. 
