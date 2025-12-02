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

```mermaid
flowchart LR
    Client[JavaFX klient\n(FXML obrazovky)] 
        --> |HTTP/JSON| API[Spring Boot backend\nREST API]

    Client <--> |WebSocket| WS[WebSocket notifikácie]

    API --> DB[(Relačná databáza\nPostgreSQL)]
    API --> FS[(Súborové úložisko\nmateriály a prílohy)]
````

### 2.2 Vysvetlenie vrstiev

* **JavaFX klient**

  * zobrazuje GUI (FXML scény),
  * rieši prihlasovanie, zobrazenie skupín, úloh, materiálov, odovzdávok,
  * komunikuje so serverom cez REST API a WebSocket,
  * zohľadňuje roly používateľa a roly v skupine (čo môže vidieť/meniť).

* **Spring Boot backend**

  * implementuje obchodnú logiku,
  * poskytuje REST API pre klienta (CRUD operácie nad entitami),
  * zabezpečuje autentifikáciu a autorizáciu (JWT + Google OAuth2),
  * spracúva súbory (uloženie materiálov a odovzdaných riešení),
  * posiela notifikácie cez WebSocket (napr. nové úlohy, deadline).

* **Databáza**

  * ukladá dáta o používateľoch, skupinách, úlohách, materiáloch, odovzdaných riešeniach a notifikáciách,
  * zabezpečuje konzistenciu dát a väzby medzi entitami.


---

## 3. Databázový model (ER diagram)
![ER diagram](db.png)

Звісно!
Ось **повна словацька версія** твоєї стислої API-документації.

---

# 4. Dokumentácia REST API a WebSocket endpointov

Základná URL: **`http://localhost:8080`**

---

# Auth API

## **POST /api/auth/register**

Vytvorenie nového používateľa.
**Body:** `RegisterRequest`
**Response:** stav (200–201)

## **POST /api/auth/login**

Prihlásenie používateľa.
**Body:** `{ username, password }`
**Response:** JWT token / user objekt

## **POST /api/auth/logout**

Odhlásenie
**Response:** stav

## **GET /api/auth/me**

Získanie informácií o aktuálnom používateľovi
**Response:** `UserDetailedDTO`

---

# Users API

## **GET /api/user**

Zoznam používateľov
**Query params:** `page`, `size`, `search`
**Response:** `PageUserPartialDTO`

## **POST /api/user**

Vytvoriť používateľa
**Body:** `UserCreateDTO`
**Response:** stav

## **GET /api/user/{id}**

Získať jedného používateľa
**Response:** `UserDetailedDTO`

## **PATCH /api/user/{id}**

Aktualizovať používateľa
**Body:** `UserUpdateDTO`
**Response:** stav

## **DELETE /api/user/{id}**

Odstrániť používateľa
**Response:** stav

## **GET /api/user/{id}/stats**

Štatistiky používateľa (počet úloh v jednotlivých stavoch)
**Response:** `UserStats`

---

# Groups API

## **GET /api/group**

Zoznam skupín
**Query params:** `search`, `userId`, `page`, `size`
**Response:** `PageGroupPartialDTO`

## **POST /api/group**

Vytvoriť skupinu
**Body:** `GroupCreateDTO`
**Response:** stav

## **GET /api/group/{id}**

Detail skupiny
**Response:** `GroupDetailedDTO`

## **PATCH /api/group/{id}**

Aktualizovať skupinu
**Body:** `GroupUpdateDTO`
**Response:** stav

## **DELETE /api/group/{id}**

Odstrániť skupinu
**Response:** stav

---

# Memberships API

## **POST /api/membership**

Pridať používateľa do skupiny
**Body:** `MembershipCreateDTO`
**Response:** stav

## **PATCH /api/membership/{id}**

Aktualizovať rolu v skupine
**Body:** `MembershipUpdateDTO`
**Response:** stav

## **DELETE /api/membership/{id}**

Odstrániť člena skupiny
**Response:** stav

---

# Tasks API

## **GET /api/task**

Zoznam úloh
**Query params:**

* `groupId`, `userId`, `status`
* stránkovanie: `page`, `size`
  **Response:** `PageTaskPartialDTO`

## **POST /api/task**

