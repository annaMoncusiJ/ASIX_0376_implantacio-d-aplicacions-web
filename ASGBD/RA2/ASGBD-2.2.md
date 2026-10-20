# ASGBD-2.2 · Connecta des de l’ordinador a MariaDB amb reenviament de ports
**0377 ASGBD · RA2 · 2 hores**

## Què aconseguiràs?
Faràs una connexió SQL des de l’ordinador al SGBD de Debian a través del port 3306 de l’ordinador. També prepararàs el compte local que utilitzarà PHP.

## Abans de començar
MariaDB amb `asix_lab` i `asix_lab.comprovacio` —la base i la taula que vas crear a **ASGBD-1.2**, amb la fila `fila-demo`—, client MariaDB o DBeaver instal·lat a l'ordinador, i reenviament VirtualBox 127.0.0.1:3306 → convidat 3306.

Comprova-ho abans de començar, amb una consulta de només lectura:

```bash
sudo mariadb -e "SELECT id, etiqueta FROM asix_lab.comprovacio;"
```

Si la base o la taula no existeixen, torna a crear-les amb l'estructura documentada a ASGBD-1.2 abans de continuar. No en creïs una altra amb un altre nom ni la deixis buida: les activitats següents comproven exactament aquesta fila.

### Com es comprova el camí en aquesta activitat
El camí ordinador → VirtualBox → MariaDB es comprova des de l'**ordinador** amb `Test-NetConnection`, una eina que **ja ve amb Windows** (PowerShell, mòdul NetTCPIP): no cal instal·lar res ni tenir permisos d'administrador. Només comprova que el port respon; no fa cap autenticació SQL.


### Si vols fer també la connexió des de l'ordinador amb entorn gràfic
És **opcional** i no canvia el que has de lliurar com a mínim: el camí ja queda demostrat amb `Test-NetConnection` i la prova SQL amb Debian.Repeteix la connexió des de l'ordinador i pots veure-la des del punt de vista del client. Els dos que funcionen **sense instal·lar i sense permisos d'administrador**:

| Eina | Com s'obté | Avantatge | Avís |
|---|---|---|---|
| **HeidiSQL portable** | Paquet «Windows x64 Portable» de heidisql.com | No necessita Java ni descarregar cap controlador: porta la biblioteca de client dins la carpeta, així que funciona també sense Internet | Perquè desi la configuració dins la carpeta i no al registre, crea-hi un fitxer buit anomenat `portable.lock` |
| **DBeaver Community portable** | «Windows ZIP archive» de dbeaver.io | Més general i conegut | Porta el Java dins el ZIP, però **descarrega el controlador MariaDB la primera vegada** |

