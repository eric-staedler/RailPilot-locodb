# Base de données RailPilot Locomotive (LOCODB)

Ce référentiel fournit une base de données PostgreSQL et une API PostGrest pour l'application RailPilot. La base de données stocke des informations sur les locomotives (par exemple, noms, adresses, icônes, paramètres de vitesse, etc.), que l'application RailPilot doit fonctionner. Le schéma de base de données est défini dans`SQL/schema.sql`et un fichier docker compose (`docker-compose.yaml`) automatise la construction et l'exécution à la fois de la base de données PostgreSQL et du service PostGrest.

> **Avertissement:**La base de données est**pas**Protégé par mot de passe au-delà des informations d'identification PostgreSQL de base et fait**pas**appliquer l'authentification des utilisateurs. Faire**pas**Exposez directement cette base de données ou API à l'Internet public. Gardez-le sur un réseau local de confiance ou derrière un pare-feu.

* * *

## Table des matières

1.  [Condition préalable](#prerequisites)
2.  [Contenu du référentiel](#repository-contents)
3.  [Installation sur Raspberry Pi OS](#installation-on-raspberry-pi-os)
    -   [1. Installer Docker & Docker Compose](#1-install-docker--docker-compose)
    -   [2. Clone ce référentiel](#2-clone-this-repository)
    -   [3. Configurer les variables d'environnement / mot de passe](#3-configure-environment-variables--password)
    -   [4. Services de lancement avec Docker Compose](#4-launch-services-with-docker-compose)
4.  [Comment ça marche](#how-it-works)
5.  [Usage](#usage)
6.  [Considérations de sécurité](#security-considerations)
7.  [Présentation du schéma de base de données](#database-schema-overview)
8.  [Licence](#license)

* * *

## Condition préalable

-   Un Raspberry Pi Running**Raspberry Pi OS**(32 bits ou 64 bits; ce guide suppose un système d'exploitation basé sur Debian). Ou tout autre appareil exécutant un système d'exploitation basé à Debian.
-   Un compte utilisateur avec`sudo`privilèges.
-   Une connexion Internet fonctionnelle (pour extraire des images Docker et télécharger le schéma SQL lors de l'initialisation).
-   Au moins**4 Go**de stockage gratuit pour les images, volumes et données Docker.
-   Connaissance de base avec la ligne de commande / terminal.

* * *

## Contenu du référentiel

    RailPilot-locodb/
    ├── docker-compose.yaml    # Defines PostgreSQL (locodb) and PostgREST (locoapi) services
    ├── LICENSE                # MIT License
    ├── README.md              # (This file)
    └── SQL/
        └── schema.sql         # SQL dump: tables, sequences, sample data for locomotives & functions

-   **docker-compose.yaml**
    -   Mettez en place deux services nommés:
        1.  **db**(`postgres:14-alpine`)
        2.  **post-poitrine**(`locoapi`)
    -   Expose PostgreSQL sur le port**5432**et post-prest sur le port**3000**.
    -   Le`db`Service télécharge et exécute automatiquement le schéma à partir de`SQL/schema.sql`(via un`curl … | psql`Commande), créant toutes les tables et chargeant des données de locomotive initiales.
    -   Le`postgrest`le service se connecte au`db`en utilisant les mêmes informations d'identification.

-   **Sql / schema.sql**
    -   Un vidage postgresql complet contenant:
        -   Rôles (par exemple,`web_anon`)
        -   Schémas, tables (`locos`,`functions`, etc.), Séquences et contraintes
        -   `COPY`commandes qui préchargent la base de données avec des exemples de locomotives et des données «fonctions» associées.

-   **LICENCE**
    -   Ma licence

* * *

## Installation sur Raspberry Pi OS

Suivez ces étapes pour installer Docker, configurez la base de données de locomotive RailPilot et lancez les services sur votre Raspberry Pi.

### 1. Installer Docker & Docker Compose

Suivre[Ce guide](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

### 2. Clone ce référentiel

1.  Changez en répertoire où vous souhaitez stocker le code, par exemple votre dossier à domicile:
    ```bash
    cd ~
    ```

2.  Clone le référentiel GitHub:
    ```bash
    git clone https://github.com/RealPCBUILD3R/RailPilot-locodb.git
    ```
    > Si vous avez téléchargé le zip par d'autres moyens, décompressez-le où vous le souhaitez et`cd`dans ce dossier.

3.  Déménager dans le répertoire du projet:
    ```bash
    cd RailPilot-locodb
    ```

* * *

### 3. Configurer les variables d'environnement / mot de passe

Le`docker-compose.yaml`Le fichier utilise des espaces réservés pour les informations d'identification PostgreSQL. Vous devez remplacer`<Your-Password>`dans**deux**lieux:

1.  **Postgres_password**(pour le`db`service)

2.  **PGRST_DB_URI**(pour le`postgrest`service, dans la chaîne de connexion)

3.  Ouvrir`docker-compose.yaml`Dans un éditeur de texte (par exemple,`nano`,`vi`):
    ```bash
    nano docker-compose.yaml
    ```

4.  Localisez ces sections:

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

5.  Remplacer chaque instance de`<Your-Password>`avec un**mot de passe fort**de votre choix (par exemple,`s3cur3Pa$$w0rd`). Par exemple:

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

6.  Enregistrer et sortir:

    -   Dans**nano**: presse`CTRL+O`(écrire),`Enter`, alors`CTRL+X`(sortie).
    -   Dans**vous / vim**: presse`Esc`, taper`:wq`, puis appuyez sur`Enter`.

> **Conseil:**
>
> -   Faire**pas**Commissez ou partagez votre mot de passe publiquement.

* * *

### 4. Services de lancement avec Docker Compose

1.  De l'intérieur du`RailPilot-locodb`répertoire, exécuter:

    ```bash
    docker-compose up -d
    ```

    -   Le`-d`Le drapeau signifie «détaché» (exécuter en arrière-plan).
    -   Docker tirera les images nécessaires (`postgres:14-alpine`et`postgrest/postgrest:latest`), créer un volume docker nommé`db_data`et démarrer deux conteneurs nommés`locodb`et`locoapi`.

2.  Pour vérifier que les conteneurs fonctionnent:

    ```bash
    docker ps
    ```

    Tu devrais voir les deux`locodb`(Postgresql) et`locoapi`(PostGrest) Running.

3.  **Initialisation de premier temps**

    1.  Crée toutes les tables, séquences, contraintes, etc.
    2.  Copie des exemples de données dans le`locos`et`functions`tableaux.

    -   Une fois cette fin terminée, le`locodb`Le conteneur continue de s'exécuter avec les données chargées.

4.  **Vérifiez l'initialisation de la base de données**(facultatif)
    1.  Ouvrir un navigateur Web
    2.  Aller sur http&#x3A; //<IP-Address of your Device running locodb>: 3000 / Locos
    3.  Vous devriez voir quelque chose comme ceci: \[{"id": 46, "nom": "Salzburger", "Adresse": 52, "icône": "\\Kassaka, 4414151536 Bakha 4 Frère ...

5.  **API PostGrest**
    -   Le`locoapi`(PostGrest) Le conteneur attendra`locodb`être disponible.
    -   Une fois qu'il peut se connecter, il expose une API JSON reposante à**Port 3000**. Par défaut, le rôle anonyme`web_anon`peut interroger le`public`schéma.

* * *

## Comment ça marche

1.  **Base de données PostgreSQL (`locodb`)**
    -   Se déroule`postgres:14-alpine`.
    -   Variables d'environnement:
        -   `POSTGRES_USER=admin`
        -   `POSTGRES_PASSWORD=<your_password>`
        -   `POSTGRES_DB=locodb`

2.  **Service post-poitrel (`locoapi`)**
    -   Courses`postgrest/postgrest:latest`.
    -   Se connecte au`locodb`base de données à l'aide du`admin`utilisateur et mot de passe que vous avez configuré.
    -   Sert une API JSON reposante sur le port**3000**. Par exemple:
        -   `GET http://<IP-Address of your Device running locodb>:3000/locos`Renvoie toutes les locomotives.
        -   `GET http://<IP-Address of your Device running locodb>:3000/functions?loco_address=eq.52`Renvoie toutes les fonctions pour la locomotive avec l'adresse 52.
    -   Le`PGRST_DB_ANON_ROLE=web_anon`La variable d'environnement signifie que toute personne sur votre réseau local peut interroger l'API sans authentification supplémentaire.

3.  **Volume`db_data`**
    -   Un volume Docker qui persiste les données (répertoire de données de PostgreSQL) à travers les redémarrages ou les loisirs.
    -   Si jamais vous devez effacer complètement les données, vous pouvez supprimer le volume:
        ```bash
        docker-compose down -v
        ```
        > **Avertissement:**Cela supprimera toutes les données de locomotive.

* * *

## Usage

1.  **Configuration de l'application RailPilot**
    -   Dans votre application RailPilot (en cours d'exécution sur votre iPhone, iPad ou un autre appareil), définissez l'adresse IP de la base de données de locomotive sur l'adresse de votre appareil exécutant LocOdB, définissez le port sur**3000**et le protocole de http&#x3A; //.
    -   L'application peut ensuite récupérer les informations de locomotive (par exemple, les noms, les adresses, les icônes) et les définitions de fonction directement à partir de cette API locale.

2.  **Gérer les conteneurs**
    -   **Arrêt**Les deux services:
        ```bash
        docker-compose down
        ```
    -   **Commencer**Encore une fois (détaché):
        ```bash
        docker-compose up -d
        ```
    -   **Afficher les journaux des conteneurs**(par exemple, pour déboguer les problèmes d'initialisation):
        ```bash
        docker-compose logs -f
        ```

* * *

## Considérations de sécurité

-   **N'exposez pas à l'Internet public**
    -   La base de données n'a pas d'authentification utilisateur / mot de passe intégrée au-delà de l'utilisateur «admin» de PostgreSQL.
    -   PostGrest par défaut permet Anonymous (`web_anon`) accès au`public`schéma.
    -   Si vous exposez le port 5432 ou 3000 sur Internet sans pare-feu ou VPN, n'importe qui pourrait lire, modifier, supprimer ou même télécharger des données nocives dans votre base de données.
    -   **Recommandation:**Gardez le Raspberry Pi derrière le pare-feu d'un routeur ou sur un VLAN privé. Si vous avez besoin d'un accès à distance, utilisez un VPN.

-   **Modifier le mot de passe par défaut**
    -   Choisissez un mot de passe solide et unique dans`docker-compose.yaml`pour`POSTGRES_PASSWORD`.

* * *

## Présentation du schéma de base de données

Le`schema.sql`Le fichier définit:

1.  **Rôles et schémas**
    -   UN`web_anon`Rôle avec des privilèges de base de base sur`public`.
    -   Le`public`schéma appartenant à la`admin`utilisateur.

2.  **Tables**

    -   `public.locos`
        -   Colonnes:`id`,`name`,`address`,`icon`(blob binaire),`thumbnail`(blob binaire),`maxspeed`,`speedstep`,`fullname`,`length`,`weight`,`owner`,`dateadded`
        -   Les contraintes garantissent des plages d'adresses valides, des paramètres de vitesse, etc.
        -   Exemples de rangées pour diverses locomotives (par exemple, «Salzburger», etc.).

    -   `public.functions`
        -   Colonnes:`id`,`loco_address`,`number`,`shortname`,`longname`,`icontype`,`icon`,`type`(`switch`,`button`, ou`timer`),`time`(pour les minuteries)
        -   Définit les fonctions (lumières, sons, etc.) par adresse de locomotive.

3.  **Séquences**
    -   `public.locos_id_seq`et`public.functions_id_seq`à l'inscription automatique`id`colonnes.

4.  **Échantillons de données**
    -   Diverses locomotives (par exemple, avec des adresses 21, 42, 52, etc.) et leurs fonctions par défaut.
    -   Ces données sont chargées via`COPY … FROM stdin`commandes intégrées dans`schema.sql`.

* * *

## Licence

Ce projet est open source sous le**Ma licence**. Voir[LICENCE](LICENSE)pour plus de détails.

* * *

> **Résumé**
>
> -   Cloner le repo, modifier`docker-compose.yaml`Pour définir un mot de passe fort.
> -   Installez Docker & Docker Compose sur votre Raspberry Pi.
> -   Courir`docker-compose up -d`Pour construire et démarrer la base de données (`locodb`) et API (`locoapi`).
> -   Gardez les services derrière un pare-feu; n'exposez pas les ports 5432/3000 à Internet public.
> -   Profitez d'une base de données hors ligne locale pour votre configuration Railpilot!
