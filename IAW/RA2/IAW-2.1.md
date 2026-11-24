# IAW-2.1 · Compara i tria un CMS
**0376 IAW · RA2 · 1 hora**

## Què aconseguiràs?

Identificaràs què és un gestor de continguts (CMS) i per a què serveix, a partir de la informació que publiquen les seves webs oficials. Compararàs tres CMS amb els mateixos criteris i els classificaràs segons el tipus de lloc web que permeten muntar.

Acabaràs triant el CMS amb què faràs la resta de la pràctica i justificaràs la tria amb els requisits de les activitats següents. Abans de tancar la decisió, comprovaràs que la teva VM neta amb A01 compleix els requisits tècnics del CMS.

No instal·laràs res en aquesta activitat: investigues, compares i decideixes. La instal·lació arriba a IAW-2.2.

## Abans de començar

Necessites una **VM Debian neta** amb SSH funcional i Apache, PHP i MariaDB operatius.
Si reutilitzes una VM on ja has fet altres pràctiques, neteja'n la configuració o torna a importar una còpia nova abans de començar: una configuració heretada et pot donar errors.

Treballaràs individualment amb una única VM Debian dins de l'ordinador. Els serveis tradicionals s'executen dins d'aquesta mateixa Debian; Docker s'hi instal·larà a IAW-2.2.

1. Al terminal del teu **ordinador**, connecta't així:
```bash
ssh alumne@127.0.0.1 -p 8022
```
> [!IMPORTANT]
> Tingues en compte que `alumne` és el nom d'usuari de l'exemple. Si el teu usuari de Debian té un altre nom, substitueix-lo a la comanda SSH. `sudo` pot demanar la contrasenya d'aquest usuari; no l'escriguis al lliurament.

2. Quan aparegui la sessió del teu usuari a Debian, executa-hi les comandes de Linux i `sudo`. **Totes les comandes de l'enunciat són dins de Debian, excepte les marcades explícitament «ordinador».**
3. Per consultar les webs dels CMS i obrir pàgines, utilitza el navegador del teu ordinador.

| Servei | Des del teu ordinador | Destinació dins de Debian |
|---|---|---|
| SSH | 127.0.0.1:8022 | 22 |
| Apache tradicional | http://127.0.0.1:80 | 80 |
| MariaDB tradicional | 127.0.0.1:3306 | 3306 |


## Com has de treballar i lliurar

Desa una còpia d'**aquest mateix document** com `IAW-2.1_COGNOMS_NOM.md`. Completa únicament on s'especifica i lliura **només aquest fitxer**. Enganxa fragments curts i enllaços, no captures redundants ni pàgines senceres copiades. No lliuris secrets, contrasenyes ni hashes; substitueix-los per `[OCULT]`.

**Nom i cognoms:** …

## 1. Investiga tres CMS a les webs oficials · E1

### Què has de saber abans de començar

Un **gestor de continguts** (CMS, *Content Management System*) és una aplicació web que permet crear, publicar i administrar contingut sense escriure cada pàgina a mà. La informació no viu dins d'un fitxer HTML: viu en una **base de dades** i el CMS la combina amb una **plantilla** per generar la pàgina que rep el navegador.

Un CMS es compon de:
- **Nucli:** el codi base del producte. Normalment no es modifica.
- **Tema o plantilla:** defineix l'aspecte visual.
- **Extensions, mòduls o connectors:** afegeixen funcionalitats (formularis, botiga, còpies de seguretat…).

La **llicència** indica què pots fer amb el programari. Les més habituals en CMS són:
- **GPL / AGPL:** programari lliure; pots usar-lo, estudiar-lo, modificar-lo i redistribuir-lo, i les versions derivades mantenen la mateixa llicència.
- **MIT / Apache 2.0:** també lliures i permissives, però permeten distribuir versions derivades amb una altra llicència.
- **Llicència privativa:** ús limitat per contracte; habitual en plataformes allotjades (SaaS).

