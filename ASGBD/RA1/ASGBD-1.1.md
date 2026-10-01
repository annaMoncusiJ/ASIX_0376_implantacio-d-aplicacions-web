# ASGBD-1.1 · Tria un SGBD i comprova els recursos de Debian
**0377 ASGBD · RA1 · 1 hora**

## Què aconseguiràs?
Compararàs SGBD i justificaràs una selecció per a un servei amb clients remots. Després comprovaràs els recursos de la teva VM.

## Abans de començar
VM Debian ja creada amb SSH instal·lat i consula els apunts 1.1, 1.2.1, 1.3, 3.1–3.2 i 4.1.

Treballaràs individualment amb una única VM Debian dins de l’ordinador.

1. Al terminal del teu **ordinador**, connecta’t així:
```bash
ssh alumne@127.0.0.1 -p 8022
```
> [!IMPORTANT]
> Tingues en compte que `alumne` és el nom d’usuari de l’exemple. Si el teu usuari de Debian té un altre nom, substitueix-lo a la comanda SSH. `sudo` pot demanar la contrasenya d’aquest usuari; no l’escriguis al lliurament.


> [!IMPORTANT]
> Per obrir pàgines o fer les proves de client, utilitza el navegador/terminal del teu ordinador. Si és PowerShell i `curl` no és el client cURL real, utilitza `curl.exe`.

- Afegeix les seguents redireccions de ports a la configuració de VirtualBox

| Nom | Protocol | IP amfitrió | Port amfitrió | IP convidat | Port convidat |
   |-----|----------|-------------|---------------|-------------|----------------|
   | `MariaDB` | `TCP` | `127.0.0.1` | `3306` |  | `3306` |

## Com has de treballar i lliurar
Desa una còpia d’**aquest mateix document** com `ASGBD-1.1_COGNOMS_NOM.md`. Completa únicament on s’especifica i lliura **només aquest fitxer**. Enganxa fragments curts, no captures redundants ni logs complets. Els fitxers de servei es queden a Debian. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## Cas que has de resoldre
Una petita botiga prepara una aplicació web per consultar clients, productes i comandes. Una comanda pertany a un client i pot incloure diversos productes. Cal guardar aquestes dades de manera coherent i permetre que diferents persones les consultin o actualitzin.

Has de recomanar un **SGBD per al servei de dades**. Per a aquesta introducció disposem dels requisits següents:

| Codi | Requisit del cas | Què vol dir aquí? |
|---|---|---|
| R1 | Dades relacionals i coherents | Representar clients, productes i comandes en taules relacionades. Poder impedir, per exemple, una comanda vinculada a un client inexistent. |
| R2 | Servei de BD independent amb connexions de xarxa | El SGBD ha de tenir un procés servidor propi. Un client instal·lat fora de Debian ha de poder connectar-s’hi directament amb les eines del SGBD, sense construir una API intermediària. |
| R3 | Autenticació i permisos | Identificar els comptes i poder donar permisos diferents; per exemple, un compte només de consulta i un compte d’administració. |
| R4 | Operacions simultànies | Permetre diversos clients i coordinar les seves operacions. Es preveuen fins a 20 usuaris simultanis, però avui no es demana demostrar temps de resposta ni dimensionar un servei de producció. |
| R5 | Instal·lació oberta a Debian | Utilitzar una opció oberta disponible a Debian, amb servidor i eines client sense compra obligatòria d’una llicència comercial per al laboratori. Això no significa que administrar-la tingui cost zero. |

**Què sabem i què no sabem:** coneixem el tipus de dades i d’accés. No coneixem el volum real de dades, les consultes ni el ritme d’operacions; per tant, no podem afirmar quin producte serà més ràpid ni garantir capacitat de producció amb una RAM determinada.

**Com ho representarem:** treballaràs individualment amb la Debian que ja has creat. Més endavant, Apache/PHP i MariaDB seran serveis diferents dins d’aquesta mateixa VM, i també provaràs un client de BD al teu ordinador com a client remot.

