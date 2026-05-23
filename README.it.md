# Database delle locomotive RailPilot (locodb)

Questo repository fornisce un database PostgreSQL e un'API PostgREST per l'applicazione RailPilot. Il database memorizza le informazioni sulle locomotive (ad esempio nomi, indirizzi, icone, impostazioni di velocità, ecc.), di cui l'app RailPilot ha bisogno per funzionare. Lo schema del database è definito in`SQL/schema.sql`e un file Docker Compose (`docker-compose.yaml`) automatizza la creazione e l'esecuzione sia del database PostgreSQL che del servizio PostgREST.

> **Avvertimento:**La banca dati è**non**protetto da password oltre le credenziali PostgreSQL di base e lo fa**non**imporre l'autenticazione dell'utente. Fare**non**esporre questo database o API direttamente alla rete Internet pubblica. Conservalo su una rete locale affidabile o dietro un firewall.

* * *

## Sommario

1.  [Prerequisiti](#prerequisites)
2.  [Contenuti del deposito](#repository-contents)
3.  [Installazione su sistema operativo Raspberry Pi](#installation-on-raspberry-pi-os)
    -   [1. Installa Docker e Docker Compose](#1-install-docker--docker-compose)
    -   [2. Clona questo repository](#2-clone-this-repository)
    -   [3. Configurare variabili d'ambiente/password](#3-configure-environment-variables--password)
    -   [4. Avvia i servizi con Docker Compose](#4-launch-services-with-docker-compose)
4.  [Come funziona](#how-it-works)
5.  [Utilizzo](#usage)
6.  [Considerazioni sulla sicurezza](#security-considerations)
7.  [Panoramica dello schema del database](#database-schema-overview)
8.  [Licenza](#license)

* * *

## Prerequisiti

-   Un Raspberry Pi in funzione**Sistema operativo Raspberry Pi**(32 bit o 64 bit; questa guida presuppone un sistema operativo basato su Debian). O qualsiasi altro dispositivo che esegue un sistema operativo basato su Debian.
-   Un account utente con`sudo`privilegi.
-   Una connessione Internet funzionante (per estrarre le immagini Docker e scaricare lo schema SQL durante l'inizializzazione).
-   Almeno**4GB**di spazio di archiviazione gratuito per immagini, volumi e dati Docker.
-   Familiarità di base con la riga di comando/terminale.

* * *

## Contenuti del deposito

    RailPilot-locodb/
    ├── docker-compose.yaml    # Defines PostgreSQL (locodb) and PostgREST (locoapi) services
    ├── LICENSE                # MIT License
    ├── README.md              # (This file)
    └── SQL/
        └── schema.sql         # SQL dump: tables, sequences, sample data for locomotives & functions

-   **docker-compose.yaml**
    -   Imposta due servizi denominati:
        1.  **db**(`postgres:14-alpine`)
        2.  **post-grest**(`locoapi`)
    -   Espone PostgreSQL sulla porta**5432**e PostgREST sulla porta**3000**.
    -   IL`db`Il servizio scarica ed esegue automaticamente lo schema da`SQL/schema.sql`(via a`curl … | psql`comando), creando tutte le tabelle e caricando i dati iniziali della locomotiva.
    -   IL`postgrest`il servizio si collega a`db`utilizzando le stesse credenziali.

-   **SQL/schema.sql**
    -   Un dump PostgreSQL completo contenente:
        -   Ruoli (ad es.`web_anon`)
        -   Schemi, tabelle (`locos`,`functions`, ecc.), sequenze e vincoli
        -   `COPY`comandi che precaricano il database con locomotive campione e dati di “funzioni” associati.

-   **LICENZA**
    -   LA MIA Licenza

* * *

## Installazione su sistema operativo Raspberry Pi

Segui questi passaggi per installare Docker, configurare il database delle locomotive RailPilot e avviare i servizi sul tuo Raspberry Pi.

### 1. Installa Docker e Docker Compose

Seguire[questa guida](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

### 2. Clona questo repository

1.  Passa alla directory in cui desideri memorizzare il codice, ad esempio la tua cartella home:
    ```bash
    cd ~
    ```

2.  Clona il repository GitHub:
    ```bash
    git clone https://github.com/RealPCBUILD3R/RailPilot-locodb.git
    ```
    > Se hai scaricato lo ZIP con altri mezzi, decomprimilo dove preferisci e`cd`in quella cartella.

3.  Passare alla directory del progetto:
    ```bash
    cd RailPilot-locodb
    ```

* * *

### 3. Configurare variabili d'ambiente/password

IL`docker-compose.yaml`il file utilizza segnaposto per le credenziali PostgreSQL. Devi sostituire`<Your-Password>`In**due**luoghi:

1.  **POSTGRES_PASSWORD**(per il`db`servizio)

2.  **PGRST_DB_URI**(per il`postgrest`servizio, nella stringa di connessione)

3.  Aprire`docker-compose.yaml`in un editor di testo (ad es.`nano`,`vi`):
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

5.  Sostituisci ogni istanza di`<Your-Password>`con a**password complessa**di tua scelta (ad es.`s3cur3Pa$$w0rd`). Per esempio:

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
    -   In**Voi**: premere`Esc`, tipo`:wq`, quindi premere`Enter`.

> **Mancia:**
>
> -   Fare**non**impegna o condividi pubblicamente la tua password.

* * *

### 4. Avvia i servizi con Docker Compose

1.  Dall'interno del`RailPilot-locodb`directory, esegui:

    ```bash
    docker-compose up -d
    ```

    -   IL`-d`flag significa “staccato” (esecuzione in background).
    -   Docker estrarrà le immagini necessarie (`postgres:14-alpine`E`postgrest/postgrest:latest`), creare un volume Docker denominato`db_data`e iniziare due contenitori denominati`locodb`E`locoapi`.

2.  Per verificare che i contenitori siano in esecuzione:

    ```bash
    docker ps
    ```

    Dovresti vederli entrambi`locodb`(PostgreSQL) e`locoapi`(PostgREST) ​​in esecuzione.

3.  **Prima inizializzazione**

    1.  Crea tutte le tabelle, sequenze, vincoli, ecc.
    2.  Copia i dati del campione nel file`locos`E`functions`tavoli.

    -   Al termine di questa operazione, il file`locodb`il contenitore continua a funzionare con i dati caricati.

4.  **Verificare l'inizializzazione del database**(opzionale)
    1.  Apri un browser web
    2.  Vai a http&#x3A;//<IP-Address of your Device running locodb>:3000/locomotive
    3.  Dovresti vedere qualcosa di simile a questo: \[{"id":46,"name":"Salzburger","address":52,"icon":"\\4414151536 bkha4kh...

5.  **API PostgREST**
    -   IL`locoapi`(PostgREST) ​​il contenitore attenderà`locodb`essere disponibile.
    -   Una volta che può connettersi, espone un'API JSON RESTful su**porto 3000**. Per impostazione predefinita, il ruolo anonimo`web_anon`può interrogare il`public`schema.

* * *

## Come funziona

1.  **Database PostgreSQL (`locodb`)**
    -   Continua a funzionare`postgres:14-alpine`.
    -   Variabili d'ambiente:
        -   `POSTGRES_USER=admin`
        -   `POSTGRES_PASSWORD=<your_password>`
        -   `POSTGRES_DB=locodb`

2.  **Servizio PostgREST (`locoapi`)**
    -   Corre`postgrest/postgrest:latest`.
    -   Si collega a`locodb`database utilizzando il file`admin`utente e password configurati.
    -   Fornisce un'API JSON RESTful sulla porta**3000**. Per esempio:
        -   `GET http://<IP-Address of your Device running locodb>:3000/locos`restituisce tutte le locomotive.
        -   `GET http://<IP-Address of your Device running locodb>:3000/functions?loco_address=eq.52`restituisce tutte le funzioni per la locomotiva con indirizzo 52.
    -   IL`PGRST_DB_ANON_ROLE=web_anon`la variabile di ambiente significa che chiunque sulla tua rete locale può interrogare l'API senza ulteriore autenticazione.

3.  **Volume`db_data`**
    -   Un volume Docker che rende persistenti i dati (directory dei dati di PostgreSQL) durante i riavvii o le ricreazioni del contenitore.
    -   Se mai avessi bisogno di cancellare completamente i dati, puoi rimuovere il volume:
        ```bash
        docker-compose down -v
        ```
        > **Avvertimento:**Ciò cancellerà tutti i dati della locomotiva.

* * *

## Utilizzo

1.  **Configurazione dell'app RailPilot**
    -   Nella tua applicazione RailPilot (in esecuzione sul tuo iPhone, iPad o un altro dispositivo), imposta l'indirizzo IP del database delle locomotive sull'indirizzo del tuo dispositivo su cui è in esecuzione locodb, imposta la porta su**3000**e il protocollo su http&#x3A;//.
    -   L'app può quindi recuperare informazioni sulla locomotiva (ad esempio nomi, indirizzi, icone) e definizioni di funzioni direttamente da questa API locale.

2.  **Gestire i contenitori**
    -   **Fermare**entrambi i servizi:
        ```bash
        docker-compose down
        ```
    -   **Inizio**ancora (staccato):
        ```bash
        docker-compose up -d
        ```
    -   **Visualizza i log del contenitore**(ad esempio, per eseguire il debug dei problemi di inizializzazione):
        ```bash
        docker-compose logs -f
        ```

* * *

## Considerazioni sulla sicurezza

-   **Non esporre all'Internet pubblica**
    -   Il database non dispone di autenticazione utente/password incorporata oltre all'utente "amministratore" di PostgreSQL.
    -   PostgREST per impostazione predefinita consente l'anonimo (`web_anon`) accesso al`public`schema.
    -   Se esponi la porta 5432 o 3000 a Internet senza firewall o VPN, chiunque potrebbe leggere, modificare, eliminare o persino caricare dati dannosi nel tuo database.
    -   **Raccomandazione:**Mantieni il Raspberry Pi dietro il firewall di un router o su una VLAN privata. Se hai bisogno di un accesso remoto, usa una VPN.

-   **Modifica password predefinita**
    -   Scegli una password complessa e univoca in`docker-compose.yaml`per`POSTGRES_PASSWORD`.

* * *

## Panoramica dello schema del database

IL`schema.sql`il file definisce:

1.  **Ruoli e schemi**
    -   UN`web_anon`ruolo con privilegi SELECT di base attivi`public`.
    -   IL`public`schema di proprietà di`admin`utente.

2.  **Tabelle**

    -   `public.locos`
        -   Colonne:`id`,`name`,`address`,`icon`(blob binario),`thumbnail`(blob binario),`maxspeed`,`speedstep`,`fullname`,`length`,`weight`,`owner`,`dateadded`
        -   I vincoli garantiscono intervalli di indirizzi validi, impostazioni di velocità, ecc.
        -   Righe campione per diverse locomotive (ad es. “Salzburger”, ecc.).

    -   `public.functions`
        -   Colonne:`id`,`loco_address`,`number`,`shortname`,`longname`,`icontype`,`icon`,`type`(`switch`,`button`, O`timer`),`time`(per i timer)
        -   Definisce le funzioni (luci, suoni, ecc.) per indirizzo della locomotiva.

3.  **Sequenze**
    -   `public.locos_id_seq`E`public.functions_id_seq`per incrementare automaticamente il`id`colonne.

4.  **Dati campione**
    -   Varie locomotive (ad esempio con indirizzi 21, 42, 52, ecc.) e le loro funzioni predefinite.
    -   Questi dati vengono caricati tramite`COPY … FROM stdin`comandi incorporati in`schema.sql`.

* * *

## Licenza

Questo progetto è open source sotto il**LA MIA Licenza**. Vedere[LICENZA](LICENSE)per i dettagli.

* * *

> **Riepilogo**
>
> -   Clona il repository, modifica`docker-compose.yaml`per impostare una password complessa.
> -   Installa Docker e Docker Compose sul tuo Raspberry Pi.
> -   Correre`docker-compose up -d`per creare e avviare il database (`locodb`) e API (`locoapi`).
> -   Mantenere i servizi dietro un firewall; non esporre le porte 5432/3000 all'Internet pubblica.
> -   Goditi un database locale offline per la configurazione di RailPilot!
