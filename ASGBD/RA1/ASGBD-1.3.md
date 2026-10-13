# ASGBD-1.3 · Interpreta logs i recupera MariaDB
**0377 ASGBD · RA1 · 1 hora**

## Què aconseguiràs?
Resoldràs una fallada del SGBD dins de Debian sense esborrar dades ni modificar el servidor web.

## Abans de començar
MariaDB i fila-demo funcionals de l'activitat ASGBD-1.2. Crearàs localment la incidència amb les instruccions d’aquest document.
Apunts a consultar 6.1.1 i 6.2.


## Com has de treballar i lliurar
Desa una còpia d’**aquest mateix document** com `ASGBD-1.3_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …


## 1. El registre no mostra cap error · E1

### Què has de saber abans de començar
Un servidor que arrenca escriu al registre una línia final del tipus **ready for connections**. Aquesta línia diu dos detalls que després necessitaràs: el **socket** (fitxer local que els clients usen per connectar-se sense xarxa) i el **port** TCP. El registre demostra que el procés s’ha iniciat i amb quins paràmetres escolta; **no demostra que cap client pugui connectar-s’hi**. Un registre net no descarta, doncs, un problema de connexió.

### Què has de fer
Aquest fragment és **simulat**. No el presentis com si fos una sortida real de la teva VM:

```text
systemd[1]: Started MariaDB 10.11 database server.
mariadbd[1042]: 2026-10-13  19:12:04 0 [Note] Server socket created on IP: '::'.
mariadbd[1042]: 2026-10-13  19:12:04 0 [Note] /usr/sbin/mariadbd: ready for connections.
             Version: '10.11.6-MariaDB'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306