## 1. Relaciona situacions amb funcions i components

### Què has de saber abans de començar
Un **SGBD** és el programari que permet definir, emmagatzemar, consultar i administrar bases de dades. La base de dades és el conjunt de dades i estructures; el SGBD és el sistema que les gestiona. Una aplicació no ha de resoldre per si sola tots els problemes d’accés simultani, permisos o consistència.

Les funcions principals són: **gestió/consulta**, per recuperar i modificar dades; **integritat**, per mantenir regles de coherència; **seguretat**, per controlar qui pot fer què; **concurrència**, per coordinar operacions simultànies; i **recuperació**, per tornar a un estat consistent davant de fallades mitjançant els mecanismes previstos.

Internament, el processador de consultes interpreta i prepara SQL; el gestor d’emmagatzematge organitza l’accés a les dades físiques; el diccionari o catàleg descriu els objectes. Són funcions/components relacionats, però no són sinònims.

### Què has de fer
Assigna una funció a cada situació. Utilitza una vegada cadascuna: **gestió/consulta de dades, integritat, seguretat, concurrència i recuperació**.
- F1. Es rebutja una comanda perquè fa referència a un client que no existeix.
- F2. El compte del web pot llegir dades, però no modificar-les.
- F3. Es coordinen dues operacions simultànies que volen actualitzar la mateixa dada.
- F4. Després d’una fallada, el SGBD utilitza els seus mecanismes per recuperar un estat consistent.
- F5. Es recuperen les files que compleixen la condició d’una consulta.

Relaciona també C1–C3 amb **processador de consultes, gestor d’emmagatzematge i diccionari/catàleg**:
- C1. Analitza una petició SQL i en prepara l’execució.
- C2. Gestiona l’accés a blocs/pàgines de dades del disc.
- C3. Guarda informació sobre taules, columnes i altres objectes.

### Escriu la teva resposta aquí
| Cas | Funció o component |
|---|---|
| F1 | … |
| F2 | … |
| F3 | … |
| F4 | … |
| F5 | … |
| C1 | … |
| C2 | … |
| C3 | … |

## 2. Compara les tres opcions

### Què has de saber abans de començar
Cal comparar dues dimensions diferents. El **model de dades** descriu com es representen i relacionen les dades: en el model relacional hi ha taules, files, columnes i restriccions. L’**arquitectura de desplegament** descriu com s’executa el servei i com hi accedeixen els clients.

En una arquitectura servidor/client, un procés servidor rep connexions de clients. En un sistema integrat, la biblioteca de bases de dades forma part de l’aplicació que l’utilitza; això no el fa menys relacional. Un servidor i un client poden executar-se en màquines diferents o compartir màquina. Al nostre laboratori, el procés web i el procés MariaDB comparteixen Debian, però també provarem un client a l’ordinador.

### Què has de fer
Llegeix aquestes fitxes i completa la taula sense copiar tot el text:
- **MariaDB Community Server:** SGBD relacional amb procés servidor i eines client, disponible a Debian. Admet connexions de xarxa autenticades i permisos per compte. Amb InnoDB, permet transaccions, claus foranes i operacions concurrents. És programari lliure; el laboratori no requereix comprar una llicència comercial. El suport comercial és opcional.
- **PostgreSQL:** SGBD objecte-relacional amb arquitectura servidor/client i eines disponibles a Debian. Admet connexions remotes autenticades, permisos per compte, claus foranes i transaccions concurrents. Té una llicència oberta permissiva, sense compra obligatòria per al laboratori; també hi ha serveis de suport.
- **SQLite:** biblioteca relacional integrada a una aplicació, disponible a Debian i habitualment amb un fitxer de dades. El seu codi és de domini públic. Té transaccions i admet usos concurrents amb limitacions; no és correcte dir que només pot servir a un usuari. No incorpora un procés servidor de xarxa propi ni gestiona comptes de connexió de xarxa com els dos servidors anteriors. Per exposar operacions a la xarxa caldria afegir una altra capa, que R2 exclou en aquest cas.

