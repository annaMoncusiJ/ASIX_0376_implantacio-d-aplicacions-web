# IAW-1.3 · Resol incidències d’Apache i del camí d’accés
**0376 IAW · RA1 · 2 hores**

## Què aconseguiràs?
Provocaràs quatre incidències controlades a la teva VM, observaràs els errors que produeixen i recuperaràs el servei amb proves abans i després.

## Abans de començar
Necessites:
- Apache/PHP funcionals de l’activitat 1.2
- ****Instantania a virtualBox****

Treballaràs individualment amb una única VM Debian dins de l’ordinador. Els serveis s’executen dins d’aquesta mateixa Debian.

Al terminal del teu **ordinador**, connecta’t per ssh al servidor:
```bash
ssh alumne@127.0.0.1 -p 8022
```
> [!IMPORTANT]
> Tingues en compte que `alumne` és el nom d’usuari de l’exemple. Si el teu usuari de Debian té un altre nom, substitueix-lo a la comanda SSH. `sudo` pot demanar la contrasenya d’aquest usuari; no l’escriguis al lliurament.

Per obrir pàgines o fer les proves de client, utilitza el navegador/terminal del teu ordinador. Si és PowerShell i `curl` no és el client cURL real, utilitza `curl.exe`.

> [!IMPORTANT]
> No es tracta d’endevinar un error desconegut: coneixeràs el canvi que has fet. L’objectiu és demostrar **quin símptoma produeix, quines proves el detecten i per què la correcció funciona**. No desfacis immediatament el canvi sense recollir l’evidència.

> [!IMPORTANT]
> Provoca només una incidència cada vegada. Abans de passar a la següent. Si l’estat inicial no coincideix amb el model d’IAW 1.2, atura’t i consulta el professorat mostrant-li la teva pantalla.