Una **versió** és un número concret del producte i canvia sovint; el que has de fer servir és el que la web oficial mostra com a estable **el dia que consultes**. La **comunitat** és qui dona suport: fòrums, documentació, extensions de tercers i empreses que ofereixen servei.

> [!TIP]
> No copiïs el text de màrqueting de la portada. Busca les pàgines «About», «Requirements», «Download» o «License» i redacta tu la informació en una frase per camp.

### Què has de fer

1. Tria **tres CMS**: **WordPress** (és la proposta de CMS principal per a la resta de la pràctica) i **dos més de tipus diferent**. La proposta és **Joomla** i **Drupal**. Pots canviar algun d'aquests dos per un altre CMS, però has de justificar-ho en una línia.
2. Consulta la **web oficial** de cadascun (no blogs ni resums de tercers) i anota la informació de la taula.
3. Indica la data de consulta: la versió i les xifres de comunitat canvien.

```text
Webs oficials de referència (proposta):
- WordPress → https://wordpress.org
- Joomla    → https://www.joomla.org
- Drupal    → https://www.drupal.org
```

> [!IMPORTANT]
> WordPress és la proposta de CMS principal per a la resta de l'activitat, però **el professorat confirmarà la decisió** abans de IAW-2.2. Si proposes un altre CMS principal, argumenta-ho a l'E3.

### Escriu la teva resposta aquí

| Camp | WordPress | CMS 2: … | CMS 3: … |
|---|---|---|---|
| Web oficial consultada | … | … | … |
| Què és (una frase) | … | … | … |
| Ús principal | … | … | … |
| Llicència | … | … | … |
| Versió estable observada | … | … | … |
| Comunitat i suport (fòrum, documentació, extensions) | … | … | … |
| Data de consulta | … | … | … |

**Si has substituït Joomla o Drupal per un altre CMS, justifica-ho:** …

## 2. Classifica'ls i compara'ls amb criteris fixos · E2

### Què has de saber abans de començar

Els CMS es poden classificar pel **tipus de lloc web** que permeten gestionar. Un mateix CMS pot servir per a més d'un tipus, però normalment en destaca un:

| Tipus de lloc | Què hi predomina | Exemples de CMS o plataformes |
|---|---|---|
| Blog | Entrades ordenades per data, autoria, categories i etiquetes | WordPress, Ghost |
| Portal d'informació | Notícies, seccions, molta actualització i diversos redactors | Drupal, Joomla |
| Web corporativa | Pàgines estàtiques, serveis, formularis de contacte, multilingüe | WordPress, Joomla, Drupal |
| Comerç electrònic | Catàleg, carret, pagaments, gestió d'estocs i comandes | PrestaShop, WooCommerce, Magento |
| Comunitat o fòrum | Usuaris registrats, perfils, missatges i moderació | Discourse, phpBB |
| Formació (LMS) | Cursos, inscripcions, activitats i qualificacions | Moodle |

Comparar no és fer una llista d'avantatges: és aplicar **els mateixos criteris** a tots els productes i deixar clar en què es diferencia cada un. Els criteris d'aquesta pràctica són fixos:

- **Tipus de llicència.**
- **Ús principal** (per a quin tipus de lloc està pensat).
- **Facilitat d'ús** per a una persona sense experiència.
- **Ecosistema d'extensions** (quantitat i qualitat de temes i complements).
- **Comunitat i suport** (documentació, fòrums, servei professional).
- **Flexibilitat i personalització** (quant es pot adaptar sense tocar el nucli).

### Què has de fer

1. Assigna a cada CMS **un tipus de lloc principal** i, si vols, un de secundari. Usa els tipus de la taula anterior o proposa'n un altre i explica'l.
2. Completa la taula comparativa. Cada cel·la ha de ser una valoració breu i **comparable**: si dius «molta» a un, digues «mitjana» o «poca» als altres, no ho deixis buit.
3. Afegeix una valoració final pròpia en una frase per CMS.

### Escriu la teva resposta aquí

| CMS | Tipus de lloc principal | Tipus secundari |
|---|---|---|
| WordPress | … | … |
| CMS 2: … | … | … |
| CMS 3: … | … | … |