Vytvoriť úlohu
**Body:** `TaskCreateDTO`
**Response:** stav

## **GET /api/task/{id}**

Získať detail úlohy
**Response:** `TaskDetailedDTO`

## **PATCH /api/task/{id}**

Aktualizovať úlohu
**Body:** `TaskUpdateDTO`
**Response:** stav

## **DELETE /api/task/{id}**

Odstrániť úlohu
**Response:** stav

## **GET /api/task/active**

Aktívne úlohy (IN_PROGRESS + IN_REVIEW)
**Query params:** `groupId`, `userId`, `page`, `size`
**Response:** `PageTaskPartialDTO`

---

# Resources API

## **GET /api/resource**

Zoznam zdrojov
**Query params:** `taskId`, `page`, `size`
**Response:** `PageResourceDetailedDTO`

## **POST /api/resource**

Vytvoriť zdroj
**Body:** `ResourceCreateDTO`
**Response:** `ResourceShortDTO`

## **GET /api/resource/{id}**

Získať zdroj
**Response:** `ResourceDetailedDTO`

## **PATCH /api/resource/{id}**

Aktualizovať zdroj
**Body:** `ResourceUpdateDTO`
**Response:** stav

## **DELETE /api/resource/{id}**

Odstrániť zdroj
**Response:** stav

---

# Files for Resources (Súbory)

## **POST /api/resource/{id}/file**

Nahrať súbor
**Body:** multipart/form-data (file)

## **GET /api/resource/{id}/file**

Stiahnuť alebo získať súbor

## **DELETE /api/resource/{id}/file**

Odstrániť súbor

---

# Logs API

## **GET /api/log**

Zoznam aktivít
**Query params:** `page`, `size`
**Response:** `PageActivityLogDetailedDTO`

## **GET /api/log/{id}**

Detail aktivity
**Response:** `ActivityLogDetailedDTO`

## **GET /api/log/user/{id}**

Aktivity podľa používateľa
**Query params:** `page`, `size`
**Response:** `PageActivityLogDetailedDTO`

---

# WebSocket Notifikácie

Backend používa WebSocket kanál na odosielanie real-time notifikácií používateľom.
Odosielanie správ zabezpečuje služba **`NotificationService`**, ktorá posiela správy pomocou:

* `messageHandler.sendToUser(userId, message)`
* používateľ je identifikovaný cez svoj *userId*
* správy majú formát `NotificationMessage`

---

# Formát notifikácie

Každá notifikácia sa odosiela ako objekt:

```json
{
  "type": "TASK_CREATED | DEADLINE_SOON | TASK_UPDATED | TASK_COMPLETED | GROUP_INVITATION",
  "title": "string",
  "message": "string",
  "taskId": 123,    
  "groupId": 456
}
```

### Popis polí

| Pole        | Typ       | Popis                      |
| ----------- | --------- | -------------------------- |
| **type**    | enum      | Typ notifikácie            |
| **title**   | string    | Krátky nadpis notifikácie  |
| **message** | string    | Detailný text              |
| **taskId**  | long/null | ID úlohy, ak sa týka úlohy |
| **groupId** | long      | ID skupiny                 |

---

# Typy notifikácií (NotificationType)

| Typ                | Kedy sa používa                   |
| ------------------ | --------------------------------- |
| `TASK_CREATED`     | Nová úloha bola vytvorená         |
| `DEADLINE_SOON`    | Blíži sa deadline úlohy           |
| `TASK_UPDATED`     | Úloha bola upravená               |
| `TASK_COMPLETED`   | Úloha bola dokončená              |
| `GROUP_INVITATION` | Používateľ bol pozvaný do skupiny |

---

# WebSocket endpoint

Frontend sa pripája na WebSocket cez:

```
ws://localhost:8080/ws
```

Každý používateľ má súkromný kanál, na ktorý server posiela správy:

```
/user/{userId}/queue/notifications
```

---

# Činnosť NotificationService

Služba automaticky vyhľadá všetkých používateľov v skupine a odošle im notifikáciu.

## **1. notifyTaskCreated(Task task)**

Odosiela notifikáciu všetkým členom skupiny, ak bola vytvorená nová úloha.

