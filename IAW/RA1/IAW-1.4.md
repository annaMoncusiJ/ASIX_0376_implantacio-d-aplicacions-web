# IAW-1.4 · Integra PHP i MariaDB dins de Debian
**0376 IAW · RA1 · 3 hores**

## Què aconseguiràs?
La pàgina PHP consultarà MariaDB per TCP local dins de Debian. El navegador de l’ordinador només es connectarà al web a través del reenviament.

## Abans de començar
Apache/PHP i MariaDB instal·lats a la mateixa Debian. Base asix_lab, taula comprovacio i compte webapp@127.0.0.1 preparats a ASGBD 2.2, amb secret de laboratori.


## Com has de treballar i lliurar
Desa una còpia d’**aquest mateix document** com `IAW-1.4_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## 1. Comprova la implantació del SGBD que compartirà Debian · E1

### Què has de saber abans de començar
MariaDB ja està instal·lat a la teva Debian des d’ASGBD i, en aquesta activitat, el compartirà el web. Comprovar-ho vol dir **tres coses diferents**: el **paquet** instal·lat i la seva versió, el **servei** actiu i escoltant, i una **consulta real** que retorna la fila coneguda. Que un paquet estigui instal·lat no demostra que el servei accepti connexions, i que el servei estigui actiu no demostra que les dades del laboratori segueixin allà.

**No cal reinstal·lar res.** El MariaDB que verifiques és el **mateix** que continuaran utilitzant PHP i ASGBD, no un SGBD temporal en una segona VM. Si en algun moment el gestor de paquets et pregunta què fer amb un fitxer de configuració modificat, conserva la configuració de laboratori existent i confirma-ho amb el professorat.

### Què has de fer
MariaDB ja s’ha treballat a ASGBD, però en aquest document has d’aportar les teves pròpies evidències d’implantació i verificació d’IAW.
1. Comprova el paquet instal·lat i la seva versió:

```bash
dpkg -l mariadb-server | tail -n 1
```

```bash
sudo systemctl start mariadb
```

```bash
systemctl is-active mariadb
```

2. Fes una consulta real des de la sessió de Debian:

```bash
sudo mariadb -e "SELECT VERSION(); SELECT 1; SELECT etiqueta FROM asix_lab.comprovacio WHERE id=1;"
```

3. Demostra que **les dades i la configuració** són les del laboratori, no només que el servei està actiu: la consulta de la fila coneguda i una opció de configuració vigent ho mostren. És el mateix servidor que utilitzaran PHP i ASGBD.
4. **Deixa MariaDB actiu.** No l’aturis ni el deshabilitis: la integració amb PHP depèn d’aquest servei.

### Escriu la teva resposta aquí
**Paquet/versió comprovats:** …  
**Configuració conservada:** …  
**Servei, SELECT 1 i fila-demo:** …  
**Per què aquí no has d’aturar ni deshabilitar MariaDB?** …

## 2. Configura PHP per connectar per TCP local · E2

### Què has de saber abans de començar
**PDO** és una interfície de PHP per accedir a bases de dades. Necessita un controlador adequat, aquí `pdo_mysql`, que és compatible amb MariaDB en aquest ús. El **DSN** descriu la destinació de la connexió: protocol/controlador, host, port i base.

PHP i MariaDB viuen dins de Debian; per això PHP utilitza **127.0.0.1:3306**. Aquesta connexió no necessita travessar el NAT de VirtualBox. Utilitzar `127.0.0.1` força l’adreça TCP indicada; alguns clients interpreten `localhost` de manera especial i poden utilitzar un socket Unix.

Separar les credencials en un fitxer fora del DocumentRoot evita incloure-les al codi públic. `0640 root:www-data` permet al procés web llegir la configuració sense fer-la llegible per a tots els usuaris. Cal protegir també còpies temporals i lliuraments.

### Què has de fer
1. Dins de Debian, comprova el connector:

```bash
sudo apt install php-mysql
```

```bash
php -m | grep -E 'PDO|pdo_mysql'
```

```bash
sudo systemctl reload apache2
```

```bash
sudo install -d -o root -g www-data -m 0750 /etc/asix
```

2. Crea amb `sudo nano /etc/asix/db.ini` aquesta configuració. Substitueix només el secret pel que has preparat/rebut per a webapp local:

```ini
host = "127.0.0.1"
port = "3306"
database = "asix_lab"
user = "webapp"
password = "SECRET_LOCAL"
```

```bash
sudo chown root:www-data /etc/asix/db.ini
```

```bash
sudo chmod 0640 /etc/asix/db.ini
```

3. Aquí PHP és dins de Debian: **127.0.0.1:3306** és correcte. Tot i que el número 3306 coincideix amb el del client de l’ordinador, aquí PHP parla amb MariaDB dins de Debian: no passa per VirtualBox ni pel reenviament SSH. El fitxer queda fora del DocumentRoot.

### Escriu la teva resposta aquí
**Connector detectat:** …  
**Host/port/base/usuari, sense secret:** …  
**Recorregut complet: navegador de l’ordinador → reenviament → Apache/PHP → MariaDB (indica ports):** …

## 3. Publica i executa la pàgina de consulta · E3

### Què has de saber abans de començar
La cadena completa és **navegador → HTTP → Apache/PHP → consulta SQL → MariaDB → resultat → resposta HTTP**. El navegador no necessita credencials de la base de dades: és el codi del servidor qui fa la consulta.

El codi facilitat llegeix la configuració, construeix la connexió PDO i executa una consulta fixa. `fetchColumn()` recupera el valor d’una columna de la fila retornada. La marca de resposta només s’escriu després de rebre el valor esperat. Així es diferencia una consulta real d’un text escrit manualment en una pàgina HTML. No es demana desenvolupar formularis ni processar entrades d’usuari en aquest bloc.

### Què has de fer
1. Crea `/var/www/asix/public/db.php` i copia aquest codi. No hi has d’afegir credencials:

```php
<?php
header('Content-Type: text/plain; charset=utf-8');
try {
    $c = @parse_ini_file('/etc/asix/db.ini', false, INI_SCANNER_RAW);
    if (!$c) { throw new RuntimeException('configuracio absent'); }
    foreach (['host','port','database','user','password'] as $k) {
        if (!isset($c[$k]) || $c[$k] === '') { throw new RuntimeException('configuracio incompleta'); }
    }
    $dsn = 'mysql:host='.$c['host'].';port='.(int)$c['port'].';dbname='.$c['database'].';charset=utf8mb4';
    $pdo = new PDO($dsn, $c['user'], $c['password'], [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION, PDO::ATTR_TIMEOUT => 3]);
    $result = $pdo->query('SELECT etiqueta FROM comprovacio WHERE id=1')->fetchColumn();
    if ($result !== 'fila-demo') { throw new RuntimeException('dada inesperada'); }
    echo "ASIX_DB_OK: " . $result . "
";
} catch (Throwable $e) {
    http_response_code(503);
    echo "Servei de dades temporalment no disponible.
";
    // No registrar missatge complet, DSN ni contrasenya. Només classe i codi.
    error_log('ASIX_DB_ERROR tipus='.get_class($e).' codi='.$e->getCode());
}
```

2. Aplica permisos 0644 al fitxer. Obre el projecte amb l’entorn integrat utilitzat anteriorment.
3. Des de l’**ordinador**, executa `curl -i http://127.0.0.1:8080/db.php`. Has d’obtenir **HTTP 200** i **ASIX_DB_OK: fila-demo**.
4. Si no funciona, comprova el connector, la configuració i la connexió SQL TCP local amb l’ajuda del professorat abans de continuar. Escriure aquesta marca directament en un HTML no és una prova vàlida: ha de provenir de la consulta.

