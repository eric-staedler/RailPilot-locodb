# Database Locomotive RailPilot (LOCODB)

Questo repository fornisce un database PostgreSQL e un'API Postgrest per l'applicazione RailPilot. Il database memorizza le informazioni sulle locomotive (ad es. Nomi, indirizzi, icone, impostazioni di velocità, ecc.), Che l'app RailPilot deve funzionare. Lo schema del database è definito in`SQL/schema.sql`e un file di compositore Docker (`docker-compose.yaml`) Automatizza la costruzione e l'esecuzione sia del database PostgreSQL che del servizio Postgrest.

> **Avvertimento:**Il database è**non**protetto da password oltre le credenziali di base PostgreSQL e lo fa**non**imporre l'autenticazione dell'utente. Fare**non**Esporre questo database o API direttamente a Internet pubblico. Tienilo su una rete locale di fiducia o dietro un firewall.

* * *

## Sommario

1.  [Prerequisiti](#prerequisites)
2.  [Contenuto del repository](#repository-contents)
3.  [Installazione sul sistema operativo Raspberry Pi](#installation-on-raspberry-pi-os)
    -   [1. Installa Docker & Docker Compose](#1-install-docker--docker-compose)
    -   [2. Clona questo repository](#2-clone-this-repository)
    -   [3. Configurare variabili / password di ambiente](#3-configure-environment-variables--password)
    -   [4. Servizi di lancio con Docker Compose](#4-launch-services-with-docker-compose)
4.  [Come funziona](#how-it-works)
5.  [Utilizzo](#usage)
6.  [Considerazioni sulla sicurezza](#security-considerations)
7.  [Panoramica dello schema del database](#database-schema-overview)
8.  [Licenza](#license)

* * *

## Prerequisiti

-   Un Raspberry Pi in esecuzione**Raspberry Pi OS**(32 bit o 64 bit; questa guida presuppone un sistema operativo basato su Debian). O qualsiasi altro dispositivo che esegue un sistema operativo basato su Debian.
-   Un account utente con`sudo`privilegi.
-   Una connessione Internet funzionante (per estrarre le immagini Docker e scaricare lo schema SQL durante l'inizializzazione).
-   Almeno**4GB**di archiviazione gratuita per immagini, volumi e dati Docker.
-   Familiarità di base con la riga di comando / terminale.

* * *

## Contenuto del repository

    RailPilot-locodb/
    ├── docker-compose.yaml    # Defines PostgreSQL (locodb) and PostgREST (locoapi) services
    ├── LICENSE                # MIT License
    ├── README.md              # (This file)
    └── SQL/
        └── schema.sql         # SQL dump: tables, sequences, sample data for locomotives & functions

-   **Docker-compose.yaml**
    -   Imposta due servizi nominati:
        1.  **db**(`postgres:14-alpine`)
        2.  **Postgrest**(`locoapi`)
    -   Espone PostgreSQL sulla porta**5432**e Postgrest sul porto**3000**.
    -   IL`db`Il servizio scarica automaticamente ed esegue lo schema da`SQL/schema.sql`(via a`curl … | psql`comando), creazione di tutte le tabelle e caricando i dati di locomotive iniziali.
    -   IL`postgrest`Il servizio si collega al file`db`usando le stesse credenziali.

-   **SQL/Schema.Sql**
    -   Un dump PostgreSQL completo contenente:
        -   Ruoli (ad es.`web_anon`)
        -   Schemi, tavoli (`locos`,`functions`, ecc.), sequenze e vincoli
        -   `COPY`Comandi che precarica il database con locomotive di esempio e dati "funzioni" associati.

-   **LICENZA**
    -   La mia licenza

* * *

## Installazione sul sistema operativo Raspberry Pi

Seguire questi passaggi per installare Docker, configurare il database Locomotive RailPilot e avviare i servizi sul Raspberry Pi.

### 1. Installa Docker & Docker Compose

Seguire[questa guida](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

### 2. Clona questo repository

1.  Passa in una directory in cui si desidera archiviare il codice, ad esempio la cartella di casa:
    ```bash
    cd ~
    ```

2.  Clona il repository GitHub:
    ```bash
    git clone https://github.com/RealPCBUILD3R/RailPilot-locodb.git
    ```
    > Se hai scaricato la zip con altri mezzi, decomprimerlo dove vuoi e`cd`in quella cartella.

3.  Spostati nella directory del progetto:
    ```bash
    cd RailPilot-locodb
    ```

* * *

### 3. Configurare variabili / password di ambiente

IL`docker-compose.yaml`Il file utilizza segnaposto per le credenziali di PostgreSQL. È necessario sostituire`<Your-Password>`In**due**luoghi:

1.  **Postgres_Password**(per il`db`servizio)

2.  **PGRST_DB_URI**(per il`postgrest`Servizio, nella stringa di connessione)

3.  Aprire`docker-compose.yaml`In un editor di testo (ad es.,`nano`,`vi`):
    ```bash
    nano docker-compose.yaml
    ```

4.  Individua queste sezioni:

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

5.  Sostituire ogni istanza di`<Your-Password>`con a**Password forte**di tua scelta (ad es.`s3cur3Pa$$w0rd`). Per esempio:

    ```yaml
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: s3cur3Pa$$w0rd
      POSTGRES_DB: locodb
    ```

    E:

    ```yaml
    environment:
      PGRST_DB_URI: postgres://admin:s3cur3Pa$$w0rd@db:5432/locodb
      PGRST_DB_SCHEMA: public
      PGRST_DB_ANON_ROLE: web_anon
    ```

6.  Salva ed esci:

    -   In**nano**: premere`CTRL+O`(scrivere),`Enter`, Poi`CTRL+X`(Uscita).
    -   In**tu/vim**: premere`Esc`, tipo`:wq`, quindi premere`Enter`.

> **Mancia:**
>
> -   Fare**non**Impegna o condividi la tua password pubblicamente.

* * *

### 4. Servizi di lancio con Docker Compose

1.  Dall'interno del`RailPilot-locodb`directory, esegui:

    ```bash
    docker-compose up -d
    ```

    -   IL`-d`Flag significa "distaccato" (eseguire in background).
    -   Docker tirerà le immagini necessarie (`postgres:14-alpine`E`postgrest/postgrest:latest`), crea un volume Docker denominato`db_data`e inizia due contenitori chiamati`locodb`E`locoapi`.

2.  Per verificare che i contenitori siano in esecuzione:

    ```bash
    docker ps
    ```

    Dovresti vedere entrambi`locodb`(PostgreSQL) e`locoapi`(Postgrest) Running.

3.  **Inizializzazione in prima volta**

    -   IL`locodb`(PostgreSQL) Container esegue un solo colpo`command:`che installa`curl`e poi prendono`SQL/schema.sql`Dal repository github`main`ramo. Si tuba direttamente in cui SQL`psql`, Quale:
        1.  Crea tutte le tabelle, sequenze, vincoli, ecc.
        2.  Copie i dati di esempio nel file`locos`E`functions`tavoli.

    -   Dopo questo è completato, il`locodb`Il contenitore continua a funzionare con i dati caricati.

4.  **Verificare l'inizializzazione del database**(opzionale)
    1.  Apri un browser web
    2.  Vai a http&#x3A; //<IP-Address of your Device running locodb>: 3000/locos
    3.  Dovresti vedere qualcosa del genere: \[{"id": 46, "nome": "Salzburger", "indirizzo": 52, "icona": "\\Kassaka, 4414151536 Bakha 4 fratello ...

5.  **API Postgrest**
    -   IL`locoapi`(Postgrest) Container aspetterà`locodb`essere disponibile.
    -   Una volta che può connettersi, espone un'API JSON RESTful a**Porta 3000**. Per impostazione predefinita, il ruolo anonimo`web_anon`può interrogare il`public`schema.

* * *

## Come funziona

1.  **Database PostgreSQL (`locodb`)**
    -   Corre su`postgres:14-alpine`.
    -   Variabili di ambiente:
        -   `POSTGRES_USER=admin`
        -   `POSTGRES_PASSWORD=<your_password>`
        -   `POSTGRES_DB=locodb`
    -   All'avvio, installa`curl`e corre:
        ```bash
        curl -H 'Cache-Control: no-cache' -fsSL        https://raw.githubusercontent.com/RealPCBUILD3R/RailPilot-locodb/refs/heads/main/SQL/schema.sql        | psql -U admin -d locodb
        ```
    -   Ciò inizializza lo schema del database (tabelle, sequenze, vincoli) e carica i dati di locomotiva di esempio.

2.  **Servizio postgrest (`locoapi`)**
    -   Corse`postgrest/postgrest:latest`.
    -   Si collega a`locodb`database utilizzando il`admin`utente e password che hai configurato.
    -   Serve un'API JSON RESTful on Port**3000**. Per esempio:
        -   `GET http://<IP-Address of your Device running locodb>:3000/locos`Restituisce tutte le locomotive.
        -   `GET http://<IP-Address of your Device running locodb>:3000/functions?loco_address=eq.52`Restituisce tutte le funzioni per la locomotiva con indirizzo 52.
    -   IL`PGRST_DB_ANON_ROLE=web_anon`L'ambiente variabile significa che chiunque sulla rete locale può interrogare l'API senza ulteriore autenticazione.

3.  **Volume`db_data`**
    -   Un volume Docker che persiste sui dati (directory dati di PostgreSQL) attraverso il riavvio o le ricreazioni del contenitore.
    -   Se hai mai bisogno di pulire completamente i dati, puoi rimuovere il volume:
        ```bash
        docker-compose down -v
        ```
        > **Avvertimento:**Questo eliminerà tutti i dati della locomotiva.

* * *

## Utilizzo

1.  **Configurazione dell'app RailPilot**
    -   Nella tua applicazione RailPilot (in esecuzione su iPhone, iPad o un altro dispositivo), impostare l'indirizzo IP del database Locomotive sull'indirizzo del dispositivo in esecuzione LOCODB, imposta la porta su**3000**e il protocollo a http&#x3A; //.
    -   L'app può quindi recuperare informazioni sulla locomotiva (ad es. Nomi, indirizzi, icone) e definizioni di funzione direttamente da questa API locale.

2.  **Gestire i contenitori**
    -   **Fermare**Entrambi i servizi:
        ```bash
        docker-compose down
        ```
    -   **Inizio**di nuovo (distaccato):
        ```bash
        docker-compose up -d
        ```
    -   **Visualizza i registri del contenitore**(ad esempio, per il debug di problemi di inizializzazione):
        ```bash
        docker-compose logs -f
        ```

* * *

## Considerazioni sulla sicurezza

-   **Non esporre a Internet pubblico**
    -   Il database non ha un'autenticazione utente/password integrata oltre l'utente "amministratore" di PostgreSQL.
    -   Postgrest per impostazione predefinita consente Anonimo (`web_anon`) accesso al file`public`schema.
    -   Se si esponi la porta 5432 o 3000 a Internet senza firewall o una VPN, chiunque potrebbe leggere, modificare, eliminare o persino caricare dati dannosi nel tuo database.
    -   **Raccomandazione:**Tieni il Raspberry Pi dietro un firewall di un router o su una VLAN privata. Se hai bisogno di accesso remoto, utilizzare una VPN.

-   **Modificare la password predefinita**
    -   Scegli una password forte e unica in`docker-compose.yaml`per`POSTGRES_PASSWORD`.

* * *

## Panoramica dello schema del database

IL`schema.sql`Il file definisce:

1.  **Ruoli e schemi**
    -   UN`web_anon`ruolo con i privilegi di selezione di base`public`.
    -   IL`public`Schema di proprietà del`admin`utente.

2.  **Tavoli**

    -   `public.locos`
        -   Colonne:`id`,`name`,`address`,`icon`(Binary Blob),`thumbnail`(Binary Blob),`maxspeed`,`speedstep`,`fullname`,`length`,`weight`,`owner`,`dateadded`
        -   I vincoli garantiscono intervalli di indirizzo validi, impostazioni di velocità, ecc.
        -   Righe campione per varie locomotive (ad es. "Salzburger", ecc.).

    -   `public.functions`
        -   Colonne:`id`,`loco_address`,`number`,`shortname`,`longname`,`icontype`,`icon`,`type`(`switch`,`button`, O`timer`),`time`(per i timer)
        -   Definisce le funzioni (luci, suoni, ecc.) Per indirizzo locomotivo.

3.  **Sequenze**
    -   `public.locos_id_seq`E`public.functions_id_seq`per auto -incremento il`id`colonne.

4.  **Dati di esempio**
    -   Varie locomotive (ad esempio, con indirizzi 21, 42, 52, ecc.) E le loro funzioni predefinite.
    -   Questi dati vengono caricati tramite`COPY … FROM stdin`comandi incorporati`schema.sql`.

* * *

## Licenza

Questo progetto è open source ai sensi del**La mia licenza**. Vedere[LICENZA](LICENSE)per i dettagli.

* * *

> **Riepilogo**
>
> -   Clona il repository, modifica`docker-compose.yaml`per impostare una password forte.
> -   Installa Docker & Docker Composi su Raspberry Pi.
> -   Correre`docker-compose up -d`Per creare e avviare il database (`locodb`) e API (`locoapi`).
> -   Conservare i servizi dietro un firewall; Non esporre le porte 5432/3000 a Internet pubblico.
> -   Goditi un database offline locale per la tua configurazione RailPilot!