**Typ:** `TASK_CREATED`
**Message:** „New task '<title>' in group <groupName>“

---

## **2. notifyDeadlineSoon(Task task)**

Odosiela upozornenie, že sa blíži deadline úlohy.

**Typ:** `DEADLINE_SOON`
**Message:** „Task '<title>' deadline is near“

---

## **3. notifyTaskUpdated(Task task)**

Odosiela informáciu, že úloha bola aktualizovaná.

**Typ:** `TASK_UPDATED`
**Message:** „Task '<title>' was updated“

---

## **4. notifyTaskCompleted(Task task)**

Odosiela správu, že úloha bola dokončená.

**Typ:** `TASK_COMPLETED`
**Message:** „Task '<title>' was marked as completed“

---

## **5. notifyGroupInvitation(Group group, int invitedUserId)**

Odosiela pozvánku do skupiny jednému používateľovi.

**Typ:** `GROUP_INVITATION`
**Message:** „You were invited to group '<groupName>'“

---

# Príklad správy, ktorú frontend dostane

```json
{
  "type": "TASK_UPDATED",
  "title": "Task updated",
  "message": "Task 'Prepare slides' was updated",
  "taskId": 12,
  "groupId": 4
}
```

# Ukážky používateľského Rozhrania

![Login okno](docs/ui-login.png)
*Prihlasovacie okno – používateľ sa môže prihlásiť cez email/heslo alebo Google OAuth2.*

![Register okno](docs/ui-register.png)
*Registračne okno – používateľ sa môže registrovať cez email/heslo alebo Google OAuth2.*

![Zoznam skupín](docs/ui-groups.png)
*Prehľad skupín, do ktorých používateľ patrí. Možnosť vytvoriť novú skupinu.*

![Zoznam úloh](docs/ui-group-tasks.png)
*Detail vybranej skupiny s prehľadom úloh, členov a základnými štatistikami.*

![Detail úlohy a odovzdanie riešenia](docs/ui-task-submission.png)
*Detail úlohy, zoznam materiálov a formulár na odovzdanie riešenia.*





---

## 3. Databázový model (ER diagram)

### 3.1 ER diagram

> Sem vložte obrázok exportovaný z IntelliJ / iného nástroja.

```markdown
![ER diagram](./docs/er-diagram.png)
```

### 3.2 Hlavné entity a väzby (stručne)

Príklad (prispôsobte reálnemu modelu):

* **User**

  * `id`, `email`, `passwordHash`, `name`, `globalRole` (napr. STUDENT, TEACHER, ADMIN)
* **Group**

  * `id`, `name`, `description`
* **GroupMembership**

  * `user_id`, `group_id`, `roleInGroup` (napr. OWNER, TUTOR, MEMBER)
  * väzba medzi `User` a `Group` (M:N)
* **Task**

  * `id`, `group_id`, `title`, `description`, `deadline`, `status`
* **TaskMaterial**

  * `id`, `task_id`, `filePath` alebo `url`, `type`
* **Submission**

  * `id`, `task_id`, `author_id`, `submittedAt`, `status`
* **SubmissionFile**

  * `id`, `submission_id`, `filePath`
* **Notification**

  * `id`, `user_id`, `type`, `message`, `createdAt`, `read`

Databázový model podporuje funkcie aplikácie nasledovne:

* väzba **User – Group – GroupMembership** umožňuje rôzne roly v skupinách,
* väzba **Group – Task – TaskMaterial** umožňuje organizovať úlohy a materiály v rámci skupiny,
* väzba **Task – Submission – SubmissionFile** pokrýva proces odovzdávania riešení,
* tabuľka **Notification** umožňuje ukladať a zobrazovať systémové notifikácie.

---

## 4. Dokumentácia REST API a WebSocket endpointov

Nižšie je prehľad hlavných endpointov. (Endpointy prispôsobte podľa reálnej aplikácie.)

### 4.1 Autentifikácia

| Metóda | URL                  | Popis                              |
| ------ | -------------------- | ---------------------------------- |
| POST   | `/api/auth/login`    | Prihlásenie používateľa, vráti JWT |
| POST   | `/api/auth/register` | Registrácia nového používateľa     |
| GET    | `/api/auth/me`       | Informácie o aktuálne prihlásenom  |

