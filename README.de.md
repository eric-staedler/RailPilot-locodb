# Railpilot Lokomotive -Datenbank (Locodb)

Dieses Repository bietet eine PostgreSQL -Datenbank und eine Postgrest -API für die Railpilot -Anwendung. In der Datenbank werden Informationen zu Lokomotiven (z. B. Namen, Adressen, Symbole, Geschwindigkeitseinstellungen usw.) gespeichert, die die Railpilot -App für den Betrieb benötigt. Das Datenbankschema ist in definiert in`SQL/schema.sql`und eine Docker -Komponierungsdatei (`docker-compose.yaml`) Automatisiert das Erstellen und Ausführen sowohl der PostgreSQL -Datenbank als auch des Postgrest -Dienstes.

> **Warnung:**Die Datenbank ist**nicht**Passwort geschützt über die grundlegenden PostgreSQL-Anmeldeinformationen und tut es**nicht**durchsetzen der Benutzerauthentifizierung. Tun**nicht**Stellen Sie diese Datenbank oder API direkt dem öffentlichen Internet aus. Behalten Sie es in einem vertrauenswürdigen lokalen Netzwerk oder hinter einer Firewall.

* * *

## Inhaltsverzeichnis

1.  [Voraussetzungen](#prerequisites)
2.  [Repository -Inhalt](#repository-contents)
3.  [Installation auf Raspberry Pi OS](#installation-on-raspberry-pi-os)
    -   [1. Installieren Sie Docker & Docker Compose](#1-install-docker--docker-compose)
    -   [2. Klonen Sie dieses Repository klonen](#2-clone-this-repository)
    -   [3. Konfigurieren Sie Umgebungsvariablen / Passwort](#3-configure-environment-variables--password)
    -   [4. Startdienste mit Docker Compose](#4-launch-services-with-docker-compose)
4.  [Wie es funktioniert](#how-it-works)
5.  [Verwendung](#usage)
6.  [Sicherheitsüberlegungen](#security-considerations)
7.  [Datenbankschemaübersicht](#database-schema-overview)
8.  [Lizenz](#license)

* * *

## Voraussetzungen

-   Ein Himbeer -Pi läuft**Raspberry Pi OS**(32 Bit oder 64 Bit; dieser Leitfaden nimmt ein Debian-basiertes Betriebssystem an). Oder ein anderes Gerät, das ein Debian-basiertes Betriebssystem ausführt.
-   Ein Benutzerkonto mit`sudo`Privilegien.
-   Eine funktionierende Internetverbindung (um Docker -Bilder zu ziehen und das SQL -Schema während der Initialisierung herunterzuladen).
-   Mindestens**4 GB**von freiem Speicher für Docker -Bilder, Volumina und Daten.
-   Grundlegende Vertrautheit mit der Befehlszeile / dem Terminal.

* * *

## Repository -Inhalt

    RailPilot-locodb/
    ├── docker-compose.yaml    # Defines PostgreSQL (locodb) and PostgREST (locoapi) services
    ├── LICENSE                # MIT License
    ├── README.md              # (This file)
    └── SQL/
        └── schema.sql         # SQL dump: tables, sequences, sample data for locomotives & functions

-   **Docker-compose.yaml**
    -   Richtet zwei benannte Dienste ein:
        1.  **db**(`postgres:14-alpine`)
        2.  **Postgrest**(`locoapi`)
    -   Enthält PostgreSQL am Port**5432**und Postgrest am Port**3000**.
    -   Der`db`Der Service lädt das Schema automatisch herunter und führt aus`SQL/schema.sql`(über a`curl … | psql`Befehl), Erstellen aller Tabellen und Laden Sie anfängliche Lokomotivdaten.
    -   Der`postgrest`Service verbindet sich mit dem`db`Verwenden der gleichen Anmeldeinformationen.

-   **Sql/schema.sql**
    -   Ein vollständiger Postgresql -Dump mit:
        -   Rollen (z. B.,,`web_anon`)
        -   Schemas, Tabellen (`locos`,`functions`usw.), Sequenzen und Einschränkungen
        -   `COPY`Befehle, die die Datenbank mit Beispiellokomotiven und zugehörigen „Funktionen“ -Daten vorladen.

-   **LIZENZ**
    -   MIT License

* * *

## Installation auf Raspberry Pi OS

Befolgen Sie diese Schritte zur Installation von Docker, konfigurieren Sie die Railpilot -Lokomotive -Datenbank und starten Sie die Dienste auf Ihrem Raspberry PI.

### 1. Installieren Sie Docker & Docker Compose

Folgen[Dieser Leitfaden](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

### 2. Klonen Sie dieses Repository klonen

1.  Wechseln Sie in ein Verzeichnis, in dem Sie den Code speichern möchten, zum Beispiel in Ihrem Heimatordner:
    ```bash
    cd ~
    ```

2.  Klonen Sie das Github -Repository:
    ```bash
    git clone https://github.com/RealPCBUILD3R/RailPilot-locodb.git
    ```
    > Wenn Sie den Reißverschluss mit anderen Mitteln heruntergeladen haben, entpacken Sie ihn, wo Sie möchten und`cd`in diesen Ordner.

3.  Umzug in das Projektverzeichnis:
    ```bash
    cd RailPilot-locodb
    ```

* * *

### 3. Konfigurieren Sie Umgebungsvariablen / Passwort

Der`docker-compose.yaml`Die Datei verwendet Platzhalter für PostgreSQL -Anmeldeinformationen. Sie müssen ersetzen`<Your-Password>`In**zwei**Orte:

1.  **Postgres_password**(für die`db`Service)

2.  **Pgrst_db_uri**(für die`postgrest`Dienst in der Verbindungszeichenfolge)

3.  Offen`docker-compose.yaml`in einem Texteditor (z. B.,,`nano`,`vi`):
    ```bash
    nano docker-compose.yaml
    ```

4.  Suchen Sie diese Abschnitte:

    ```yaml
    services:
      db:
        image: postgres:14-alpine
        container_name: locodb
        environment:
          POSTGRES_USER: admin
          POSTGRES_PASSWORD: <Your-Password>
          POSTGRES_DB: locodb
        …

      postgrest:
        image: postgrest/postgrest:latest
        container_name: locoapi
        environment:
          PGRST_DB_URI: postgres://admin:<Your-Password>@db:5432/locodb
          PGRST_DB_SCHEMA: public
          PGRST_DB_ANON_ROLE: web_anon
        …
    ```

5.  Ersetzen Sie jede Instanz von`<Your-Password>`mit a**starkes Passwort**Ihrer Wahl (z. B.,,`s3cur3Pa$$w0rd`). Zum Beispiel:

    ```yaml
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: s3cur3Pa$$w0rd
      POSTGRES_DB: locodb
    ```

    Und:

    ```yaml
    environment:
      PGRST_DB_URI: postgres://admin:s3cur3Pa$$w0rd@db:5432/locodb
      PGRST_DB_SCHEMA: public
      PGRST_DB_ANON_ROLE: web_anon
    ```

6.  Speichern und beenden:

    -   In**Nano**: Drücken Sie`CTRL+O`(schreiben),`Enter`, Dann`CTRL+X`(Ausfahrt).
    -   In**du/vim**: Drücken Sie`Esc`, Typ`:wq`, dann drücken Sie`Enter`.

> **Tipp:**
>
> -   Tun**nicht**Verpfändung oder teilen Sie Ihr Passwort öffentlich.

* * *

### 4. Startdienste mit Docker Compose

1.  Von im Inneren`RailPilot-locodb`Verzeichnis, rennen:

    ```bash
    docker-compose up -d
    ```

    -   Der`-d`Flag bedeutet "abgelöst" (im Hintergrund laufen).
    -   Docker wird die erforderlichen Bilder ziehen (`postgres:14-alpine`Und`postgrest/postgrest:latest`), erstellen Sie einen Docker -Band mit dem Namen`db_data`, und starten Sie zwei benannte Behälter`locodb`Und`locoapi`.

2.  Um zu überprüfen, ob Container ausgeführt werden:

    ```bash
    docker ps
    ```

    Sie sollten beide sehen`locodb`(Postgresql) und`locoapi`(Postgrest) Laufen.

3.  **Initialisierung der ersten Zeit**

    -   Der`locodb`(PostgreSQL) Container führt einen Ein -Shot aus`command:`das installiert`curl`und dann holt sich`SQL/schema.sql`Aus dem Github -Repository's`main`Zweig. Es wächst, in die sich direkt an die SQL befasst`psql`, welche:
        1.  Erstellt alle Tabellen, Sequenzen, Einschränkungen usw.
        2.  Kopiert Beispieldaten in die`locos`Und`functions`Tische.

    -   Nach Abschluss ist die`locodb`Der Container wird mit den geladenen Daten fortgesetzt.

4.  **Überprüfen Sie die Datenbankinitialisierung**(optional)
    1.  Öffnen Sie einen Webbrowser
    2.  Gehen Sie zu http&#x3A; //<IP-Address of your Device running locodb>: 3000/locos
    3.  Sie sollten so etwas sehen: \[{"ID": 46, "Name": "Salzburger", "Adresse": 52, "Symbol": "\\Kassaka, 4414151536 Bakha 4 Bruder ...

5.  **Postgrest -API**
    -   Der`locoapi`(Postgrest) Container warten auf`locodb`verfügbar sein.
    -   Sobald es eine Verbindung herstellen kann, enthüllt es eine erholsame JSON -API bei**Port 3000**. Standardmäßig die anonyme Rolle`web_anon`kann die abfragen`public`Schema.

* * *

## Wie es funktioniert

1.  **PostgreSQL -Datenbank (`locodb`)**
    -   Läuft weiter`postgres:14-alpine`.
    -   Umgebungsvariablen:
        -   `POSTGRES_USER=admin`
        -   `POSTGRES_PASSWORD=<your_password>`
        -   `POSTGRES_DB=locodb`
    -   Installationen beim Start`curl`und läuft:
        ```bash
        curl -H 'Cache-Control: no-cache' -fsSL        https://raw.githubusercontent.com/RealPCBUILD3R/RailPilot-locodb/refs/heads/main/SQL/schema.sql        | psql -U admin -d locodb
        ```
    -   Dies initialisiert das Datenbankschema (Tabellen, Sequenzen, Einschränkungen) und lädt Beispiel -Lokomotivdaten.

2.  **Postgrest -Service (`locoapi`)**
    -   Läuft`postgrest/postgrest:latest`.
    -   Verbindet sich mit dem`locodb`Datenbank mit dem`admin`Benutzer und Passwort, das Sie konfiguriert haben.
    -   Dient einer erholsamen JSON -API am Port**3000**. Zum Beispiel:
        -   `GET http://<IP-Address of your Device running locodb>:3000/locos`Gibt alle Lokomotiven zurück.
        -   `GET http://<IP-Address of your Device running locodb>:3000/functions?loco_address=eq.52`Gibt alle Funktionen für die Lokomotive mit Adresse 52 zurück.
    -   Der`PGRST_DB_ANON_ROLE=web_anon`Umgebungsvariable bedeutet, dass jeder in Ihrem lokalen Netzwerk die API ohne zusätzliche Authentifizierung abfragen kann.

3.  **Volumen`db_data`**
    -   Ein Docker -Volumen, das Daten (PostgreSQL's Data Directory) über Container -Neustarts oder Erholungen hinweg bestehen.
    -   Wenn Sie die Daten jemals vollständig löschen müssen, können Sie das Volumen entfernen:
        ```bash
        docker-compose down -v
        ```
        > **Warnung:**Dadurch wird alle Lokomotivdaten gelöscht.

* * *

## Verwendung

1.  **Railpilot App -Konfiguration**
    -   Stellen Sie in Ihrer Railpilot -Anwendung (ausgeführt auf Ihrem iPhone, iPad oder einem anderen Gerät) die IP -Adresse der Lokomotivdatenbank auf die Adresse Ihres Geräts ein, das locodB ausgeführt wird. Stellen Sie den Port auf den Port fest**3000**und das Protokoll zu http&#x3A; //.
    -   Die App kann dann Lokomotive -Informationen (z. B. Namen, Adressen, Symbole) und Funktionsdefinitionen direkt aus dieser lokalen API abrufen.

2.  **Behälter verwalten**
    -   **Stoppen**Beide Dienste:
        ```bash
        docker-compose down
        ```
    -   **Start**wieder (abgelöst):
        ```bash
        docker-compose up -d
        ```
    -   **Containerprotokolle anzeigen**(z. B. Debugug Initialisierungsprobleme):
        ```bash
        docker-compose logs -f
        ```

* * *

## Sicherheitsüberlegungen

-   **Nicht dem öffentlichen Internet aussetzen**
    -   Die Datenbank verfügt über keine integrierte Benutzer-/Kennwortauthentifizierung über den Benutzer von PostgreSQL "Admin".
    -   Standardmäßig postgrest ermöglicht Anonymous (`web_anon`) Zugang zum`public`Schema.
    -   Wenn Sie Port 5432 oder 3000 ohne Firewall oder VPN dem Internet dem Internet aussetzen, kann jemand schädliche Daten in Ihre Datenbank lesen, ändern, löschen oder sogar hochladen.
    -   **Empfehlung:**Halten Sie den Himbeer -Pi hinter der Firewall eines Routers oder auf einem privaten VLAN. Wenn Sie Remote -Zugriff benötigen, verwenden Sie ein VPN.

-   **Standardkennwort ändern**
    -   Wählen Sie ein starkes, eindeutiges Passwort in`docker-compose.yaml`für`POSTGRES_PASSWORD`.

* * *

## Datenbankschemaübersicht

Der`schema.sql`Datei definiert:

1.  **Rollen und Schemata**
    -   A`web_anon`Rolle mit grundlegenden Auswahlrechten auf`public`.
    -   Der`public`Schema des Besitzes der`admin`Benutzer.

2.  **Tische**

    -   `public.locos`
        -   Spalten:`id`,`name`,`address`,`icon`(binärer Blob),`thumbnail`(binärer Blob),`maxspeed`,`speedstep`,`fullname`,`length`,`weight`,`owner`,`dateadded`
        -   Einschränkungen gewährleisten gültige Adressbereiche, Geschwindigkeitseinstellungen usw.
        -   Probenreihen für verschiedene Lokomotiven (z. B. „Salzburger“ usw.).

    -   `public.functions`
        -   Spalten:`id`,`loco_address`,`number`,`shortname`,`longname`,`icontype`,`icon`,`type`(`switch`,`button`, oder`timer`),`time`(für Timer)
        -   Definiert Funktionen (Lichter, Geräusche usw.) pro Lokomotivadresse.

3.  **Sequenzen**
    -   `public.locos_id_seq`Und`public.functions_id_seq`automatisch inkrementieren`id`Spalten.

4.  **Beispieldaten**
    -   Verschiedene Lokomotiven (z. B. mit den Adressen 21, 42, 52 usw.) und deren Standardfunktionen.
    -   Diese Daten werden über geladen`COPY … FROM stdin`Befehle eingebettet`schema.sql`.

* * *

## Lizenz

Dieses Projekt ist Open Source unter dem**MIT License**. Sehen[LIZENZ](LICENSE)für Details.

* * *

> **Zusammenfassung**
>
> -   Klonen Sie das Repo, bearbeiten Sie`docker-compose.yaml`ein starkes Passwort festlegen.
> -   Installieren Sie Docker & Docker Compose auf Ihrem Raspberry Pi.
> -   Laufen`docker-compose up -d`Erstellen und Start der Datenbank (`locodb`) und API (`locoapi`).
> -   Halten Sie die Dienste hinter einer Firewall; Stellen Sie die Ports 5432/3000 nicht dem öffentlichen Internet aus.
> -   Genießen Sie eine lokale Offline -Datenbank für Ihr Railpilot -Setup!
