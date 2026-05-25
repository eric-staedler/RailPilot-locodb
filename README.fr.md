# Base de données des locomotives RailPilot (locodb)

Ce référentiel fournit une base de données PostgreSQL et une API PostgREST pour l'application RailPilot. La base de données stocke des informations sur les locomotives (par exemple, noms, adresses, icônes, paramètres de vitesse, etc.) dont l'application RailPilot a besoin pour fonctionner. Le schéma de la base de données est défini dans`SQL/schema.sql`, et un fichier Docker Compose (`docker-compose.yaml`) automatise la création et l'exécution de la base de données PostgreSQL et du service PostgREST.

> **Avertissement:**La base de données est**pas**protégé par mot de passe au-delà des informations d'identification de base de PostgreSQL et ne**pas**appliquer l'authentification des utilisateurs. Faire**pas**exposer cette base de données ou API directement à l’Internet public. Conservez-le sur un réseau local de confiance ou derrière un pare-feu.

* * *

## Table des matières

1.  [Conditions préalables](#prerequisites)
2.  [Contenu du référentiel](#repository-contents)
3.  [Installation sur le système d'exploitation Raspberry Pi](#installation-on-raspberry-pi-os)
    -   [1. Installez Docker et Docker Compose](#1-install-docker--docker-compose)
    -   [2. Clonez ce référentiel](#2-clone-this-repository)
    -   [3. Configurer les variables d'environnement/mot de passe](#3-configure-environment-variables--password)
    -   [4. Lancer les services avec Docker Compose](#4-launch-services-with-docker-compose)
4.  [Comment ça marche](#how-it-works)
5.  [Usage](#usage)
6.  [Considérations de sécurité](#security-considerations)
7.  [Présentation du schéma de base de données](#database-schema-overview)
8.  [Licence](#license)

* * *

## Conditions préalables

-   Un Raspberry Pi en marche**Système d'exploitation Raspberry Pi**(32 bits ou 64 bits ; ce guide suppose un système d'exploitation basé sur Debian). Ou tout autre appareil exécutant un système d’exploitation basé sur Debian.
-   Un compte utilisateur avec`sudo`privilèges.
-   Une connexion Internet fonctionnelle (pour extraire les images Docker et télécharger le schéma SQL lors de l'initialisation).
-   Au moins**4 Go**de stockage gratuit pour les images, les volumes et les données Docker.
-   Familiarité de base avec la ligne de commande/le terminal.

* * *

## Contenu du référentiel

    RailPilot-locodb/
    ├── docker-compose.yaml    # Defines PostgreSQL (locodb) and PostgREST (locoapi) services
    ├── LICENSE                # MIT License
    ├── README.md              # (This file)
    └── SQL/
        └── schema.sql         # SQL dump: tables, sequences, sample data for locomotives & functions

-   **docker-compose.yaml**
    -   Configure deux services nommés :
        1.  **base de données**(`postgres:14-alpine`)
        2.  **post-greffe**(`locoapi`)
    -   Expose PostgreSQL sur le port**5432**et PostgREST sur le port**3000**.
    -   Le`db`le service télécharge et exécute automatiquement le schéma à partir de`SQL/schema.sql`(via un`curl … | psql`commande), créant toutes les tables et chargeant les données initiales de la locomotive.
    -   Le`postgrest`le service se connecte au`db`en utilisant les mêmes informations d'identification.

-   **SQL/schéma.sql**
    -   Un dump PostgreSQL complet contenant :
        -   Rôles (par ex.`web_anon`)
        -   Schémas, tableaux (`locos`,`functions`, etc.), les séquences et les contraintes
        -   `COPY`commandes qui préchargent la base de données avec des exemples de locomotives et les données de « fonctions » associées.

-   **LICENCE**
    -   MA Licence

* * *

## Installation sur le système d'exploitation Raspberry Pi

Suivez ces étapes pour installer Docker, configurer la base de données des locomotives RailPilot et lancer les services sur votre Raspberry Pi.

### 1. Installez Docker et Docker Compose

Suivre[ce guide](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

### 2. Clonez ce référentiel

1.  Accédez à un répertoire dans lequel vous souhaitez stocker le code, par exemple votre dossier personnel :
    ```bash
    cd ~
    ```

2.  Clonez le dépôt GitHub :
    ```bash
    git clone https://github.com/eric-staedler/RailPilot-locodb.git
    ```
    > Si vous avez téléchargé le ZIP par d'autres moyens, décompressez-le où vous le souhaitez et`cd`dans ce dossier.

3.  Accédez au répertoire du projet :
    ```bash
    cd RailPilot-locodb
    ```

* * *

### 3. Configurer les variables d'environnement/mot de passe

Le`docker-compose.yaml`Le fichier utilise des espaces réservés pour les informations d'identification PostgreSQL. Vous devez remplacer`<Your-Password>`dans**deux**lieux:

1.  **POSTGRES_PASSWORD**(pour le`db`service)

2.  **PGRST_DB_URI**(pour le`postgrest`service, dans la chaîne de connexion)

3.  Ouvrir`docker-compose.yaml`dans un éditeur de texte (par exemple,`nano`,`vi`):
    ```bash
    nano docker-compose.yaml
    ```

4.  Localisez ces sections :

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

5.  Remplacez chaque instance de`<Your-Password>`avec un**mot de passe fort**de votre choix (par exemple,`s3cur3Pa$$w0rd`). Par exemple:

    ```yaml
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: s3cur3Pa$$w0rd
      POSTGRES_DB: locodb
    ```

    Et:

    ```yaml
    environment:
      PGRST_DB_URI: postgres://admin:s3cur3Pa$$w0rd@db:5432/locodb
      PGRST_DB_SCHEMA: public
      PGRST_DB_ANON_ROLE: web_anon
    ```

6.  Enregistrez et quittez :

    -   Dans**nano**: presse`CTRL+O`(écrire),`Enter`, alors`CTRL+X`(sortie).
    -   Dans**toi**: presse`Esc`, taper`:wq`, puis appuyez sur`Enter`.

> **Conseil:**
>
> -   Faire**pas**validez ou partagez votre mot de passe publiquement.

* * *

### 4. Lancer les services avec Docker Compose

1.  De l'intérieur du`RailPilot-locodb`répertoire, exécutez :

    ```bash
    docker-compose up -d
    ```

    -   Le`-d`flag signifie « détaché » (exécuté en arrière-plan).
    -   Docker extraira les images nécessaires (`postgres:14-alpine`et`postgrest/postgrest:latest`), créez un volume Docker nommé`db_data`, et démarrez deux conteneurs nommés`locodb`et`locoapi`.

2.  Pour vérifier que les conteneurs sont en cours d'exécution :

    ```bash
    docker ps
    ```

    Tu devrais voir les deux`locodb`(PostgreSQL) et`locoapi`(PostgREST) ​​en cours d'exécution.

3.  **Première initialisation**

    1.  Crée toutes les tables, séquences, contraintes, etc.
    2.  Copie les exemples de données dans le`locos`et`functions`tableaux.

    -   Une fois cette opération terminée, le`locodb`Le conteneur continue de s'exécuter avec les données chargées.

4.  **Vérifier l'initialisation de la base de données**(facultatif)
    1.  Ouvrez un navigateur Web
    2.  Allez sur http&#x3A;//<IP-Address of your Device running locodb>:3000/loco
    3.  Vous devriez voir quelque chose comme ceci : \[{"id":46,"name":"Salzburger","address":52,"icon":"\\4414151536bkha4kh...

5.  **API PostgREST**
    -   Le`locoapi`Le conteneur (PostgREST) ​​attendra`locodb`être disponible.
    -   Une fois qu'il peut se connecter, il expose une API RESTful JSON à**port 3000**. Par défaut, le rôle anonyme`web_anon`peut interroger le`public`schéma.

* * *

## Comment ça marche

1.  **Base de données PostgreSQL (`locodb`)**
    -   Fonctionne sur`postgres:14-alpine`.
    -   Variables d'environnement :
        -   `POSTGRES_USER=admin`
        -   `POSTGRES_PASSWORD=<your_password>`
        -   `POSTGRES_DB=locodb`

2.  **Service PostgREST (`locoapi`)**
    -   Fonctionne`postgrest/postgrest:latest`.
    -   Se connecte au`locodb`base de données utilisant le`admin`utilisateur et mot de passe que vous avez configurés.
    -   Sert une API RESTful JSON sur le port**3000**. Par exemple:
        -   `GET http://<IP-Address of your Device running locodb>:3000/locos`renvoie toutes les locomotives.
        -   `GET http://<IP-Address of your Device running locodb>:3000/functions?loco_address=eq.52`renvoie toutes les fonctions de la locomotive avec l'adresse 52.
    -   Le`PGRST_DB_ANON_ROLE=web_anon`La variable d'environnement signifie que n'importe qui sur votre réseau local peut interroger l'API sans authentification supplémentaire.

3.  **Volume`db_data`**
    -   Un volume Docker qui conserve les données (le répertoire de données de PostgreSQL) lors des redémarrages ou des recréations du conteneur.
    -   Si jamais vous devez effacer complètement les données, vous pouvez supprimer le volume :
        ```bash
        docker-compose down -v
        ```
        > **Avertissement:**Cela supprimera toutes les données de la locomotive.

* * *

## Usage

1.  **Configuration de l'application RailPilot**
    -   In your RailPilot application (running on your iPhone, iPad, or another device), set the IP address of the locomotive database to the address of your Device running locodb, set the port to **3000**et le protocole vers http&#x3A;//.
    -   L'application peut ensuite récupérer des informations sur les locomotives (par exemple, noms, adresses, icônes) et des définitions de fonctions directement à partir de cette API locale.

2.  **Gestion des conteneurs**
    -   **Arrêt**les deux services :
        ```bash
        docker-compose down
        ```
    -   **Commencer**encore (détaché):
        ```bash
        docker-compose up -d
        ```
    -   **Afficher les journaux du conteneur**(par exemple, pour déboguer les problèmes d'initialisation) :
        ```bash
        docker-compose logs -f
        ```

* * *

## Considérations de sécurité

-   **Ne pas exposer à l'Internet public**
    -   La base de données n'a pas d'authentification utilisateur/mot de passe intégrée au-delà de l'utilisateur « administrateur » de PostgreSQL.
    -   PostgREST autorise par défaut les anonymes (`web_anon`) accès au`public`schéma.
    -   Si vous exposez le port 5432 ou 3000 à Internet sans pare-feu ni VPN, n'importe qui pourrait lire, modifier, supprimer ou même télécharger des données nuisibles dans votre base de données.
    -   **Recommandation:**Gardez le Raspberry Pi derrière le pare-feu d'un routeur ou sur un VLAN privé. Si vous avez besoin d'un accès à distance, utilisez un VPN.

-   **Changer le mot de passe par défaut**
    -   Choisissez un mot de passe fort et unique dans`docker-compose.yaml`pour`POSTGRES_PASSWORD`.

* * *

## Présentation du schéma de base de données

Le`schema.sql`le fichier définit :

1.  **Rôles et schémas**
    -   UN`web_anon`rôle avec des privilèges SELECT de base sur`public`.
    -   Le`public`schéma appartenant au`admin`utilisateur.

2.  **Tableaux**

    -   `public.locos`
        -   Colonnes :`id`,`name`,`address`,`icon`(blob binaire),`thumbnail`(blob binaire),`maxspeed`,`speedstep`,`fullname`,`length`,`weight`,`owner`,`dateadded`
        -   Les contraintes garantissent des plages d'adresses valides, des paramètres de vitesse, etc.
        -   Exemples de lignes pour différentes locomotives (par ex. « Salzburger », etc.).

    -   `public.functions`
        -   Colonnes :`id`,`loco_address`,`number`,`shortname`,`longname`,`icontype`,`icon`,`type`(`switch`,`button`, ou`timer`),`time`(pour les minuteries)
        -   Définit les fonctions (lumières, sons, etc.) par adresse de locomotive.

3.  **Séquences**
    -   `public.locos_id_seq`et`public.functions_id_seq`pour incrémenter automatiquement le`id`colonnes.

4.  **Exemples de données**
    -   Diverses locomotives (par exemple avec les adresses 21, 42, 52, etc.) et leurs fonctions par défaut.
    -   Ces données sont chargées via`COPY … FROM stdin`commandes intégrées dans`schema.sql`.

* * *

## Licence

Ce projet est open source sous le**MA Licence**. Voir[LICENCE](LICENSE)pour plus de détails.

* * *

> **Résumé**
>
> -   Cloner le dépôt, modifier`docker-compose.yaml`pour définir un mot de passe fort.
> -   Installez Docker & Docker Compose sur votre Raspberry Pi.
> -   Courir`docker-compose up -d`pour construire et démarrer la base de données (`locodb`) et API (`locoapi`).
> -   Gardez les services derrière un pare-feu ; n'exposez pas les ports 5432/3000 à l'Internet public.
> -   Profitez d'une base de données locale et hors ligne pour votre configuration RailPilot !