## Com has de treballar i lliurar
Desa una còpia d’**aquest mateix document** com `IAW-1.3_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## Preparació inicial · només al començament de la primera classe
Aquesta preparació no afegeix un ítem al lliurament: garanteix que pots recuperar el teu propi entorn.

### A. Comprova que el lloc funciona abans de provocar errors
Dins de Debian:
```bash
sudo apachectl configtest
```
```bash
systemctl is-active apache2
```
```bash
sudo stat -c '%a %n' /var/www/asix/public/index.html
```
Has d’obtenir `Syntax OK`, `active` i el fitxer públic amb mode `644`, com al model d’IAW 1.2. Si no és així, resol aquesta situació abans de començar els casos.


<details>
<summary><b>Opcions per a Linux</b></summary>

Al terminal de l’**ordinador**:
```bash
curl --max-time 5 -i http://127.0.0.1:80/index.html
```
```bash
curl --max-time 5 -i http://127.0.0.1:80/prova.php
```
</details>


<details>
<summary><b>Opcions per a Windows</b></summary>

Al terminal de l’**ordinador**:
```bash
curl.exe --max-time 5 -i http://127.0.0.1:80/index.html
```
```bash
curl.exe --max-time 5 -i http://127.0.0.1:80/prova.php
```
</details>


Has de veure `200` i les marques `ASIX_WEB_OK` / `ASIX_PHP_OK`.

### B. Desa còpies dels dos fitxers de configuració que modificaràs
Dins de Debian, crea una carpeta exclusiva per a aquesta activitat:
```bash
mkdir -m 700 "$HOME/iaw13-copies"
```
> [!IMPORTANT]
> Si la carpeta ja existeix d’un intent anterior, no sobreescriguis les còpies a cegues. Revisa-les i conserva una còpia de l’estat sa actual abans de continuar. El bloc de còpia s’executa només ara, no després de provocar una incidència.

```bash
sudo cp -a /etc/apache2/sites-available/asix.conf "$HOME/iaw13-copies/asix.conf.inicial"
```
```bash
sudo cp -a /etc/apache2/ports.conf "$HOME/iaw13-copies/ports.conf.inicial"
```
```bash
sudo ls -l "$HOME/iaw13-copies"
```
No es modifiquen les dades ni el contingut de les pàgines durant els casos. Si disposes d’una instantània de VirtualBox de l’estat correcte, conserva-la com a recuperació addicional. **Les còpies anteriors ja permeten revertir els canvis de configuració d’aquesta pràctica.**

## 1. A: la pàgina no respon · E1

### Què has de saber abans de començar
Una incidència té un **símptoma** observable i una **causa** que s’ha de comprovar. «El navegador no obre la pàgina» és un símptoma: pot faltar el servei, fallar un reenviament o haver-hi una adreça incorrecta.

Comença per comprovacions de baix risc. Si `systemctl is-active` indica que Apache no està actiu, investiga aquest estat abans de reinstal·lar. `enabled` només indica configuració d’arrencada; no garanteix que el procés estigui funcionant ara. La recuperació s’ha de verificar amb una petició real, no només amb una comanda d’inici que no mostri errors.

### Què has de fer
#### A. Provoca la incidència a la teva Debian

```bash
sudo systemctl stop apache2
```

#### B. Observa abans de corregir
Al terminal de l’**ordinador**, intenta obrir la pàgina:

<details>
<summary><b>Opcions per a Linux</b></summary>

```bash
curl --max-time 5 -i http://127.0.0.1:80/prova.php
```
</details>
<details>
<summary><b>Opcions per a Windows</b></summary>

```bash
curl.exe --max-time 5 -i http://127.0.0.1:80/prova.php
```
</details>
Dins de Debian, comprova l’estat i els sockets en escolta:

```bash
systemctl is-active apache2
```

```bash
sudo ss -ltnp
```

Anota el missatge real del client i l’estat del servei. No inventis un codi HTTP: si no s’ha rebut cap resposta HTTP, indica-ho. Relaciona l’absència del servei amb el símptoma.

#### C. Resol i comprova
Decideix quina acció mínima recupera un servei aturat i executa-la amb systemctl.
Després repeteix `systemctl is-active apache2` dins de Debian i la petició des de l’ordinador. Has de recuperar `active`, `HTTP 200` i `ASIX_PHP_OK` abans de passar al cas B.

### Escriu la teva resposta aquí
1. **Canvi que he fet i símptoma observat:** …
2. **Evidència abans del canvi (comanda i fragment):** …
3. **Relació entre el canvi i la causa, justificada amb les proves:** …
4. **Canvi mínim aplicat:** …
5. **Prova posterior (origen, HTTP i marca):** …

## 2. B: el servei falla després d’un canvi · E2

### Què has de saber abans de començar
Apache interpreta els fitxers de configuració abans d’aplicar-los. Una directiva desconeguda, un bloc mal tancat o un argument incorrecte pot impedir-ne la càrrega. `apachectl configtest` ajuda a localitzar el fitxer i la línia, però no comprova totes les funcions de l’aplicació.

Un servei pot continuar atenent peticions amb la configuració antiga si una recàrrega falla; en canvi, després d’una aturada podria no tornar a iniciar. Cal interpretar l’estat real. La correcció segura consisteix a modificar la causa mínima, tornar a validar i només llavors iniciar o recarregar.

### Què has de fer
#### A. Provoca un error de configuració controlat
Comprova que el cas A està resolt i que tens la còpia inicial. Obre el lloc:

```bash
sudo nano /etc/apache2/sites-available/asix.conf
```

Afegeix **una sola línia**, just abans de `</VirtualHost>`:
```apache
ASIXDirectivaInexistent On
```
No canviïs cap altra línia. Desa el fitxer.

#### B. Observa la configuració abans d’aplicar-la

```bash
sudo apachectl configtest
```

Recull el missatge, el fitxer i la línia que indica. **En un servei real no aplicaries aquesta configuració:** la corregiries immediatament després del test. Només en aquesta VM de laboratori, i un cop recollit el missatge, intenta el reinici per observar-ne l’efecte:

```bash
sudo systemctl restart apache2
```

```bash
systemctl is-active apache2
```

```bash
sudo journalctl -u apache2 -n 15 --no-pager
```

Al terminal de l’**ordinador**, prova la URL habitual:


<details>
<summary><b>Opcions per a Linux</b></summary>

```bash
curl --max-time 5 -i http://127.0.0.1:80/prova.php
```

</details>
<details>
<summary><b>Opcions per a Windows</b></summary>

```bash
curl.exe --max-time 5 -i http://127.0.0.1:80/prova.php
```

</details>



El reinici fallit pot deixar el servei aturat o mantenir el procés anterior segons el mecanisme de la instal·lació. Registra **què ha passat realment**, no assumeixis el resultat. L’error inequívoc que has de demostrar és el de `configtest`.

#### C. Corregeix la causa i verifica
Resol l'error i comprova que el test dona correcte juntament amb totes les altres validacions.


<details>
<summary><b>Pista</b></summary>
per a reiniciar apache has de fer:

```bash
sudo systemctl restart apache2
```

</details>

### Escriu la teva resposta aquí
1. **Canvi que he fet i símptoma observat:** …
2. **Relació entre el canvi i la causa, justificada amb les proves:** …
3. **Canvi mínim aplicat:** …
4. **Proves posteriors:** …

## 3. Cas C: el port no coincideix amb el reenviament · E3

### Què has de saber abans de començar
El camí d’accés té diversos trams: **client de l’ordinador → port amfitrió de VirtualBox → port de Debian → Apache**. Els números dels ports poden ser diferents, però han de correspondre’s amb la configuració de cada tram.

Una prova HTTP feta **dins de Debian** permet comprovar el servei sense passar pel reenviament de VirtualBox. Si aquesta funciona però la de l’ordinador no, investiga els trams que la primera prova ha evitat. `ss -ltnp` mostra sockets TCP en escolta, ports numèrics i processos quan es disposa dels permisos necessaris. No canviïs diversos trams alhora: perdries la traçabilitat de la causa.

### Què has de fer
#### A. Provoca el desajust de ports dins de Debian
Confirma que has recuperat el cas B. Comprova els ports amb:

```bash
sudo ss -ltnp
```

El port 8082 de Debian hauria d’estar lliure.

Obre el fitxer de ports:

```bash
sudo nano /etc/apache2/ports.conf
```

Canvia **només** la línia `Listen 80` per:
```apache
Listen 8082
```
Obre el lloc:

```bash
sudo nano /etc/apache2/sites-available/asix.conf
```

Canvia **només** `<VirtualHost *:80>` per:
```apache
<VirtualHost *:8082>
```
Valida i, si el test és correcte, reinicia:

```bash
sudo apachectl configtest
```

```bash
sudo systemctl restart apache2
```

**No canviïs VirtualBox:** ha de continuar reenviant ordinador:80 → Debian:80.

#### B. Compara els dos camins d’accés
Dins de Debian:

```bash
sudo ss -ltnp
```

```bash
curl --max-time 5 -i http://127.0.0.1:8082/prova.php
```

Al terminal de l’**ordinador**:

```bash
curl --max-time 5 -i http://127.0.0.1:80/prova.php
```

El servei ha de funcionar dins de Debian al 8082, mentre que l’accés habitual de l’ordinador ja no arriba al port on escolta. Explica en quin tram es trenca la cadena.

#### C. Restaura el contracte original
Torna a posar `Listen 80` i `<VirtualHost *:80>` als dos fitxers. Valida amb configtest i, només si és correcte, reinicia Apache. Prova des de l’ordinador a 80: ha de tornar a funcionar.
No «arreglis» el cas creant un reenviament nou: l’objectiu és recuperar l’arquitectura inicial sense tocar SSH ni VirtualBox.

### Escriu la teva resposta aquí
**Canvi que he fet i símptoma a l’ordinador:** …  
**Port observat dins de Debian i prova local:** …  
**Desajust amb el reenviament:** …  
**Quins problemes pot suposar en un entorn real?:** …

## 4. D: el fitxer HTML retorna accés denegat · E4

### Què has de saber abans de començar
Que el servei estigui actiu no implica que pugui llegir tots els fitxers. El procés web necessita **lectura del fitxer** i **travessa dels directoris** que hi condueixen. A més, Apache pot aplicar restriccions pròpies.

Un 403 s’ha de relacionar amb el log, el recurs afectat i els permisos reals. No tots els 403 es corregeixen de la mateixa manera. Canviar permisos recursivament o executar el servei com a root podria ocultar el problema introduint-ne un de seguretat. Restaura només els permisos justificats per al fitxer/directori implicat.

### Què has de fer
#### A. Provoca un error de permisos en un únic fitxer públic
Executa:

```bash
sudo chmod 000 /var/www/asix/public/index.html
```

#### B. Recull la prova del rebuig
Al terminal de l’**ordinador**:

<details>
<summary><b>Opcions per a Linux</b></summary>

```bash
curl --max-time 5 -i http://127.0.0.1:80/index.html
```

</details><details>
<summary><b>Opcions per a Windows</b></summary>

```bash
curl.exe --max-time 5 -i http://127.0.0.1:80/index.html
```

</details>

Dins de Debian:

```bash
ls -l /var/www/asix/public/
```

```bash
sudo tail -n 15 /var/log/apache2/asix-error.log
```

Relaciona els permisos amb l’error HTTP, habitualment 403, i amb el missatge del registre. Si el resultat és diferent, recull-lo i explica’l amb el professorat abans de donar el cas per fet.

#### C. Recupera els permisos mínims previstos
Recorda que el fitxer tenia mode 644. Corregeix l'error i fes totes les comprovacions pertinents

### Escriu la teva resposta aquí
1. **Relació entre el canvi i la causa, justificada amb les proves:** …
2. **Solució de la causa:** …
3. **Proves posteriors:** …

## 5. Comprova l’estat final i explica les diferències · E5

### Què has de saber abans de començar
Una prova final ha de confirmar la funcionalitat i les restriccions: la pàgina correcta ha de funcionar, però un accés que ha de ser denegat ha de continuar denegat. Per això es combinen **proves positives** i **negatives**.

403 i 404 són respostes del nivell HTTP; una connexió rebutjada és un problema anterior a obtenir la resposta HTTP, compatible amb absència d’escolta o un rebuig de xarxa. Cap missatge aïllat identifica sempre tota la causa. El conjunt de proves és el que permet donar la incidència per tancada.

### Què has de fer
Amb els quatre casos tancats, executa des de l’ordinador:

<details>
<summary><b>Opcions per a Linux</b></summary>

```bash
curl -i http://127.0.0.1:80/index.html
```

```bash
curl -i http://127.0.0.1:80/prova.php
```

```bash
curl -i http://127.0.0.1:80/sense-index/
```

```bash
curl -i http://127.0.0.1:80/no-existeix
```

</details>
<details>
<summary><b>Opcions per a Windows</b></summary>

```bash
curl.exe -i http://127.0.0.1:80/index.html
```

```bash
curl.exe -i http://127.0.0.1:80/prova.php
```

```bash
curl.exe -i http://127.0.0.1:80/sense-index/
```

```bash
curl.exe -i http://127.0.0.1:80/no-existeix
```

</details>
Anota els resultats. Explica la diferència entre **403**, **404** i **connexió rebutjada**. Desa la VM en estat funcional; la necessitaràs per connectar amb la base de dades.

### Escriu la teva resposta aquí
| Prova | Resultat esperat | Resultat observat |
|---|---|---|
| HTML | 200 i ASIX_WEB_OK | … |
| PHP | 200 i ASIX_PHP_OK | … |
| Directori sense índex | 403 | … |
| Recurs inexistent | 404 | … |

**403 vol dir:** …  
**404 vol dir:** …  
**Una connexió rebutjada es diferencia perquè:** …

## Recuperació de seguretat si no pots tornar a l’estat inicial
Aquest bloc no substitueix la diagnosi ni la correcció mínima. Utilitza’l només si has perdut el control dels canvis i després d’anotar què ha passat. **Tu executes les comandes a la teva sessió; el professorat et pot orientar mirant les teves sortides o la teva pantalla, sense entrar a la VM.**

1. Comprova que les còpies existeixen i que són les de l’estat sa guardades abans dels casos:
```bash
sudo ls -l "$HOME/iaw13-copies"
```
2. Si les còpies són correctes, restaura només els dos fitxers de laboratori afectats:
```bash
sudo cp -a "$HOME/iaw13-copies/asix.conf.inicial" /etc/apache2/sites-available/asix.conf
```
```bash
sudo cp -a "$HOME/iaw13-copies/ports.conf.inicial" /etc/apache2/ports.conf
```
3. Restaura el permís conegut d’index.html:
```bash
sudo chmod 0644 /var/www/asix/public/index.html
```
4. Valida la sintaxi:
```bash
sudo apachectl configtest
```
**No continuïs si el test falla.** 

```bash
sudo systemctl restart apache2
```
5. Repeteix les quatre proves d’E5 des de l’ordinador. Si has utilitzat aquest bloc, indica-ho al tiquet; una restauració no demostra per si sola que hagis identificat la causa.
 
