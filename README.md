# IAW i ASGBD — ASIX

Repositori amb els materials dels mòduls d’**Implantació d’Aplicacions Web (IAW · 0376)** i d’**Administració de Sistemes Gestors de Bases de Dades (ASGBD · 0377)** del cicle formatiu de grau superior **ASIX (Administració de Sistemes Informàtics i Xarxes)**.

Aquí trobaràs les activitats que anirem treballant al llarg del curs: preparació de Debian amb VirtualBox i SSH, desplegament d’Apache i PHP, instal·lació i configuració de MariaDB, diagnosi d’incidències i desplegaments amb Docker.

Les activitats es fan **individualment**, reutilitzant una única VM Debian per alumne. Els dos mòduls es coordinen temporalment, però mantenen les seves activitats i qualificacions diferenciades.

---


## Índex

Les activitats apareixen en **l’ordre en què les treballarem**, alternant els dos mòduls. Cada fila correspon a una activitat o tram de treball, no necessàriament a una sola classe.

| Ordre | Mòdul | RA | Activitat | Document | Descripció |
|:-----:|:-----:|:--:|:---------:|----------|------------|
| 1 | - | - | [**Simulacre: servidor Debian amb VirtualBox i SSH**](Simulacre-servidor-Devian.md) | Crear una màquina virtual amb **Debian** a **VirtualBox** i connectar-s'hi des de l'ordinador local via **SSH**, amb xarxa **NAT** i **redireccionament de ports** (port forwarding). |
| 2 | IAW | RA1 | 1.2 | [**Instal·la Apache i PHP a Debian**](IAW/RA1/IAW-1.2.md) | Instal·lar Apache/PHP, publicar pàgines i verificar accés, permisos i registres. |
| 3 | IAW | RA1 | 1.3 | [**Resol incidències d’Apache i d’accés al web**](IAW/RA1/IAW-1.3.md) | Provocar, diagnosticar i resoldre incidències de servei, configuració, ports i permisos. |
| 4 | ASGBD | RA1 | 1.1 | [**Introducció als SGBD i selecció segons requisits**](ASGBD/RA1/ASGBD-1.1.md) | Introduir els SGBD i comparar opcions segons els requisits del cas. |
| 5 | ASGBD | RA1 | 1.2 | [**Instal·la MariaDB a Debian**](ASGBD/RA1/ASGBD-1.2.md) | Instal·lar MariaDB, verificar el servei i les dades, i localitzar configuració i registres. |

### Properament

Continuarem amb la diagnosi i la documentació de MariaDB, la configuració del SGBD, la integració amb PHP i els desplegaments amb Docker. L’índex s’ampliarà a mesura que es publiquin les activitats.

---

## 📂 Organització dels materials

- `IAW/RA1/`: activitats del RA1 d’IAW.
- `ASGBD/RA1/`: activitats del RA1 d’ASGBD.
- `ASGBD/RA2/`: activitats del RA2 d’ASGBD.

Cada enunciat indica les instruccions de treball i què cal lliurar. Consulta sempre l’activitat corresponent abans de modificar la VM.