**Príklad požiadavky – login**

```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "student@example.com",
  "password": "heslo123"
}
```

**Príklad odpovede**

```json
{
  "token": "jwt_token_string",
  "user": {
    "id": 1,
    "email": "student@example.com",
    "name": "Test Student",
    "globalRole": "STUDENT"
  }
}
```

### 4.2 Skupiny

| Metóda | URL                | Popis                                |
| ------ | ------------------ | ------------------------------------ |
| GET    | `/api/groups`      | Zoznam skupín aktuálneho používateľa |
| POST   | `/api/groups`      | Vytvorenie novej skupiny             |
| GET    | `/api/groups/{id}` | Detail skupiny (členovia, úlohy)     |
| PUT    | `/api/groups/{id}` | Úprava údajov skupiny                |
| DELETE | `/api/groups/{id}` | Zmazanie skupiny (len owner/admin)   |

**Príklad odpovede – detail skupiny**

```json
{
  "id": 10,
  "name": "OOP - projekt",
  "description": "Skupina pre semestrálny projekt",
  "members": [
    { "userId": 1, "name": "Alice", "roleInGroup": "OWNER" },
    { "userId": 2, "name": "Bob", "roleInGroup": "MEMBER" }
  ],
  "tasks": [
    {
      "id": 100,
      "title": "Implementácia servera",
      "deadline": "2025-01-15T23:59:59",
      "status": "OPEN"
    }
  ]
}
```

### 4.3 Úlohy a materiály

| Metóda | URL                             | Popis                      |
| ------ | ------------------------------- | -------------------------- |
| GET    | `/api/groups/{groupId}/tasks`   | Zoznam úloh v skupine      |
| POST   | `/api/groups/{groupId}/tasks`   | Vytvorenie novej úlohy     |
| GET    | `/api/tasks/{taskId}`           | Detail úlohy               |
| PUT    | `/api/tasks/{taskId}`           | Úprava úlohy               |
| DELETE | `/api/tasks/{taskId}`           | Zmazanie úlohy             |
| POST   | `/api/tasks/{taskId}/materials` | Nahratie materiálu k úlohe |
| GET    | `/api/tasks/{taskId}/materials` | Zoznam materiálov k úlohe  |

### 4.4 Odovzdávanie riešení

| Metóda | URL                                     | Popis                                        |
| ------ | --------------------------------------- | -------------------------------------------- |
| POST   | `/api/tasks/{taskId}/submissions`       | Odovzdanie riešenia                          |
| GET    | `/api/tasks/{taskId}/submissions`       | Zoznam odovzdaní (podľa role – učiteľ/owner) |
| GET    | `/api/submissions/{submissionId}`       | Detail konkrétneho odovzdania                |
| POST   | `/api/submissions/{submissionId}/files` | Nahratie prílohy k odovzdaniu                |

### 4.5 Notifikácie

| Metóda | URL                            | Popis                                     |
| ------ | ------------------------------ | ----------------------------------------- |
| GET    | `/api/notifications`           | Zoznam notifikácií aktuálneho používateľa |
| POST   | `/api/notifications/{id}/read` | Označenie notifikácie ako prečítanej      |

### 4.6 WebSocket endpointy

Príklad:

* endpoint: `ws://<server>/ws/notifications`
* účel: push notifikácie na klienta (napr. nová úloha v skupine, blížiaci sa deadline, nové riešenie úlohy).

Formát správy (príklad):

```json
{
  "type": "TASK_DEADLINE_SOON",
  "groupId": 10,
  "taskId": 100,
  "message": "Deadline úlohy \"Implementácia servera\" sa blíži."
}
```

---

## 5. Ukážky používateľského rozhrania

> Sem vložte screenshoty JavaFX okien a stručný popis.

Príklady (zmeniť podľa reálnych súborov):

```markdown
![Login okno](./docs/ui-login.png)
*Prihlasovacie okno – používateľ sa môže prihlásiť cez email/heslo alebo Google OAuth2.*

![Zoznam skupín](./docs/ui-groups.png)
*Prehľad skupín, do ktorých používateľ patrí. Možnosť vytvoriť novú skupinu.*

![Detail skupiny a úloh](./docs/ui-group-tasks.png)
*Detail vybranej skupiny s prehľadom úloh, členov a základnými štatistikami.*

![Detail úlohy a odovzdanie riešenia](./docs/ui-task-submission.png)
*Detail úlohy, zoznam materiálov a formulár na odovzdanie riešenia.*
```

