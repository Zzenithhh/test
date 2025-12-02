# Collaborative Study Platform

## 1. Stručný popis projektu a cieľov aplikácie

Aplikácia **Collaborative Study Platform** je desktopová študijná platforma určená pre vysokoškolských študentov, ktorí spolupracujú v tímoch na predmetových úlohách a projektoch.

Hlavné funkcie:

- tvorba študijných **skupín** (napr. podľa predmetu alebo projektu),
- správa **úloh** v rámci skupín (popis, stav, deadline),
- pridávanie **študijných materiálov** k úlohám (súbory, odkazy),
- **odovzdávanie riešení** úloh študentmi,
- sledovanie **deadline-ov** a plánovanie práce,
- **notifikácie** (napr. nová úloha, blížiaci sa deadline, nové riešenie),
- autentifikácia používateľov (klasický login + Google OAuth2),
- rôzne **rolové oprávnenia** (globálne roly používateľa + roly v skupinách).

Cieľom aplikácie je pomôcť študentom:

- mať všetky úlohy, materiály a odovzdané riešenia na jednom mieste,
- lepšie si rozplánovať prácu podľa deadline-ov,
- efektívne spolupracovať v rámci študijných skupín.

---

## 2. Architektúra systému

Aplikácia je rozdelená na tri hlavné časti:

- **Klient (frontend)** – desktopová aplikácia v JavaFX
- **Server (backend)** – Spring Boot aplikácia (REST API + WebSocket)
- **Databáza** – relačná databáza (SQLite)

### 2.1 Diagram architektúry

(Napr. mermaid diagram, ktorý vie GitHub zobraziť.)


```mermaid
flowchart LR
    Client[JavaFX klient\n(FXML obrazovky)] 
        --> |HTTP/JSON| API[Spring Boot backend\nREST API]

    Client <--> |WebSocket| WS[WebSocket notifikácie]

    API --> DB[(Relačná databáza\nPostgreSQL)]
    API --> FS[(Súborové úložisko\nmateriály a prílohy)]
```
---

## 3. Databázový model (ER diagram)
![ER diagram](db.png)
