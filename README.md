# RailPilot Locomotive Database (locodb)

This repository provides a PostgreSQL database and a PostgREST API for the RailPilot application. The database stores information about locomotives (e.g., names, addresses, icons, speed settings, etc.), which the RailPilot app needs to operate. The database schema is defined in `SQL/schema.sql`, and a Docker Compose file (`docker-compose.yaml`) automates building and running both the PostgreSQL database and the PostgREST service.

> **Warning:** The database is **not** password-protected beyond the basic PostgreSQL credentials and does **not** enforce user authentication. Do **not** expose this database or API directly to the public Internet. Keep it on a trusted local network or behind a firewall.

---

## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Repository Contents](#repository-contents)  
3. [Installation on Raspberry Pi OS](#installation-on-raspberry-pi-os)  
   - [1. Install Docker & Docker Compose](#1-install-docker--docker-compose)  
   - [2. Clone this Repository](#2-clone-this-repository)  
   - [3. Configure Environment Variables / Password](#3-configure-environment-variables--password)  
   - [4. Launch Services with Docker Compose](#4-launch-services-with-docker-compose)  
4. [How It Works](#how-it-works)  
5. [Usage](#usage)  
6. [Security Considerations](#security-considerations)  
7. [Database Schema Overview](#database-schema-overview)  
8. [License](#license)

---

## Prerequisites

- A Raspberry Pi running **Raspberry Pi OS** (32 bit or 64 bit; this guide assumes a Debian-based OS). Or any other device running a Debian-based OS. 
- A user account with `sudo` privileges.  
- A working internet connection (to pull Docker images and download the SQL schema during initialization).  
- At least **4 GB** of free storage for Docker images, volumes, and data.  
- Basic familiarity with the command line / terminal.

---

## Repository Contents

```
RailPilot-locodb/
├── docker-compose.yaml    # Defines PostgreSQL (locodb) and PostgREST (locoapi) services
├── LICENSE                # MIT License
├── README.md              # (This file)
└── SQL/
    └── schema.sql         # SQL dump: tables, sequences, sample data for locomotives & functions
```

- **docker-compose.yaml**  
  - Sets up two named services:  
    1. **db** (`postgres:14-alpine`)  
    2. **postgrest** (`locoapi`)  
  - Exposes PostgreSQL on port **5432** and PostgREST on port **3000**.  
  - The `db` service automatically downloads and runs the schema from `SQL/schema.sql` (via a `curl … | psql` command), creating all tables and loading initial locomotive data.  
  - The `postgrest` service connects to the `db` using the same credentials.

- **SQL/schema.sql**  
  - A full PostgreSQL dump containing:  
    - Roles (e.g., `web_anon`)  
    - Schemas, tables (`locos`, `functions`, etc.), sequences, and constraints  
    - `COPY` commands that preload the database with sample locomotives and associated “functions” data.

- **LICENSE**  
  - MIT License

---

## Installation on Raspberry Pi OS

Follow these steps to install Docker, configure the RailPilot locomotive database, and launch the services on your Raspberry Pi.

### 1. Install Docker & Docker Compose
Follow [this guide](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

### 2. Clone this Repository

1. Change into a directory where you want to store the code, for example your home folder:
   ```bash
   cd ~
   ```

2. Clone the GitHub repository:
   ```bash
   git clone https://github.com/RealPCBUILD3R/RailPilot-locodb.git
   ```
   > If you downloaded the ZIP by other means, unzip it wherever you like and `cd` into that folder.

3. Move into the project directory:
   ```bash
   cd RailPilot-locodb
   ```

---

### 3. Configure Environment Variables / Password

The `docker-compose.yaml` file uses placeholders for PostgreSQL credentials. You must replace `<Your-Password>` in **two** places:

1. **POSTGRES_PASSWORD** (for the `db` service)  
2. **PGRST_DB_URI** (for the `postgrest` service, in the connection string)

1. Open `docker-compose.yaml` in a text editor (e.g., `nano`, `vi`):
   ```bash
   nano docker-compose.yaml
   ```

2. Locate these sections:

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

3. Replace each instance of `<Your-Password>` with a **strong password** of your choosing (e.g., `s3cur3Pa$$w0rd`). For example:

   ```yaml
   environment:
     POSTGRES_USER: admin
     POSTGRES_PASSWORD: s3cur3Pa$$w0rd
     POSTGRES_DB: locodb
   ```

   And:

   ```yaml
   environment:
     PGRST_DB_URI: postgres://admin:s3cur3Pa$$w0rd@db:5432/locodb
     PGRST_DB_SCHEMA: public
     PGRST_DB_ANON_ROLE: web_anon
   ```

4. Save and exit:

   - In **nano**: press `CTRL+O` (write), `Enter`, then `CTRL+X` (exit).  
   - In **vi/vim**: press `Esc`, type `:wq`, then press `Enter`.

> **Tip:**  
> - Do **not** commit or share your password publicly.

---

### 4. Launch Services with Docker Compose

1. From inside the `RailPilot-locodb` directory, run:

   ```bash
   docker-compose up -d
   ```

   - The `-d` flag means “detached” (run in the background).  
   - Docker will pull the necessary images (`postgres:14-alpine` and `postgrest/postgrest:latest`), create a Docker volume named `db_data`, and start two containers named `locodb` and `locoapi`.

2. To check that containers are running:

   ```bash
   docker ps
   ```

   You should see both `locodb` (PostgreSQL) and `locoapi` (PostgREST) running.

3. **First‐Time Initialization**  
   - The `locodb` (PostgreSQL) container runs a one‐shot `command:` that installs `curl` and then fetches `SQL/schema.sql` from the GitHub repository’s `main` branch. It pipes that SQL directly into `psql`, which:
     1. Creates all tables, sequences, constraints, etc.  
     2. Copies sample data into the `locos` and `functions` tables.

   - After this completes, the `locodb` container continues running with the data loaded.

4. **Verify Database Initialization** (optional)  
   1. Open a web browser
   2. Go to http://<IP-Address of your Device running locodb>:3000/locos
   3. You should see something like this: [{"id":46,"name":"Salzburger","address":52,"icon":"\\x2f396a2f34414151536b5a4a5...

5. **PostgREST API**  
   - The `locoapi` (PostgREST) container will wait for `locodb` to be available.  
   - Once it can connect, it exposes a RESTful JSON API at **port 3000**. By default, the anonymous role `web_anon` can query the `public` schema.

---

## How It Works

1. **PostgreSQL Database (`locodb`)**  
   - Runs on `postgres:14-alpine`.  
   - Environment variables:  
     - `POSTGRES_USER=admin`  
     - `POSTGRES_PASSWORD=<your_password>`  
     - `POSTGRES_DB=locodb`  
   - On startup, installs `curl` and runs:
     ```bash
     curl -H 'Cache-Control: no-cache' -fsSL        https://raw.githubusercontent.com/RealPCBUILD3R/RailPilot-locodb/refs/heads/main/SQL/schema.sql        | psql -U admin -d locodb
     ```
   - This initializes the database schema (tables, sequences, constraints) and loads sample locomotive data.

2. **PostgREST Service (`locoapi`)**  
   - Runs `postgrest/postgrest:latest`.  
   - Connects to the `locodb` database using the `admin` user and password you configured.  
   - Serves a RESTful JSON API on port **3000**. For example:  
     - `GET http://<IP-Address of your Device running locodb>:3000/locos` returns all locomotives.  
     - `GET http://<IP-Address of your Device running locodb>:3000/functions?loco_address=eq.52` returns all functions for the locomotive with address 52.  
   - The `PGRST_DB_ANON_ROLE=web_anon` environment variable means anyone on your local network can query the API without additional authentication.  

3. **Volume `db_data`**  
   - A Docker volume that persists data (PostgreSQL’s data directory) across container restarts or recreations.  
   - If you ever need to wipe the data completely, you can remove the volume:  
     ```bash
     docker-compose down -v
     ```
     > **Warning:** This will delete all locomotive data.

---

## Usage

1. **RailPilot App Configuration**  
   - In your RailPilot application (running on your iPhone, iPad, or another device), set the IP address of the locomotive database to the address of your Device running locodb, set the port to **3000** and the protocol to http://. 
   - The app can then fetch locomotive information (e.g., names, addresses, icons) and function definitions directly from this local API.

2. **Managing the Containers**  
   - **Stop** both services:  
     ```bash
     docker-compose down
     ```
   - **Start** again (detached):  
     ```bash
     docker-compose up -d
     ```
   - **View container logs** (e.g., to debug initialization issues):  
     ```bash
     docker-compose logs -f
     ```

---

## Security Considerations

- **Do Not Expose to the Public Internet**  
  - The database has no built-in user/password authentication beyond the PostgreSQL “admin” user.  
  - PostgREST by default allows anonymous (`web_anon`) read access to the `public` schema.  
  - If you expose port 5432 or 3000 to the Internet without firewalling or a VPN, anyone could query or (depending on PostgREST permissions) modify the locomotive data.  
  - **Recommendation:** Keep the Raspberry Pi behind a router’s firewall or on a private VLAN. If you need remote access, use a VPN or SSH tunnel.

- **Change Default Password**  
  - Choose a strong, unique password in `docker-compose.yaml` for `POSTGRES_PASSWORD`.  
  - If you suspect the password is compromised, shut down the containers, change the password, and re‐initialize the services.

- **Limit PostgREST Permissions**  
  - By default, the `web_anon` role can only `SELECT` from tables in `public`. If you wish to tighten permissions, modify the schema (in `SQL/schema.sql`) to grant more restrictive privileges or add authentication layers in front of PostgREST (e.g., Nginx + JWT).

---

## Database Schema Overview

The `schema.sql` file defines:

1. **Roles and Schemas**  
   - A `web_anon` role with basic SELECT privileges on `public`.
   - The `public` schema owned by the `admin` user.

2. **Tables**  
   - `public.locos`  
     - Columns: `id`, `name`, `address`, `icon` (binary blob), `thumbnail` (binary blob), `maxspeed`, `speedstep`, `fullname`, `length`, `weight`, `owner`, `dateadded`  
     - Constraints ensure valid address ranges, speed settings, etc.  
     - Sample rows for various locomotives (e.g., “Salzburger”, etc.).

   - `public.functions`  
     - Columns: `id`, `loco_address`, `number`, `shortname`, `longname`, `icontype`, `icon`, `type` (`switch`, `button`, or `timer`), `time` (for timers)  
     - Defines functions (lights, sounds, etc.) per locomotive address.

3. **Sequences**  
   - `public.locos_id_seq` and `public.functions_id_seq` to auto‐increment the `id` columns.

4. **Sample Data**  
   - Various locomotives (e.g., with addresses 21, 42, 52, etc.) and their default functions.  
   - This data is loaded via `COPY … FROM stdin` commands embedded in `schema.sql`.

---

## License

This project is open source under the **MIT License**. See [LICENSE](LICENSE) for details.

```
MIT License

Copyright (c) 2025 PCBUILD3R

Permission is hereby granted, free of charge, to any person obtaining a copy
… [full text in LICENSE file] …
```

---

> **Summary**  
> - Clone the repo, edit `docker-compose.yaml` to set a strong password.  
> - Install Docker & Docker Compose on your Raspberry Pi.  
> - Run `docker-compose up -d` to build and start the database (`locodb`) and API (`locoapi`).  
> - Keep services behind a firewall; do not expose ports 5432/3000 to the public Internet.  
> - Point your RailPilot app to `http://<raspberrypi-ip>:3000` to read locomotive data.  
> - Enjoy a local, offline database for your RailPilot setup!
```