Passos amb qualsevol dels dos (són clients, no servidors: **no ocupen el port 3306 de l'ordinador** ni cal registre ni servei):

1. Descomprimeix el ZIP en una carpeta on puguis escriure i obre l'executable (`heidisql.exe` o `dbeaver.exe`). No l'executis com a administrador.
2. Crea una connexió nova amb aquests valors: tipus **MariaDB/MySQL (TCP/IP)**, amfitrió `127.0.0.1`, port `3306`, usuari `clientlab` i el secret pel canal privat. **No activis cap túnel SSH**: el reenviament ja el fa VirtualBox.
3. Prova de connectar. Si et dona `ERROR 1045 … 'clientlab'@'10.0.2.2'`, **has guanyat informació, no has fallat**: aquell missatge demostra que el camí des de l'ordinador arriba al servidor i quin origen hi veu. És el tercer origen dels tres possibles (`localhost`, la IP de la interfície i el del NAT).

El client gràfic **no substitueix** les proves de Debian: l'actualització denegada (autorització) i la contrasenya errònia (autenticació) es continuen fent on s'ha indicat a cada apartat.

## Com has de treballar i lliurar
Desa una còpia d’**aquest mateix document** com `ASGBD-2.2_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## 1. Prepara el camí ordinador → VirtualBox → MariaDB · E1

### Què has de saber abans de començar
Perquè un client pugui arribar al SGBD des de fora de Debian s'han de complir tres condicions: MariaDB escolta en una interfície accessible, VirtualBox reenvia el port i la política de xarxa no bloqueja el camí. **Ara utilitzem 3306 a l'ordinador i 3306 a Debian**, però continuen sent dos extrems diferents.

`bind-address=0.0.0.0` amplia l'escolta a les interfícies IPv4 de Debian; no equival a restringir clients. En aquest laboratori es combina amb VM només NAT, reenviament limitat a `127.0.0.1` de l'ordinador i comptes restringits.

L'origen que veu el servidor **depèn del camí**, no del nom que hagis escrit al client. Connectar-se a `127.0.0.1` o a la IP de la interfície de Debian (`10.0.2.15`) són dos camins diferents, i el servidor no registra el mateix origen. `skip-name-resolve` evita basar la identificació dels hosts en resolució de noms, però no elimina l'autenticació. Cal observar l'origen real, no donar per fet que és sempre `127.0.0.1`.

### Què has de fer
1. A VirtualBox, comprova el reenviament **TCP, IP amfitrió 127.0.0.1, port amfitrió 3306, port convidat 3306**.
2. Dins de Debian, edita `/etc/mysql/mariadb.conf.d/60-asix.cnf`, conservant el motor. Afegeix/corregeix:

```ini
[mysqld]
bind-address=0.0.0.0
port=3306
skip-name-resolve
```

`[mysqld]` ja pot existir: no cal duplicar el grup. L'escolta a les interfícies de Debian permet rebre el trànsit reenviat; **la VM ha de tenir només NAT, no un adaptador pont exposat a la xarxa**. El professorat haurà verificat el tallafoc de Debian. La regla de VirtualBox es limita a l'ordinador amfitrió amb 127.0.0.1.
3. Reinicia el servei i comprova:

```bash
sudo systemctl restart mariadb
```

```bash
systemctl is-active mariadb
```

```bash
sudo ss -ltnp
```

4. **Abans de comprovar res**, escriu què esperes obtenir i quin origen creus que veurà MariaDB. Després comprova el camí des de l'**ordinador**, amb l'eina que ja té Windows:

```powershell
Test-NetConnection -ComputerName 127.0.0.1 -Port 3306
Test-NetConnection -ComputerName 127.0.0.1 -Port 3307
```

`TcpTestSucceeded : True` al primer vol dir que el camí ordinador → VirtualBox → Debian arriba al port de MariaDB. `False` al segon vol dir que aquell port no té reenviament. Això demostra que el port és accessible, però **no** que puguis iniciar sessió: no s'ha fet cap autenticació SQL.
5. Dins de Debian, fes la prova SQL amb un usuari que no existeix, connectant a la IP de la interfície:

```bash
ip -br address
```

```bash
mariadb --protocol=tcp -h 10.0.2.15 -P 3306 -u sondeig -p --connect-timeout=5
```

Substitueix `10.0.2.15` per la IP de la teva interfície. Espera un rebuig SQL i fixa't en **l'origen que diu el missatge**: és el que veu el servidor.

### Escriu la teva resposta aquí
**Reenviament VirtualBox configurat:** …  
**Test-NetConnection des de l'ordinador (3306 i 3307):** …  
**Predicció abans de comprovar (què esperaves i per què):** …  

## 2. Prepara un compte per a PHP local i un per al client de xarxa · E2

### Què has de saber abans de començar
El servidor diferencia comptes segons **usuari i origen**. El PHP local arriba per `127.0.0.1`; una connexió a la IP de la interfície de Debian és un altre camí. El mateix nom d'usuari en dos orígens són dos comptes diferents, i per això se'n prepara un per a cada ús.

`CREATE USER` defineix el compte i la seva autenticació. `GRANT SELECT ON base.taula` concedeix lectura sobre un objecte concret, no administració del SGBD. Un nom semblant en dues instàncies o dos orígens no implica els mateixos permisos.

El mínim privilegi limita l'impacte d'un ús indegut del compte. Les contrasenyes van per un canal privat i no s'han de copiar al document. Desactivar l'historial de la sessió SQL de preparació redueix una possible còpia local del secret, però no substitueix la resta de controls.

### Què has de fer
Dins de Debian, entra amb `sudo env MYSQL_HISTFILE=/dev/null mariadb`. Substitueix els secrets i ORIGEN_IFACE pels valors del teu laboratori (sense cometes dins dels secrets per a aquesta plantilla):

```sql
CREATE USER 'webapp'@'127.0.0.1' IDENTIFIED BY 'SECRET_WEB_LOCAL';
GRANT SELECT ON asix_lab.comprovacio TO 'webapp'@'127.0.0.1';
CREATE USER 'clientlab'@'ORIGEN_IFACE' IDENTIFIED BY 'SECRET_CLIENT';
GRANT SELECT ON asix_lab.comprovacio TO 'clientlab'@'ORIGEN_IFACE';
```

Justifica `ORIGEN_IFACE` amb el **missatge real** que has rebut, no amb el valor de l'enunciat. Si els dos no coincideixen, explica què vols dir i quina evidència utilitzes. Mantén root/admin local; no creïs root@% ni concedeixis privilegis globals.
- `webapp`: PHP dins de Debian → TCP 127.0.0.1:3306.
- `clientlab`: client que connecta a la IP de la interfície de Debian. Si tens client a l'ordinador, l'origen del NAT és un altre compte diferent.

Comprova els permisos amb SHOW GRANTS però no copiïs informació d'autenticació, hashes ni secrets al document.

### Escriu la teva resposta aquí
| Compte | Origen autoritzat real | Permís i objecte |
|---|---|---|
| webapp | … | … |
| clientlab | … | … |

**Administració local protegida:** …

## 3. Consulta les dades pel camí de xarxa · E3

### Què has de saber abans de començar
El client SQL envia ordres i mostra resultats. Aquí connectes a la **IP de la interfície de Debian**, no a `127.0.0.1`: el tràfic surt cap a la xarxa de la VM i el servidor ho registra com un origen diferent del local. Si et connectessis a `127.0.0.1`, series dins del mateix host i no demostraries res d'això.

`USER()` descriu la identitat de connexió que veu el servidor; `CURRENT_USER()` identifica el compte emprat per a l'autenticació/autorització. Comparar-los ajuda a entendre quin origen i compte s'han seleccionat. DBeaver és una interfície client alternativa, amb la IP i el port d'aquest mateix endpoint.

### Què has de fer
Dins de Debian, connecta't a la IP de la interfície amb `clientlab`:

```bash
mariadb --protocol=tcp -h 10.0.2.15 -P 3306 -u clientlab -p -e "SELECT USER(), CURRENT_USER(); SELECT etiqueta FROM asix_lab.comprovacio WHERE id=1;"
```

Has d'obtenir fila-demo. `USER()` descriu la identitat de connexió que veu el servidor i `CURRENT_USER()` el compte que ha seleccionat; anota-les sense contrasenya.

I ara la prova que dóna sentit a tot plegat: el **mateix** usuari, des del camí local, ha de fallar:

```bash
mariadb --protocol=tcp -h 127.0.0.1 -P 3306 -u clientlab -p -e "SELECT 1;"
```

No és un error de contrasenya: el secret és el mateix. El compte `clientlab` només existeix per a l'origen de la interfície. Si tens client SQL a l'ordinador, repeteix la consulta autoritzada des d'allà: amb el reenviament de VirtualBox veuràs l'origen del NAT, i per tant caldria un compte per a aquell origen.

### Escriu la teva resposta aquí
**Endpoint utilitzat (IP i port):** …  
**Resultat del mateix usuari des de 127.0.0.1:** …  
**USER() i CURRENT_USER():** …  
**Resultat de la consulta:** …

## 4. Comprova autenticació, permisos i limitació a l'amfitrió · E4

### Què has de saber abans de començar
**Autenticació** respon «qui ets?»; **autorització** respon «què pots fer?». Una contrasenya errònia ha de fallar abans d'executar la consulta. Una UPDATE denegada amb credencials correctes demostra que l'usuari autenticat no té aquell permís.

L'adreça d'escolta del reenviament a l'ordinador és una tercera dimensió: limitar-la a 127.0.0.1 vol dir que només aquest equip pot fer servir el reenviament. Ho comproves des de l'ordinador amb `Test-NetConnection`: des d'un altre equip de la xarxa local no respondria. Inspeccionar la configuració no equival a haver fet la prova. Tampoc significa que SQL estigui xifrat: el laboratori treballa amb dades fictícies i accés controlat.

### Què has de fer
1. Connectant a la IP de la interfície, intenta la mateixa connexió amb una contrasenya incorrecta. Registra el rebuig d'autenticació.
2. Connecta amb clientlab correcte (mateix endpoint) i intenta aquesta actualització innòcua (manté el mateix valor si erròniament estigués autoritzada):

```sql
UPDATE asix_lab.comprovacio SET etiqueta=etiqueta WHERE id=1;
```

Ha de ser **denegada** perquè clientlab només té SELECT. Si funciona, revisa els permisos i repeteix fins a obtenir el rebuig.

3. **Demostra-ho des de l'ordinador amb el client gràfic portàtil** (HeidiSQL o DBeaver, els del bloc «Si vols fer també la connexió des de l'ordinador»): obre o crea la connexió a `127.0.0.1:3306` amb `clientlab` i intenta connectar.
4. **Classifica les tres proves amb la mateixa aplicació gràfica**, sense tornar a la línia d'ordres, i escriu a la taula què demostra cadascuna:
   - **Contrasenya incorrecta** (la de l'apartat 1, repetida al gràfic): el diàleg o panell d'error et mostra el missatge del servidor → **autenticació**: et rebutja abans d'executar res.
   - **UPDATE denegada** (la de l'apartat 2, executada des de l'editor de consultes del gràfic): el missatge del servidor → **autorització**: el compte ha entrat, però no té aquell permís.
   - **Connexió des de l'ordinador** (l'apartat 3): el missatge diu **quin origen veu el servidor** (`'clientlab'@'10.0.2.2'` amb el reenviament) → **abast d'escolta**: només s'hi arriba des d'aquest equip perquè la regla està limitada a `127.0.0.1`.
   Cap de les tres demostra que les dades viatgin xifrades: al laboratori es treballa sobre dades fictícies en un NAT controlat, i per parlar de xifratge caldria configurar TLS.

### Escriu la teva resposta aquí
| Comprovació | Resultat real | Què demostra? |
|---|---|---|
| Contrasenya errònia | … | … |
| UPDATE amb clientlab | … | … |
| Fila de la regla al gestor de VirtualBox (IP amfitrió) | … | … |
| Origen que veu el servidor des de l'ordinador | … | … |

**En cas de que et dongui error l'acces des de l'ordinador, que has de fer per a resoldre-ho?**: …

**Consulta final de fila-demo:** …

## 5. Diagnostica un port equivocat sense canviar SSH · E5

### Què has de saber abans de començar
El port de destinació forma part de l'endpoint. Si el client truca a un port sense reenviament, pot no arribar mai al SGBD; aleshores canviar la contrasenya no resol el problema. En canvi, un error d'autenticació SQL és evidència que s'ha arribat al servidor, i una fallada de connexió demostra que el camí no arriba.

Quan els números són iguals a tots dos costats, com el 3306 d'aquesta configuració, encara cal indicar **on s'executa la comanda**. El registre final ha de deixar clars ordinador, xarxa de la VM, Debian i compte utilitzat. El port SSH 8022 és una via d'administració independent i no s'ha de modificar per arreglar SQL.

### Què has de fer
Des de l'ordinador, prova un port sense reenviament (3307) amb `Test-NetConnection`: espera `TcpTestSucceeded : False`. Des de Debian, connecta't a aquell port i compara el missatge amb el de contrasenya incorrecta: un és de connexió (2003) i l'altre d'autenticació (1045). Torna al port 3306 i verifica la consulta.

### Escriu la teva resposta aquí
**Port de prova, símptoma i causa:** …  
**Port corregit i resultat:** …  



## Com es puntua aquesta pràctica
Cadascun dels cinc apartats val **0, 1 o 2 punts**
Aquesta pràctica representa el **20% del RA2**. 