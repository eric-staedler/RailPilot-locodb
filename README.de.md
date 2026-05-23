# RailPilot-Lokomotivdatenbank (locodb)

Dieses Repository stellt eine PostgreSQL-Datenbank und eine PostgREST-API für die RailPilot-Anwendung bereit. Die Datenbank speichert Informationen über Lokomotiven (z. B. Namen, Adressen, Symbole, Geschwindigkeitseinstellungen usw.), die die RailPilot-App zum Betrieb benötigt. Das Datenbankschema ist in definiert`SQL/schema.sql`und eine Docker Compose-Datei (`docker-compose.yaml`) automatisiert den Aufbau und die Ausführung sowohl der PostgreSQL-Datenbank als auch des PostgREST-Dienstes.

> **Warnung:**Die Datenbank ist**nicht**über die grundlegenden PostgreSQL-Anmeldeinformationen hinaus passwortgeschützt und funktioniert auch**nicht**Benutzerauthentifizierung erzwingen. Tun**nicht**Stellen Sie diese Datenbank oder API direkt dem öffentlichen Internet zur Verfügung. Bewahren Sie es in einem vertrauenswürdigen lokalen Netzwerk oder hinter einer Firewall auf.

* * *

## Inhaltsverzeichnis

1.  [Voraussetzungen](#prerequisites)
2.  [Repository-Inhalte](#repository-contents)
3.  [Installation auf Raspberry Pi OS](#installation-on-raspberry-pi-os)
    -   [1. Installieren Sie Docker und Docker Compose](#1-install-docker--docker-compose)
    -   [2. Klonen Sie dieses Repository](#2-clone-this-repository)
    -   [3. Konfigurieren Sie Umgebungsvariablen/Passwort](#3-configure-environment-variables--password)
    -   [4. Starten Sie Dienste mit Docker Compose](#4-launch-services-with-docker-compose)
4.  [Wie es funktioniert](#how-it-works)
5.  [Verwendung](#usage)
6.  [Sicherheitsüberlegungen](#security-considerations)
7.  [Übersicht über das Datenbankschema](#database-schema-overview)
8.  [Lizenz](#license)

* * *

## Voraussetzungen

-   Ein Raspberry Pi läuft**Raspberry Pi-Betriebssystem**(32 Bit oder 64 Bit; in diesem Handbuch wird ein Debian-basiertes Betriebssystem vorausgesetzt). Oder jedes andere Gerät, auf dem ein Debian-basiertes Betriebssystem ausgeführt wird.
-   Ein Benutzerkonto mit`sudo`Privilegien.
-   Eine funktionierende Internetverbindung (um Docker-Images abzurufen und das SQL-Schema während der Initialisierung herunterzuladen).
-   Mindestens**4 GB**kostenloser Speicherplatz für Docker-Images, -Volumes und -Daten.
-   Grundlegende Vertrautheit mit der Kommandozeile/Terminal.

* * *

## Repository-Inhalte

    RailPilot-locodb/
    ├── docker-compose.yaml    # Defines PostgreSQL (locodb) and PostgREST (locoapi) services
    ├── LICENSE                # MIT License
    ├── README.md              # (This file)
    └── SQL/
        └── schema.sql         # SQL dump: tables, sequences, sample data for locomotives & functions

-   **docker-compose.yaml**
    -   Richtet zwei benannte Dienste ein:
        1.  **db**(`postgres:14-alpine`)
        2.  **postgrest**(`locoapi`)
    -   Macht PostgreSQL auf dem Port verfügbar**5432**und PostgREST am Port**3000**.
    -   Der`db`Der Dienst lädt das Schema automatisch herunter und führt es aus`SQL/schema.sql`(über a`curl … | psql`Befehl), alle Tabellen erstellen und erste Lokdaten laden.
    -   Der`postgrest`Der Dienst stellt eine Verbindung zum her`db`mit den gleichen Anmeldeinformationen.

-   **SQL/schema.sql**
    -   Ein vollständiger PostgreSQL-Dump mit:
        -   Rollen (z. B.`web_anon`)
        -   Schemata, Tabellen (`locos`,`functions`usw.), Sequenzen und Einschränkungen
        -   `COPY`Befehle, die die Datenbank mit Beispiellokomotiven und zugehörigen „Funktionsdaten“ vorladen.

-   **LIZENZ**
    -   MIT License

* * *

## Installation auf Raspberry Pi OS

Befolgen Sie diese Schritte, um Docker zu installieren, die RailPilot-Lokomotivdatenbank zu konfigurieren und die Dienste auf Ihrem Raspberry Pi zu starten.

### 1. Installieren Sie Docker und Docker Compose

Folgen[diesen Leitfaden](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

### 2. Klonen Sie dieses Repository

1.  Wechseln Sie in ein Verzeichnis, in dem Sie den Code speichern möchten, zum Beispiel Ihren Home-Ordner:
    ```bash
    cd ~
    ```

2.  Klonen Sie das GitHub-Repository:
    ```bash
    git clone https://github.com/RealPCBUILD3R/RailPilot-locodb.git
    ```
    > Wenn Sie die ZIP-Datei auf andere Weise heruntergeladen haben, entpacken Sie sie, wo immer Sie möchten`cd`in diesen Ordner.

3.  In das Projektverzeichnis wechseln:
    ```bash
    cd RailPilot-locodb
    ```

* * *

### 3. Konfigurieren Sie Umgebungsvariablen/Passwort

Der`docker-compose.yaml`Die Datei verwendet Platzhalter für PostgreSQL-Anmeldeinformationen. Sie müssen ersetzen`<Your-Password>`In**zwei**Orte:

1.  **POSTGRES_PASSWORD**(für die`db`Service)

2.  **PGRST_DB_URI**(für die`postgrest`Dienst, in der Verbindungszeichenfolge)

3.  Offen`docker-compose.yaml`in einem Texteditor (z.B.`nano`,`vi`):
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

5.  Ersetzen Sie jede Instanz von`<Your-Password>`mit einem**starkes Passwort**Ihrer Wahl (z.B.`s3cur3Pa$$w0rd`). Zum Beispiel:

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

    -   In**Nano**: drücken`CTRL+O`(schreiben),`Enter`, Dann`CTRL+X`(Ausfahrt).
    -   In**Du**: drücken`Esc`, Typ`:wq`, und drücken Sie dann`Enter`.

> **Tipp:**
>
> -   Tun**nicht**Legen Sie Ihr Passwort fest oder geben Sie es öffentlich weiter.

* * *

### 4. Starten Sie Dienste mit Docker Compose

1.  Von innen`RailPilot-locodb`Verzeichnis, führen Sie Folgendes aus:

    ```bash
    docker-compose up -d
    ```

    -   Der`-d`Flag bedeutet „getrennt“ (im Hintergrund ausgeführt).
    -   Docker ruft die erforderlichen Bilder ab (`postgres:14-alpine`Und`postgrest/postgrest:latest`), erstellen Sie ein Docker-Volume mit dem Namen`db_data`, and start two containers named `locodb`Und`locoapi`.

2.  So überprüfen Sie, ob Container ausgeführt werden:

    ```bash
    docker ps
    ```

    Sie sollten beide sehen`locodb`(PostgreSQL) und`locoapi`(PostgREST) ​​läuft.

3.  **Erstmalige Initialisierung**

    1.  Erstellt alle Tabellen, Sequenzen, Einschränkungen usw.
    2.  Kopiert Beispieldaten in die`locos`Und`functions`Tische.

    -   Nachdem dies abgeschlossen ist, wird die`locodb`Der Container läuft mit den geladenen Daten weiter.

4.  **Überprüfen Sie die Datenbankinitialisierung**(optional)
    1.  Öffnen Sie einen Webbrowser
    2.  Gehen Sie zu http&#x3A;//<IP-Address of your Device running locodb>:3000/Loks
    3.  Sie sollten etwa Folgendes sehen: \[{"id":46,"name": "Salzburger", "address":52,"icon":\\4414151536 bkha4kh...

5.  **PostgREST-API**
    -   Der`locoapi`(PostgREST) ​​Container wartet auf`locodb`verfügbar sein.
    -   Sobald eine Verbindung hergestellt werden kann, stellt es eine RESTful-JSON-API unter zur Verfügung**Port 3000**. Standardmäßig die anonyme Rolle`web_anon`kann das abfragen`public`Schema.

* * *

## Wie es funktioniert

1.  **PostgreSQL-Datenbank (`locodb`)**
    -   Läuft weiter`postgres:14-alpine`.
    -   Umgebungsvariablen:
        -   `POSTGRES_USER=admin`
        -   `POSTGRES_PASSWORD=<your_password>`
        -   `POSTGRES_DB=locodb`

2.  **PostgREST-Dienst (`locoapi`)**
    -   Läuft`postgrest/postgrest:latest`.
    -   Verbindet sich mit dem`locodb`Datenbank mit der`admin`Benutzer und Passwort, die Sie konfiguriert haben.
    -   Stellt eine RESTful-JSON-API am Port bereit**3000**. Zum Beispiel:
        -   `GET http://<IP-Address of your Device running locodb>:3000/locos`gibt alle Lokomotiven zurück.
        -   `GET http://<IP-Address of your Device running locodb>:3000/functions?loco_address=eq.52`Gibt alle Funktionen für die Lok mit Adresse 52 zurück.
    -   Der`PGRST_DB_ANON_ROLE=web_anon`Umgebungsvariable bedeutet, dass jeder in Ihrem lokalen Netzwerk die API ohne zusätzliche Authentifizierung abfragen kann.

3.  **Volumen`db_data`**
    -   Ein Docker-Volume, das Daten (das Datenverzeichnis von PostgreSQL) über Container-Neustarts oder -Neuerstellungen hinweg beibehält.
    -   Wenn Sie die Daten jemals vollständig löschen müssen, können Sie das Volume entfernen:
        ```bash
        docker-compose down -v
        ```
        > **Warnung:**Dadurch werden alle Lokdaten gelöscht.

* * *

## Verwendung

1.  **RailPilot-App-Konfiguration**
    -   Stellen Sie in Ihrer RailPilot-Anwendung (die auf Ihrem iPhone, iPad oder einem anderen Gerät läuft) die IP-Adresse der Lokomotivdatenbank auf die Adresse Ihres Geräts ein, auf dem locodb läuft, und stellen Sie den Port auf ein**3000**und das Protokoll zu http&#x3A;//.
    -   Die App kann dann Lokinformationen (z. B. Namen, Adressen, Symbole) und Funktionsdefinitionen direkt von dieser lokalen API abrufen.

2.  **Verwaltung der Container**
    -   **Stoppen**beide Dienste:
        ```bash
        docker-compose down
        ```
    -   **Start**wieder (abgetrennt):
        ```bash
        docker-compose up -d
        ```
    -   **Containerprotokolle anzeigen**(z. B. um Initialisierungsprobleme zu beheben):
        ```bash
        docker-compose logs -f
        ```

* * *

## Sicherheitsüberlegungen

-   **Nicht dem öffentlichen Internet aussetzen**
    -   Die Datenbank verfügt über keine integrierte Benutzer-/Passwort-Authentifizierung außer dem PostgreSQL-Benutzer „admin“.
    -   PostgREST erlaubt standardmäßig anonyme (`web_anon`) Zugriff auf die`public`Schema.
    -   Wenn Sie Port 5432 oder 3000 ohne Firewall oder VPN dem Internet zugänglich machen, kann jeder Ihre Datenbank lesen, ändern, löschen oder sogar schädliche Daten hochladen.
    -   **Empfehlung:**Bewahren Sie den Raspberry Pi hinter der Firewall eines Routers oder in einem privaten VLAN auf. Wenn Sie Fernzugriff benötigen, verwenden Sie ein VPN.

-   **Standardpasswort ändern**
    -   Wählen Sie ein sicheres, eindeutiges Passwort`docker-compose.yaml`für`POSTGRES_PASSWORD`.

* * *

## Übersicht über das Datenbankschema

Der`schema.sql`Datei definiert:

1.  **Rollen und Schemata**
    -   A`web_anon`Rolle mit aktivierten grundlegenden SELECT-Berechtigungen`public`.
    -   Der`public`Schema im Besitz des`admin`Benutzer.

2.  **Tische**

    -   `public.locos`
        -   Spalten:`id`,`name`,`address`,`icon`(binärer Blob),`thumbnail`(binärer Blob),`maxspeed`,`speedstep`,`fullname`,`length`,`weight`,`owner`,`dateadded`
        -   Einschränkungen stellen gültige Adressbereiche, Geschwindigkeitseinstellungen usw. sicher.
        -   Beispielreihen für verschiedene Lokomotiven (z. B. „Salzburger“ usw.).

    -   `public.functions`
        -   Spalten:`id`,`loco_address`,`number`,`shortname`,`longname`,`icontype`,`icon`,`type`(`switch`,`button`, oder`timer`),`time`(für Timer)
        -   Definiert Funktionen (Lichter, Sounds etc.) pro Lokadresse.

3.  **Sequenzen**
    -   `public.locos_id_seq`Und`public.functions_id_seq`um die automatisch zu erhöhen`id`Spalten.

4.  **Beispieldaten**
    -   Verschiedene Lokomotiven (z. B. mit den Adressen 21, 42, 52 usw.) und ihre Standardfunktionen.
    -   Diese Daten werden über geladen`COPY … FROM stdin`Befehle eingebettet in`schema.sql`.

* * *

## Lizenz

Dieses Projekt ist Open Source unter der**MIT License**. Sehen[LIZENZ](LICENSE)für Einzelheiten.

* * *

> **Zusammenfassung**
>
> -   Klonen Sie das Repo und bearbeiten Sie es`docker-compose.yaml`um ein sicheres Passwort festzulegen.
> -   Installieren Sie Docker und Docker Compose auf Ihrem Raspberry Pi.
> -   Laufen`docker-compose up -d`um die Datenbank zu erstellen und zu starten (`locodb`) und API (`locoapi`).
> -   Halten Sie Dienste hinter einer Firewall. Machen Sie die Ports 5432/3000 nicht dem öffentlichen Internet zugänglich.
> -   Genießen Sie eine lokale Offline-Datenbank für Ihr RailPilot-Setup!
