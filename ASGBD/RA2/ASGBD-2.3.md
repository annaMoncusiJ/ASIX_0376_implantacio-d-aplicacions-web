# ASGBD-2.3 · Configura defaults i límits dins de Debian
**0377 ASGBD · RA2 · 1 hora**

## Què aconseguiràs?
Aplicaràs una configuració persistent i observaràs la caducitat d’una sessió de prova, sense dependre d’una segona VM.

## Abans de començar
MariaDB, compte webapp local i Python3/PyMySQL preparats dins de Debian.

## Com has de treballar i lliurar
Desa una còpia d’**aquest mateix document** com `ASGBD-2.3_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## Perfil d’explotació que has d’aplicar
| Variable | Valor docent |
|---|---|
| character_set_server | utf8mb4 |
| collation_server | utf8mb4_unicode_ci |
| max_connections | 30 |
| connect_timeout | 10 segons |
| wait_timeout | 120 segons |

Són valors de laboratori, no una recomanació universal de producció. Conserva la configuració de xarxa i motor de les activitats anteriors.

## 1. Consulta els valors inicials · E1

### Què has de saber abans de començar
Els **valors per defecte** s’utilitzen quan una operació no especifica una alternativa. El conjunt de caràcters defineix com es representen els caràcters; la col·lació defineix regles de comparació i ordenació. Són conceptes relacionats però diferents.

`max_connections` limita connexions simultànies, no el nombre total de consultes. `connect_timeout` i `wait_timeout` es mesuren en segons, però afecten moments diferents. Anotar els valors inicials permet justificar què canvia i detectar si el perfil ja estava parcialment aplicat. No s’ha de confondre el valor de la fitxa amb el que realment està en execució.

### Què has de fer
Abans de modificar res, executa a Debian:

```bash
sudo mariadb -e "SHOW GLOBAL VARIABLES WHERE Variable_name IN ('character_set_server','collation_server','max_connections','connect_timeout','wait_timeout');"
```

Omple la primera taula amb els valors reals i les unitats dels límits. No posis directament els valors objectiu si encara no són els actius.

### Escriu la teva resposta aquí
| Variable | Valor inicial real | Valor objectiu | Unitat, si escau |
|---|---|---|---|
| character_set_server | … | utf8mb4 | — |
| collation_server | … | utf8mb4_unicode_ci | — |
| max_connections | … | 30 | … |
| connect_timeout | … | 10 | … |
| wait_timeout | … | 120 | … |

**Risc de deixar cada límit massa alt o massa baix:** …

## 2. Aplica els valors persistentment i verifica · E2

### Què has de saber abans de començar
Una variable pot tenir **valor global**, **valor de sessió** i una configuració desada al fitxer. Un canvi global en execució no sempre afecta sessions ja obertes i no garanteix persistència després d’un reinici.

Per demostrar configuració persistent, modifiquem el fitxer inclòs correcte, reiniciem controladament el servei i consultem des d’una connexió nova. Si hi ha opcions duplicades o un altre fitxer les sobreescriu, el valor efectiu pot no coincidir amb el que acabes d’editar. Conserva les opcions de xarxa i motor: canviar límits no ha de trencar allò que ja funciona.

### Què has de fer
1. Edita `/etc/mysql/mariadb.conf.d/60-asix.cnf`. **Conserva el motor, bind-address i port que ja hi tens.** Afegeix o corregeix aquestes cinc opcions dins de `[mysqld]`, sense duplicar-les:

```ini
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
max_connections=30
connect_timeout=10
wait_timeout=120
```

2. Reinicia **el servei**, no cal reiniciar tota la VM:

```bash
sudo systemctl restart mariadb
```

```bash
systemctl is-active mariadb
```

3. Obre una connexió nova i repeteix la consulta d’E1. Els valors han de coincidir amb l’objectiu. Un `SET GLOBAL` sense configuració persistent no seria suficient per a aquesta tasca.
4. Si el valor efectiu **no** coincidís amb el que has escrit al fitxer, explica com localitzaries el fitxer o l’opció que el sobreescriu, sense esborrar res.

### Escriu la teva resposta aquí
**Fragment de les cinc opcions al fitxer:**
```ini
…
```
**Valors globals observats després de reiniciar el servei:**
```text
…
```

## 3. Comprova els defaults en una base nova · E3

### Què has de saber abans de començar
Els valors per defecte del servidor influeixen en **objectes nous**, però no converteixen automàticament bases i taules creades abans. Per això la prova utilitza un nom de base nou i consulta la seva definició.

`SHOW CREATE DATABASE` permet veure característiques de la base concreta, no només la intenció de configuració del servidor. Fer `CREATE DATABASE IF NOT EXISTS` sobre una base antiga no demostra que n’hagis aplicat els nous defaults: pot no haver creat ni modificat res. Canviar objectes existents és una operació diferenciada que aquí no es demana.

### Què has de fer
A Debian, executa:

```bash
sudo mariadb -e "CREATE DATABASE asix_defaults; SHOW CREATE DATABASE asix_defaults;"
```

Si la base ja existeix, tria un nom nou, com `asix_defaults_2`, i usa’l en les dues sentències. No eliminis bases desconegudes. No utilitzis `IF NOT EXISTS` per afirmar que una base antiga ha adoptat defaults nous.

Comprova els valors de charset i collation de la base que acabes de crear. Explica què passa amb les bases ja existents.

### Escriu la teva resposta aquí
**Nom de la base nova:** …
```text
Resultat de la comanda anterior.
```
**El canvi de defaults converteix automàticament totes les bases i taules anteriors? Explica-ho:** …

## 4. Observa l’expiració d’una sessió · E4

### Què has de saber abans de començar
`wait_timeout` controla la inactivitat d’una connexió no interactiva. No és el temps màxim de qualsevol consulta. Els clients interactius poden inicialitzar el temps d’espera a partir d’`interactive_timeout`; per evitar aquesta confusió, la prova utilitza una connexió PyMySQL concreta.

El guió canvia només **SESSION wait_timeout=2**, espera quatre segons i intenta reutilitzar la mateixa connexió sense reconnexió automàtica. El global continua en 120. L’error posterior s’ha de relacionar amb el temps d’espera i l’absència d’altres fallades. No cal abaixar el global de tot el servidor per observar el fenomen.

### Què has de fer
La prova següent canvia **només el wait_timeout d’una connexió de prova a 2 segons**. No canviïs el valor global 120.
1. Dins de Debian, crea `prova-timeout.py` amb aquest codi. És una eina de comprovació; no has de desenvolupar un programa nou:

```python
#!/usr/bin/env python3
"""Microtest de sessió: no canvia globals, comptes ni dades. Sense reconnexió automàtica."""
import argparse,getpass,time,sys
try:
    import pymysql