### Escriu la teva resposta aquí
**Client d’origen:** …
```text
Enganxa la línia HTTP i el resultat retornat per db.php.
```
**Quina consulta ha executat el codi i d’on surt fila-demo?** …

## 4. Compara dos errors de connexió · E4

### Què has de saber abans de començar
Una connexió SQL pot fallar abans de contactar amb el servidor, durant l’autenticació o quan s’intenta utilitzar un recurs sense permís. Un **port incorrecte** i una **contrasenya incorrecta** no tenen la mateixa causa i no s’han de corregir amb el mateix canvi.

L’aplicació retorna un 503 genèric per no exposar detalls interns al navegador. El registre tècnic permet investigar sense publicar secrets. Un codi SQLSTATE o de client ajuda a classificar l’error, però s’ha de contrastar amb el canvi controlat i les proves. Restaura cada error abans de provocar el següent per evitar una diagnosi amb dues causes simultànies.

### Què has de fer
Fes les proves **d’una en una**, modificant només `/etc/asix/db.ini` a Debian:
1. Desa una còpia privada per poder restaurar-la. Canvia temporalment el port a un port de prova sense servei indicat pel professorat. Demana db.php i consulta el log d’Apache. Restaura el port i comprova 200.
2. Canvia temporalment la contrasenya per una d’incorrecta. Repeteix la petició i la lectura del log. Restaura el secret correcte i comprova 200.
3. Prova un **tercer cas**: base de dades inexistent. Classifica l’error i explica en què es distingeix dels dos anteriors. Restaura i comprova 200.
4. Els tres casos es fan **d’un en un**, modificant només una línia cada vegada i restaurant abans de continuar.
3. El navegador ha de rebre un **503 genèric**, no el secret ni un error intern detallat. El codi facilitat registra la classe/codi de l’error, no la contrasenya.