```

1. Indica què demostra aquest fragment i què no demostra.
2. Treu'n les dues dades concretes que et serviran per al pas següent.
3. Explica per què l’absència d’errors al registre no descarta el problema del client.

### Escriu la teva resposta aquí
**Què demostra el fragment:** …  
**Què no demostra:** …  
**Socket que diu el registre:** …  
**Port que diu el registre:** …  
**Per què un registre sense errors no descarta la fallada del client:** …

## 2. El client falla: quin missatge i quina causa · E2

### Què has de saber abans de començar
El client local es connecta per defecte al **socket Unix**, no a la xarxa. Si el socket no és al camí que el client prova, el missatge és `ERROR 2002 (HY000)` i el número entre parèntesis indica el motiu del sistema: **2 «No such file or directory»** (no hi és) o **13 «Permission denied»** (hi és, però no hi pots accedir). Aquest número és la pista que separa les causes. Una fallada de xarxa dona un altre codi, `2003`, i una contrasenya incorrecta dona `1045`: si la connexió no s’arriba a establir, no pot ser un problema d’autenticació.

### Què has de fer
Els tres blocs següents són **simulats**, tots de la mateixa màquina i del mateix moment:

```text
$ systemctl is-active mariadb
active
```

```text
$ mariadb -e "SELECT etiqueta FROM asix_lab.comprovacio WHERE id=1;"
ERROR 2002 (HY000): Can't connect to local server through socket '/var/run/mysqld/mysqld.sock' (2 "No such file or directory")
```

```ini
# /etc/mysql/mariadb.conf.d/99-asix.cnf
[mysqld]
socket = /var/run/mysqld/mysqld-asix.sock
```

1. Digues quina és la causa confirmada i amb quina evidència del material ho demostres.
2. Descarta **dues** causes alternatives amb una comprovació concreta per a cadascuna, no només dient que no són.
3. Indica quina ordre executaries a la teva VM per confirmar-ho sense modificar res.

### Escriu la teva resposta aquí
**Causa confirmada:** …  
**Evidència que ho demostra:** …  
**Causa descartada 1:** … · **comprovació que la descarta:** …  
**Causa descartada 2:** … · **comprovació que la descarta:** …  
**Ordre de comprovació sense canvis:** …

## 3. No es pot iniciar el servidor · E3

### Què has de saber abans de començar
Per acceptar connexions TCP, un procés necessita vincular-se a una adreça i un port. Si un altre procés ja ocupa la combinació necessària, el servidor pot fallar amb **Address already in use**.

Cal identificar qui escolta abans d’aturar processos: podria ser el servei que realment s’ha de conservar o una altra instància iniciada accidentalment. `ss -ltnp` relaciona sockets i processos quan es disposa dels privilegis necessaris. Canviar un port també afecta els clients i els reenviaments: no és una correcció neutra que s’hagi de fer automàticament.

### Què has de fer
Aquest segon text també és **simulat**:

```text
[ERROR] Can't start server: Bind on TCP/IP port. Got error: 98: Address already in use
[ERROR] Do you already have another server running on port: 3306?
```

Indica què comprovaries abans de canviar el port o aturar processos. No s’ha de solucionar a cegues executant kill ni reinstal·lant MariaDB.

### Escriu la teva resposta aquí
**Causa probable:** …  
**Comanda de comprovació:** …  
**Quina informació necessites abans d’actuar?** …

## 4. Tanca el tiquet · E4

### Què has de saber abans de començar
Un tiquet tècnic permet seguir el raonament d’una altra persona: símptoma observat, evidència, causa confirmada, correcció, verificació posterior i prevenció. La **hipòtesi inicial** pot ser incorrecta; no és un problema si expliques com les proves l’han descartada. La prevenció ha de correspondre a aquesta incidència: conservar una còpia de la configuració abans de modificar-la, comprovar immediatament després d’un canvi o no canviar diverses coses alhora són exemples vàlids. «Anar amb compte» és massa imprecís.

### Què has de fer
Resumeix la incidència simulada de l'apartat 3 en sis línies. Pots referenciar els fragments dels apartats anteriors en lloc de copiar-los de nou. No cal executar res.

### Escriu la teva resposta aquí
1. **Símptoma:** …  
2. **Evidència:** …  
3. **Causa confirmada:** …  
4. **Correcció:** …  
5. **Verificació posterior:** …  
6. **Prevenció:** …

## 5. Dos casos, un sol símptoma · E5

### Què has de saber abans de començar
Els usuaris expliquen el símptoma, no la causa: «no puc connectar» pot venir d’un socket desplaçat, d’un permís canviat o d’un servei que escolta només en una adreça concreta. El que discrimina els casos és **el codi del missatge** i l’**estat del servei**, no la descripció de l’usuari. Per això diagnosticar inclou explicar què **no** és la causa i amb quina comprovació es descarta.

### Què has de fer
Els tres casos són **simulats**. Per a cadascun indica la causa més probable, la comprovació única que el diferencia dels altres dos i la correcció mínima.

**Cas A**

```text
$ systemctl is-active mariadb
active
$ mariadb -e "SELECT 1;"
ERROR 2002 (HY000): Can't connect to local server through socket '/var/run/mysqld/mysqld.sock' (2 "No such file or directory")
```

**Cas B**

```text
$ systemctl is-active mariadb
active
$ mariadb -e "SELECT 1;"
ERROR 2002 (HY000): Can't connect to local server through socket '/var/run/mysqld/mysqld.sock' (13 "Permission denied")
```

**Cas C**

```text
$ systemctl is-active mariadb
active
$ mariadb -h 127.0.0.1 -P 3306 -e "SELECT 1;"
ERROR 2003 (HY000): Can't connect to MySQL server on '127.0.0.1:3306' (111 "Connection refused")
```

### Escriu la teva resposta aquí
**Cas A:**
- **causa:** … 
- **comprovació:** … 
- **correcció:** …  

**Cas B:**
- **causa:** … 
- **comprovació:** … 
- **correcció:** …  

**Cas C:**
- **causa:** … 
- **comprovació:** … 
- **correcció:** …  


## Com es puntua aquesta pràctica
Cadascun dels cinc apartats val **0, 1 o 2 punts**: 2 si és complet i verificat; 1 si hi ha una part correcta però falta una comprovació o justificació; 0 si no hi ha resposta o és incorrecta.

Aquesta pràctica representa el **15% del RA1**. 
