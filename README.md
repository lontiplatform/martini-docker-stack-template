# Martini Runtime Docker Stack

This repository provides a Docker-based stack to run Martini Runtime with support for external database integration. It includes automated bind mounting and a modular structure to support custom configuration and data persistence.

## Stack Overview

This `docker-compose` implementation includes:

- **Martini Runtime** (via `lontiplatform/martini-server-runtime`)
- Support for **external databases** such as relational (e.g., PostgreSQL) and NoSQL (e.g., Cassandra)
- Automatic mounting of configuration files into the Martini container
- Persistent storage for logs, data, and packages on the host machine
- Example setup using PostgreSQL, MySQL, SQL Server, or Oracle as external JDBC datasources, and Cassandra or DynamoDB as alternatives for Tracker database

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
# Martini Runtime
MR_LICENSE=your_martini_license_key
MR_TRACKER_DATABASE_NAME=
MR_TRACKER_ENABLE_EMBEDDED_DATABASE=false        # Set value to `true` to use the embedded database or `false` to use Cassandra or DynamoDB as the Tracker database
MR_TRACKER_DYNAMODB_TABLE=
MR_TRACKER_DYNAMODB_STATE_TABLE=
MR_TRACKER_DYNAMODB_STATE_CONTENT_TABLE_NAME=
MR_TRACKER_DYNAMODB_TYPE_TABLE_NAME=

# Postgres
POSTGRES_USER=your_pg_user
POSTGRES_PASSWORD=your_pg_password
POSTGRES_DB=your_pg_db

#Cassandra
CASSANDRA_USER=your_cassandra_user
CASSANDRA_PASSWORD=your_cassandra_password

#MySQL
MYSQL_USER=your_mysql_user
MYSQL_ROOT_PASSWORD=your_mysql_root_password
MYSQL_PASSWORD=your_mysql_user_password
MYSQL_DATABASE=your_mysql_db

#SQL Server
SA_PASSWORD=your_mssql_password
ACCEPT_EULA=Y                   

#Oracle
ORACLE_PASSWORD=your_oracle_password
APP_USER=your_app_user
APP_USER_PASSWORD=your_app_user_password
```

> You can adjust or extend this file based on the external databases you plan to use.

### Environment Variables in `docker-compose.yml`

* `MR_TRACKER_DATABASE_NAME` sets the name of the external database used by Martini Tracker.
* `MR_TRACKER_ENABLE_EMBEDDED_DATABASE=false` disables the embedded Nitrite database in favor of external DBs, in this case it uses Cassandra for Tracker.
* `MR_TRACKER_DYNAMODB_TABLE` Name of the DynamoDB table used to store tracker records.
* `MR_TRACKER_DYNAMODB_STATE_TABLE` Name of the DynamoDB table used to store tracker state information.
* `MR_TRACKER_DYNAMODB_STATE_CONTENT_TABLE_NAME` Name of the DynamoDB table used to store tracker state content.
* `MR_TRACKER_DYNAMODB_TYPE_TABLE_NAME` Name of the DynamoDB table used to store tracker document type mappings.

> **Note:** If you prefer not to use an external database for Tracker (e.g., Cassandra or DynamoDB), you can revert to the embedded **Nitrite** database by simply removing or commenting out the related `MR_TRACKER_*` environment variables in the `docker-compose.yml` file. Martini will automatically fall back to using the embedded database.

## Usage

To build and run the stack:

```bash
    docker-compose up -d
```

This will:

* Start Martini on ports **8080 (HTTP)** and **8443 (HTTPS)**
* Start example databases on their default ports (e.g., PostgreSQL on 5432, Cassandra on 9042)

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
* Check databases connectivity using the API explorer.

### Connecting to Databases from Martini

This setup supports connecting Martini to one or more external databases. By default, the stack includes examples for different databases, but you can customize it to use any compatible database by:

1. Adding the service definition in docker-compose.yml

2. Mounting the relevant .dbxml configuration files into the conf/db-pool/ directory

When defining your database connection pools in Martini (e.g., via `.dbxml` files), you can use the service names defined in `docker-compose.yml` as hostnames instead of static IP addresses:

| Database                     | Hostname    | Port |
| ---------------------------- | ----------- | ---- |
| PostgreSQL                   | `postgres`  | 5432 |
| Cassandra                    | `cassandra` | 9042 |
| MySQL                        | `mysql`     | 3306 |
| Oracle                       | `oracle`    | 1521 |
| Microsoft SQL Server (MSSQL) | `mssql`     | 1433 |

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

This approach applies equally to other database types (e.g., MySQL, MongoDB) with the appropriate JDBC URL and driver. For detailed examples and database-specific properties, refer to the official [Martini JDBC configuration documentation](https://developer.lonti.com/docs/martini/installation-configuration/server-runtime/self-managed/configuration/dependencies/databases/jdbc/#database-properties). 

> This works because Docker Compose creates a shared network (`martini_network`) where each service name acts as its hostname.

### Enabling Additional Databases

By default, only **PostgreSQL** and **Cassandra** are enabled in the `docker-compose.yml` file as examples. Other database services such as **MySQL**, **Oracle**, **Microsoft SQL Server**, and **DynamoDB (for Tracker)** are included but **commented out** to keep the stack lightweight.

To enable any of these additional databases:

1. **Open** the `docker-compose.yml` file.
2. **Locate** the commented-out service block for the database you want to use.
3. **Uncomment** the service definition and any associated volume or network configuration.

For example, to enable **MySQL**, remove the `#` symbols from:

```yaml
# mysql:
#   image: mysql:latest
#   environment:
#     MYSQL_ROOT_PASSWORD: your_password
#     MYSQL_DATABASE: your_db
#     MYSQL_USER: your_user
#     MYSQL_PASSWORD: your_password
#    ports:
#      - "3306:3306"
#   volumes:
#     - ./mysql-data:/var/lib/mysql
#   networks:
#     - martini_network
```

Be sure to update and mount the corresponding `.dbxml` configuration in `conf/db-pool/` with the correct JDBC connection URL, credentials, and driver class name for the database you enabled. This repository includes sample `.dbxml` files for each supported database under the `conf/db-pool/` directory. You can use or customize these as needed for your environment.

## Notes

* Ensure Docker is installed and running before using this stack.
* Martini requires a valid license key to run. Set it via the `MR_LICENSE` environment variable.
* Logs and data will persist across container restarts through bind mounts to the host filesystem.