```bash
sudo tail -n 15 /var/log/apache2/asix-error.log
```

### Escriu la teva resposta aquí
| Cas | Símptoma HTTP | Fragment/codi del log sense secrets | Causa | Prova després de restaurar |
|---|---|---|---|---|
| Port incorrecte | … | … | … | … |
| Contrasenya incorrecta | … | … | … | … |
| Base de dades inexistent | … | … | … | … |

## 5. Verifica la protecció i tanca el procediment · E5

### Què has de saber abans de començar
El **mínim privilegi** consisteix a concedir només les accions necessàries. Una pàgina que llegeix una taula no necessita administrar usuaris, crear bases ni modificar dades. Si el compte de l’aplicació es veiés compromès, limitar-ne els permisos reduiria l’abast de les accions possibles.

La protecció té diferents capes: ubicació del secret fora del directori públic, permisos del fitxer i privilegis del compte SQL. Un 404 en demanar `/db.ini` no demostra tot sol que el secret sigui segur; també cal comprovar on es troba realment i qui pot llegir-lo. No copiïs informació d’autenticació o hashes que alguna versió pugui mostrar amb SHOW GRANTS.

### Què has de fer
1. Executa `sudo stat -c '%U:%G %a %n' /etc/asix/db.ini`. Ha de mostrar root:www-data i 640.
2. Comprova que `/etc/asix/db.ini` és fora de DocumentRoot. Una petició a `/db.ini` no ha de lliurar el contingut real del fitxer.
3. **Què passaria si `/etc/asix/db.ini` fos dins de DocumentRoot?** Com ho comprovaries des de l’ordinador? …
4. Des de Debian, verifica els permisos del compte local TCP amb contrasenya interactiva:

```bash
mariadb --protocol=tcp -h 127.0.0.1 -P 3306 -u webapp -p -e "SHOW GRANTS FOR CURRENT_USER;"
```

Substitueix el port si és diferent. El compte ha de tenir només els permisos de lectura previstos sobre la taula, no privilegis administratius. Si SHOW GRANTS mostra informació d’autenticació, **no copiïs aquesta línia**: recull només els privilegis i l’origen sense hashes ni secrets.
4. Escriu els passos essencials per repetir la integració. Conserva Debian i MariaDB operatives.

### Escriu la teva resposta aquí
**Ubicació/permisos del fitxer:** …  
**Resultat de demanar /db.ini:** …  
**Risc si el fitxer fos servible pel web i com ho has comprovat:** …  
**Permisos del compte (sense informació d’autenticació):** …  
**Procediment de connexió i verificació (màxim 8 passos):**
1. …
2. …

## Com es puntua aquesta pràctica
Cadascun dels cinc apartats val **0, 1 o 2 punts**

Aquesta pràctica representa el **15% del RA1**. 