Distingeix **model de dades** i **arquitectura**. Un SGBD pot ser relacional sense tenir un servidor de xarxa propi.

### Escriu la teva resposta aquí

Completa la taula amb les teves paraules:

| SGBD | Model de dades | Servidor/client o integrat? | Pot fer de servidor BD remot independent en aquest cas? Per què? |
|---|---|---|---|
| MariaDB | … | … | … |
| PostgreSQL | … | … | … |
| SQLite | … | … | … |

## 3. Escull i justifica

### Què has de saber abans de començar
Seleccionar un producte significa **relacionar característiques amb requisits**, no triar el que tingui més funcions. Cal considerar accés per xarxa, concurrència, tipus de dades, administració, compatibilitat amb l’aplicació, suport i manteniment.

Una opció descartada per a aquest cas pot ser adequada per a una aplicació local o integrada. La justificació ha d’explicar aquesta adequació concreta. No podem afirmar quin producte és més ràpid sense definir càrrega, dades i proves comparables. En aquesta activitat s’avalua una decisió tècnica argumentada; la plataforma comuna del laboratori és una decisió organitzativa diferent.

### Què has de fer
Torna a la taula **R1–R5 del cas de la botiga**. Utilitza només aquella taula i les fitxes d’E2; no cal cercar informació externa.
1. Recomana una de les tres opcions per al cas complet, no per un sol requisit aïllat.
2. Tria **dos requisits diferents**, escriu-ne el codi i relaciona cadascun amb una característica concreta de l’opció escollida.
3. Identifica una alternativa que **no compleixi algun requisit** i indica quin. Si dues opcions són adequades, no inventis un inconvenient per distingir-les: pots reconèixer que totes dues serien vàlides.

Per redactar cada motiu pots utilitzar aquest esquema: «El requisit R… demana …; l’opció … ho permet perquè …». No n’hi ha prou amb dir «és el millor», «és conegut» o «és sempre més ràpid». No has de fer un estudi de rendiment.

### Escriu la teva resposta aquí
**SGBD escollit:** …  
**Motiu 1 — codi del requisit + necessitat + característica que el satisfà:** …  
**Motiu 2 — un altre requisit + necessitat + característica que el satisfà:** …  
**Alternativa no adequada — opció + requisit que no compleix + explicació:** …

## 4. Identifica els recursos de la teva VM

### Què has de saber abans de començar
Abans d’instal·lar un servei, cal conèixer l’entorn on funcionarà:
sistema operatiu, processadors disponibles, memòria i espai de disc.

En aquesta activitat utilitzaràs la VM Debian que ja vas preparar
a IAW 1.1. No establim un perfil nou ni et demanem que en modifiquis
els recursos.

Cal distingir:
- **vCPU:** processadors virtuals que Debian pot utilitzar.
- **RAM assignada:** memòria configurada a VirtualBox.
- **RAM detectada:** memòria que Debian reconeix com a utilitzable;
  pot ser una mica inferior a l’assignada.
- **Espai lliure:** espai disponible al sistema de fitxers.
  No és el mateix que la capacitat total del disc virtual.

Aquestes dades permeten descriure el punt de partida, però no
demostren que la VM pugui atendre els 20 usuaris del cas amb un
rendiment determinat. Per valorar-ho caldria conèixer les dades,
les consultes i la càrrega de treball, i fer proves.

### Què has de fer
1. Executa aquestes comandes dins de la teva VM Debian.

```bash
cat /etc/os-release
```

```bash
lscpu
```

```bash
free -m
```

```bash
df -h /
```

2. Completa la taula amb els valors reals i les unitats.

### Escriu la teva resposta aquí
| Recurs | Valor observat i unitat | Comanda utilitzada |
|---|---|---|
| Sistema operatiu i versió | … | … |
| vCPU | … | … |
| RAM total detectada | … | … |
| Espai lliure a `/` | … | … |

**Per què l’espai lliure a `/` és rellevant abans d’instal·lar MariaDB? Indica dos elements que necessitaran espai de disc:** …