| Criteri | WordPress | CMS 2: … | CMS 3: … |
|---|---|---|---|
| Tipus de llicència | … | … | … |
| Ús principal | … | … | … |
| Facilitat d'ús | … | … | … |
| Ecosistema d'extensions | … | … | … |
| Comunitat i suport | … | … | … |
| Flexibilitat i personalització | … | … | … |

**Valoració final (una frase per CMS):**
- WordPress: …
- CMS 2: …
- CMS 3: …

## 3. Tria el CMS i comprova els requisits a Debian · E3

### Què has de saber abans de començar

La tria no es fa pel CMS «que agrada més», sinó per la seva adequació a uns **requisits concrets**. A la resta de la pràctica hauràs de:

- Instal·lar-lo sobre Apache, PHP i MariaDB i, a més, desplegar-lo amb Docker (IAW-2.2).
- Configurar paràmetres generals, publicar contingut i activar mesures de seguretat (IAW-2.3).
- Gestionar usuaris i rols, mòduls, plantilles i sindicació de continguts (IAW-2.4).
- Fer còpies de seguretat, importar i exportar continguts i actualitzar-lo (IAW-2.5).
- Analitzar la seva estructura de fitxers i crear un **tema fill** (IAW-2.7 i IAW-2.8).

Per tant, el CMS triat ha de tenir: usuaris amb rols, extensions i plantilles, exportació/importació de continguts, mecanisme d'actualització i suport de temes fills o equivalents.

Els **requisits tècnics** del CMS són els mínims que necessita per funcionar: versió de PHP, extensions de PHP, servidor de bases de dades i versió, i mòduls d'Apache com `mod_rewrite` per a les URL amigables.

A l'activitat A01 vas deixar un LAMP operatiu. Comprova'l ara amb dades reals, no de memòria:

- `apache2 -v` mostra la versió d'Apache.
- `php --version` mostra la versió de PHP; `php -m` llista les extensions carregades.
- `mariadb --version` (o `mysql --version`) mostra el client de bases de dades.
- `systemctl is-active apache2 mariadb` indica si els serveis estan en marxa.
- `apache2ctl -M` llista els mòduls d'Apache carregats.

> [!IMPORTANT]
> Una versió del laboratori pot ser anterior a la que demana el CMS. No actualitzis res en aquesta activitat: anota la diferència i indica a l'E3 com la resoldries (per exemple, instal·lant una extensió de PHP a IAW-2.2). El professorat indicarà si cal ajustar l'entorn.

### Què has de fer

1. Escriu **tres requisits** del CMS que proposes com a principal, extrets de la seva documentació oficial («Requirements»).
2. Executa a **Debian** aquestes comprovacions. Si alguna et falla per permisos, torna-la a executar amb `sudo`; no instal·lis ni actualitzis res:

```bash
apache2 -v
```

```bash
php --version
```

```bash
php -m
```

```bash
mariadb --version
```

```bash
systemctl is-active apache2 mariadb
```

```bash
apache2ctl -M
```

3. Compara cada requisit amb el que has obtingut i marca si es compleix. Si alguna cosa no es compleix, proposa com resoldre-ho sense sortir de l'entorn de la pràctica.
4. Justifica la tria del CMS principal amb **tres arguments** lligats a les activitats següents (no a gustos personals).

### Escriu la teva resposta aquí

| Requisit del CMS (versió oficial) | Comprovació a Debian | Resultat obtingut | Es compleix? |
|---|---|---|---|
| PHP … | `php --version` | … | … |
| Extensió PHP … | `php -m` | … | … |
| Base de dades … | `mariadb --version` | … | … |
| Mòdul Apache … | `apache2ctl -M` | … | … |


**CMS triat com a principal:** …

**Tres arguments lligats a les activitats següents:**
1. …
2. …
3. …

**Si algun requisit no es compleix, com el resoldràs?** …

## Com es puntua aquesta pràctica

Cadascun dels tres apartats val **0, 1 o 2 punts**
Total: **6 punts**
