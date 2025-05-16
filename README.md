# Martini Runtime Docker Stack

This repository provides a Docker-based stack to run the Martini Runtime along with PostgreSQL and Cassandra. It includes automated volume mounting and configuration for external database connectivity to PostgreSQL and Cassandra.

## Stack Overview

This `docker-compose` implementation includes:

- **Martini Runtime** (via `lontiplatform/martini-server-runtime`)
- **PostgreSQL** as the relational database
- **Cassandra** as the Tracker database
- Automatic mounting of database configurations into the Martini container, while logs, data, and packages are persisted from the container to the host
- External database connectivity setup for Martini Tracker and PostgreSQL

## Project Structure

```plaintext
├── conf/
│   ├── db-pool/            # DB pool config for Martini (contains pre-created .dbxml files for database connections)
│   └── overrides/          # Override config files for Martini
├── logs/                   # Martini logs 
├── data/                   # Application data 
├── packages/               # Custom Martini packages
├── postgresql/
│   └── data/               # Persistent PostgreSQL data
├── cassandra/
│   └── data/               # Persistent Cassandra data
├── .env                    # Environment variables
└── docker-compose.yml      # Stack configuration
````

> Make sure to create the required directories if they don't exist before running the stack.

## Configuration

### `.env` File

Create a `.env` file in the root directory with the following variables:

```env
MR_LICENSE=your_martini_license_key

POSTGRES_USER=your_pg_user
POSTGRES_PASSWORD=your_pg_password
POSTGRES_DB=your_pg_db

CASSANDRA_USER=your_cassandra_user
CASSANDRA_PASSWORD=your_cassandra_password
```

### Environment Variables in `docker-compose.yml`

* `MR_TRACKER_DATABASE_NAME` sets the name of the external database used by Martini Tracker.
* `MR_TRACKER_ENABLE_EMBEDDED_DATABASE=false` disables the embedded Nitrite database in favor of external DBs, in this case it uses Cassandra for Tracker.

## Usage

To build and run the stack:

```bash
    docker-compose up -d
```

This will:

* Start Martini on ports **8080 (HTTP)** and **8443 (HTTPS)**
* Start PostgreSQL on **5432**
* Start Cassandra on **9042**

To stop the stack:

```bash
docker-compose down
```

> Use `--volumes` with `down` if you want to clear stored data (*this is only applicable for volume mounts*):

```bash
    docker-compose down --volumes
```

## Testing Connectivity

Once the containers are running:

* Visit [http://localhost:8080](http://localhost:8080) to access the Martini UI.
* Check PostgreSQL and Cassandra connectivity using the API explorer.

### Connecting to Databases from Martini

This Docker Compose implementation **automatically configures database connections** by mounting pre-created `.dbxml` files into the Martini container. These files are placed in the `conf/db-pool/` directory and are loaded by Martini at runtime.

When defining your database connection pools in Martini (e.g., via `.dbxml` files), you can use the service names defined in `docker-compose.yml` as hostnames instead of static IP addresses:

| Database     | Hostname     | Port  |
|--------------|--------------|-------|
| PostgreSQL   | `postgres`   | 5432  |
| Cassandra    | `cassandra`  | 9042  |

For example, in your `.dbxml` configuration file:

```xml
<database>
  <name>my_postgres_db</name>
  <driver>org.postgresql.Driver</driver>
  <url>jdbc:postgresql://postgres:5432/your_pg_db</url>
  <username>your_pg_user</username>
  <password>your_pg_password</password>
</database>
````

> This works because Docker Compose creates a shared network (`martini_network`) where each service name acts as its hostname.

## Notes

* Ensure Docker is installed and running before using this stack.
* Martini requires a valid license key to run. Set it via the `MR_LICENSE` environment variable.
* Logs and data will persist across container restarts via mounted volumes.