except ImportError:
    sys.exit('Falta python3-pymysql. El docent ha de preparar la dependència.')
p=argparse.ArgumentParser(description=__doc__)
p.add_argument('--host',required=True);p.add_argument('--port',type=int,default=3306)
p.add_argument('--user',default='webapp');a=p.parse_args()
c=None
try:
    c=pymysql.connect(host=a.host,port=a.port,user=a.user,password=getpass.getpass('Contrasenya de laboratori: '),connect_timeout=5,read_timeout=5,write_timeout=5,autocommit=True)
    with c.cursor() as cur:
        cur.execute('SELECT @@GLOBAL.wait_timeout, @@SESSION.wait_timeout, CONNECTION_ID()')
        print('INICIAL (global, sessió, id):',cur.fetchone())
        cur.execute('SET SESSION wait_timeout=2')
        cur.execute('SELECT @@GLOBAL.wait_timeout, @@SESSION.wait_timeout')
        print('PROVA (global, sessió):',cur.fetchone())
    time.sleep(4)
    try:
        with c.cursor() as cur: cur.execute('SELECT 1')
    except pymysql.err.OperationalError as e:
        if e.args[0] in (2006,2013):
            print('OBSERVAT: connexió perduda després de la inactivitat; codi',e.args[0])
            print('Correlaciona-ho amb wait_timeout=2 i absència d’altres fallades. No s’ha reconnectat.')
        else:
            print('Error no esperat, codi',e.args[0]);sys.exit(1)
    else:
        print('REVISAR: la connexió continua activa; investiga la configuració efectiva.');sys.exit(1)
except pymysql.MySQLError as e:
    print('No s’ha pogut completar la prova. Codi:',e.args[0]);sys.exit(1)
finally:
    if c is not None: c.close()
```

2. Executa des de Debian autoritzada, amb contrasenya interactiva:

```bash
python3 prova-timeout.py --host 127.0.0.1 --port 3306 --user webapp
```

3. Observa global/sessió abans del canvi, sessió=2 durant la prova i l’error en intentar reutilitzar la **mateixa connexió** després de 4 segons. Si la prova no caduca, anota el resultat i investiga a que pot ser degut. No inventis la sortida.

El test no reconnecta automàticament. Una nova connexió normal no ha de quedar amb el 2 temporal. Els clients interactius poden inicialitzar el temps d’espera amb `interactive_timeout`; aquí es prova una connexió no interactiva.

### Escriu la teva resposta aquí
**Sortida breu del microtest:**
```text
…
```
**Valor global que es manté i valor temporal de sessió:** …  
**Per què el test no reconnecta i què demostra això:** …  
**Per què no hem canviat wait_timeout globalment a 2?** …

## 5. Explica els canvis amb precisió · E5

### Què has de saber abans de començar
Ajustar paràmetres és una decisió d’explotació, no una recepta universal. Un límit de connexions massa baix pot rebutjar usuaris legítims; un temps d’inactivitat massa curt pot tallar sessions que l’aplicació encara espera reutilitzar.

Abans de canviar valors en producció caldria conèixer càrrega i comportament de l’aplicació. En aquest laboratori s’avalua que sàpigues configurar, verificar i explicar un perfil donat. El reinici del servei interromp connexions i s’ha de planificar; el fet que les dades siguin fictícies no elimina la necessitat d’entendre aquest efecte.

### Què has de fer
Completa quatre frases breus i indica una precaució abans de reiniciar un servidor amb usuaris connectats. No necessites una memòria de rendiment.

### Escriu la teva resposta aquí
**max_connections limita** …  
**connect_timeout no és el mateix que wait_timeout perquè** …  
**Un canvi només amb SET GLOBAL pot no persistir perquè** …  
**El reinici del servei s’ha de planificar perquè** …  
**Fitxer modificat i comprovació final de servei actiu:** …

## Com es puntua aquesta pràctica
Cadascun dels cinc apartats val **0, 1 o 2 punts**

Aquesta pràctica representa el **10% del RA2**. 
