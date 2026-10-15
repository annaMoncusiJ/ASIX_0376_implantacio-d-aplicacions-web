# ASGBD-2.1 · Configura el servei, el motor i el client local
**0377 ASGBD · RA2 · 2 hores**

## Què aconseguiràs?
Configuraràs MariaDB dins de Debian i verificaràs que recupera el servei quan reinicies la VM.

## Abans de començar
MariaDB funcional. /etc/mysql/mariadb.conf.d és un directori. 

## Com has de treballar i lliurar
Desa una còpia d’**aquest mateix document** com `ASGBD-2.1_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## 1. Comprova el cicle de vida del servei · E1

### Què has de saber abans de començar
`systemd` gestiona l’estat i l’arrencada dels serveis a Debian. `start` i `stop` actuen sobre l’execució actual; `restart` atura i inicia; `enable` prepara l’arrencada automàtica quan correspongui.

Un servei pot estar **enabled però inactive**, o **active però disabled**. Per això es comproven les dues dimensions. Reiniciar la VM talla la sessió SSH i interromp també els altres serveis de Debian. Desa el treball abans i reconnecta al mateix port 8022 quan hagi arrencat. La verificació posterior confirma el comportament real, no només la configuració prevista.

### Què has de fer
1. Executa cada acció per separat i anota l’estat que deixa:

```bash
sudo systemctl stop mariadb
```

```bash
systemctl is-active mariadb
```

```bash
sudo systemctl start mariadb
```

```bash
systemctl is-active mariadb
```

```bash
sudo systemctl enable mariadb
```

```bash
systemctl is-enabled mariadb
```

2. **Abans de reiniciar**, escriu què esperes que passi amb `is-active` i amb `is-enabled` després de l’arrencada i per què.
3. Reconnecta i comprova `systemctl is-active mariadb` i `systemctl is-enabled mariadb`. Has de diferenciar **estar actiu ara** i **estar habilitat per arrencar**.

### Escriu la teva resposta aquí
**Predicció abans del reinici i per què:** …  

| Moment | Resultat observat |
|---|---|
| Després de stop | … |
| Després de start | … |
| Després d’enable | … |
| Després de reiniciar la VM | … |

**Diferència entre active i enabled:** …

## 2. Configura InnoDB i verifica una taula nova · E2

### Què has de saber abans de començar
MariaDB és el **SGBD**; InnoDB és un **motor d’emmagatzematge** que pot utilitzar per gestionar taules. Els motors poden diferir en funcions i comportament. InnoDB aporta, entre altres elements, suport transaccional i mecanismes de recuperació.

`default_storage_engine` indica quin motor s’utilitza per defecte en crear taules sense especificar-ne un altre. Canviar-lo no transforma totes les taules ja existents. Per demostrar l’efecte, cal crear una taula nova després d’aplicar la configuració i consultar el seu camp Engine. Es verifica així tant el valor del servidor com l’objecte resultant.

### Què has de fer
1. Consulta motors i valor actual:

```bash
sudo mariadb -e "SHOW ENGINES; SHOW VARIABLES LIKE 'default_storage_engine';"
```

2. Edita `sudo nano /etc/mysql/mariadb.conf.d/60-asix.cnf`. Si el fitxer no existeix, crea’l amb:

```ini
[mysqld]
default_storage_engine=InnoDB
```

Si ja existeix, conserva la resta d’opcions i afegeix/corregeix només aquesta dins del grup `[mysqld]`.
3. Reinicia MariaDB i crea una taula nova. Si el nom ja existeix d’un intent anterior, utilitza un altre nom de prova; no esborris taules desconegudes.

```bash
sudo systemctl restart mariadb
```

```bash
sudo mariadb -e "SHOW VARIABLES LIKE 'default_storage_engine'; CREATE TABLE asix_lab.motor_21 (id INT PRIMARY KEY); SHOW TABLE STATUS FROM asix_lab LIKE 'motor_21'\G"
```

Comprova el camp **Engine** de la taula nova. Canviar el motor per defecte no transforma automàticament totes les taules existents.

### Escriu la teva resposta aquí
**Fitxer/directiva configurats:** …  
**Valor efectiu del motor per defecte:** …  
**Nom de la taula nova i Engine observat:** …  
**Diferència entre MariaDB i InnoDB:** …

## 3. Revisa i protegeix els comptes administradors · E3

### Què has de saber abans de començar
Un compte SQL no és només un nom: en MariaDB també importa **l’origen admès** i el mètode d’autenticació. La identificació, la verificació de la identitat i els permisos són aspectes relacionats però diferents.

Un compte administrador té capacitats que una aplicació normal no necessita. Conservar-lo local i protegit redueix exposició. L’autenticació local per socket pot ser adequada en aquest laboratori; no s’ha de substituir per una contrasenya ni ampliar l’origen a `%` només per comoditat. `%` és un patró ampli, no un sinònim de «la meva VM».

### Què has de fer
1. Amb l’administrador local, consulta només aquests camps; no consultis ni copiïs hashes:

```bash
sudo mariadb -e "SELECT User,Host,plugin FROM mysql.user;"
```

2. Identifica el compte d’administració local, l’origen admès i el mètode d’autenticació.
3. Mantén l’administració local segura. **No creïs root@% ni un administrador remot** per facilitar les connexions.
4. Si la instal·lació ja és segura, documenta per què no cal modificar-la. No hi haueia d'haver cap compte administrador remot inesperat.

### Escriu la teva resposta aquí
| Compte administrador revisat | Origen admès | Mètode d’autenticació | Decisió de protecció |
|---|---|---|---|
| … | … | … | … |

**Per què no has de permetre administració remota des de qualsevol origen?** …

## 4. Configura el client i resol un error de socket · E4

### Què has de saber abans de començar
El fitxer de perfil del client guarda opcions de connexió per evitar repetir-les. **No concedeix privilegis SQL**: el servidor continua comprovant el compte i el seu mètode d’autenticació.

En aquesta tasca, el client administrador s’executa amb sudo i llegeix el perfil de root previst. No s’ha d’assumir que llegirà el `.my.cnf` de l’usuari ordinari. Un socket incorrecte al client pot impedir la connexió encara que el servidor estigui perfectament operatiu. La correcció consisteix a fer que el client apunti al recurs real, no a canviar el servidor per adaptar-lo a una errada del perfil.

### Què has de fer
En aquesta pràctica es configura el **client de l’administrador local**. No es concedeixen privilegis a l’usuari ordinari.
1. Consulta el socket real amb `sudo mariadb -e "SHOW VARIABLES LIKE 'socket';"`.
2. Edita `/root/.my.cnf` amb sudo. Si ja té contingut, desa’n còpia i conserva les opcions existents. Al grup `[client]`, posa el socket real; exemple:

```ini
[client]
socket=/run/mysqld/mysqld.sock
```

3. Aplica `sudo chmod 0600 /root/.my.cnf` i comprova `sudo -H mariadb -e "SELECT 1;"`. No cal escriure-hi una contrasenya si utilitzes l’autenticació local per socket prevista.
4. Provoca **només al client** aquest error:

```bash
sudo -H mariadb --socket=/tmp/no-existeix.sock -e "SELECT 1;"
```

5. Identifica què és incorrecte. Repeteix la comanda sense l’argument erroni. No modifiquis el socket del servidor per adaptar-lo a un error del client.

### Escriu la teva resposta aquí
**Socket real i fragment [client] configurat:** …  
**Permisos del fitxer client:** …  
**Error observat amb l’argument incorrecte:** …  
**Causa i comanda corregida:** …  
**Resultat final de SELECT 1:** …

## 5. Registra els canvis i com revertir-los · E5

### Què has de saber abans de començar
Cada canvi té quatre moments: **configurar, activar, verificar i poder revertir**. Editar un fitxer no garanteix que el servei ja n’utilitzi el valor; pot caldre recarregar, reiniciar o obrir una connexió nova segons l’opció.

La reversió documentada explica com recuperar l’estat anterior si el canvi té efectes no desitjats. No significa esborrar dades ni reinstal·lar tot el sistema. La taula de canvis ha de relacionar el fitxer o element modificat amb la comprovació concreta que demostra l’efecte.

### Què has de fer
Resumeix els canvis de servei, motor i client. Indica com els has activat, com els has verificat i com tornaries a l’estat anterior si fos necessari. No executis la reversió: deixa la configuració correcta funcionant.

### Escriu la teva resposta aquí
**Ordre de reversió i què deixaria el sistema en un estat inconsistent:** …

| Element/fitxer | Canvi | Com s’activa | Comprovat amb | Reversió possible |
|---|---|---|---|---|
| Servei | … | … | … | … |
| Motor / 60-asix.cnf | … | … | … | … |
| Client / /root/.my.cnf | … | … | … | … |

**Estat final del servei:** …

## Com es puntua aquesta pràctica
Cadascun dels cinc apartats val **0, 1 o 2 punts**.

Aquesta pràctica representa el **20% del RA2**. 