**Základný tok práce s aplikáciou:**

1. Používateľ sa prihlási (alebo zaregistruje, prípadne použije Google login).
2. Zobrazí sa mu zoznam skupín, do ktorých patrí.
3. V rámci skupiny vidí úlohy, materiály a členov.
4. Študent si otvorí konkrétnu úlohu, stiahne materiály a odovzdá riešenie.
5. Vlastník/tútor skupiny môže pridávať úlohy, meniť ich stav a kontrolovať odovzdané riešenia.
6. Notifikácie informujú používateľa o nových úlohách a blížiacich sa deadline-och.

---

## 6. Popis výziev a riešení

Počas vývoja som narazil na viaceré technické výzvy:

* **Univerzálnosť klienta a roly**

  * Bolo náročné navrhnúť JavaFX obrazovky tak, aby fungovali pre rôzne roly používateľa (globálna rola) aj rôzne roly v skupinách.
  * Riešenie: oddelenie logiky právomocí od UI, použitie viacerých DTO (short/partial/detailed) podľa potreby obrazovky.

* **Prechod z cookies-session na JWT**

  * Pôvodne bol backend postavený na session s cookies.
  * Pre integráciu **Google OAuth2** a jednoduchšiu desktopovú komunikáciu som prešiel na **JWT**.
  * Bolo potrebné upraviť bezpečnostnú konfiguráciu, filter, aj klienta (posielanie tokenu v hlavičkách).

* **Veľa typov DTO**

  * V priebehu vývoja sa ukázalo, že jeden DTO pre entitu nestačí (napr. skupina v zozname vs. detail skupiny).
  * Riešenie: zaviedol som **short / partial / detailed DTO**, aby klient vždy dostal len potrebné dáta.

* **Problémy s modulárnym systémom v klientovi**

  * Java modulárny systém (module-info) komplikoval importy a používanie knižníc v JavaFX kliente.
  * Riešenie: úprava module-info, jasnejšie rozdelenie modulov a závislostí.

* **MapStruct**

  * MapStruct sa občas "záhadne" prestal kompilovať (chýbajúce generované triedy a pod.).
  * Riešenie: čistenie build-u, kontrola anotácií, správne balíčkovanie a postupné jednoduché mapovania.

---

## 7. Zhodnotenie práce s AI

Pri vývoji som využíval AI nástroje (napr. ChatGPT) najmä nasledovne:

* **Backend (server)**

  * Kód servera je takmer celý napísaný manuálne.
  * AI som používal hlavne pri riešení nezrozumiteľných **exception-ov**, pri vysvetlení správania Springu a pri konzultácii návrhu (napr. bezpečnosť, správa tokenov).
  * Naučil som sa lepšie čítať stack trace a cielene hľadať príčinu problému.

* **Frontend (JavaFX klient)**

  * Na začiatku som AI používal intenzívnejšie – potreboval som pochopiť:

    * ako funguje **FXML**, controllery a event callbacky,
    * ako sa správne **prepínajú scény**,
    * ako organizovať kód pre znovupoužiteľné obrazovky.
  * Postupne, ako som sa v JavaFX zorientoval, som AI používal skôr ako konzultanta a nie generátor hotového kódu.

* **Čo bolo potrebné manuálne doladiť**

  * Generovaný alebo navrhnutý kód od AI bolo väčšinou potrebné prispôsobiť aktuálnej štruktúre projektu (balíčky, názvy entít, DTO).
  * UI logiku (konkrétne obrazovky, rozloženie a práva používateľa) bolo nutné navrhnúť manuálne, pretože závisí od konkrétneho modelu roly v aplikácii.

* **Čo som sa naučil**

  * efektívnejšie pracovať so Spring Boot (bezpečnosť, JWT, integrácia OAuth2),
  * základy návrhu desktopového GUI v JavaFX,
  * ako si navrhnúť vlastný ER model a premietnuť ho do JPA entít.

```
