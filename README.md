# Dev Infra

A shared local development infrastructure powered by Docker Compose, providing essential services for Laravel applications and other backend projects.

Instead of running separate database, cache, storage, and email services for every project, this repository provides a centralized stack that can be reused across multiple applications.

## Services

| Service | Image | Default Port | Purpose |
|---|---|---|---|
| MySQL | `mysql:8.4.7` | `3306` | Relational database |
| Redis | `redis:8.2.3` | `6379` | Cache, queues, and sessions |
| MinIO | `minio/minio` | `9000`, `9001` | S3-compatible object storage |
| Mailpit | `axllent/mailpit:v1.28.0` | `1025`, `8025` | Local SMTP testing and email preview |
| phpMyAdmin | `phpmyadmin:5.2.3` | `8080` | MySQL administration |
| RedisInsight | `redis/redisinsight:3.6` | `5540` | Redis monitoring and management |

## Requirements

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) or Docker Engine
- Docker Compose v2
- Git (optional)

## Project Structure

```text
dev-infra/
├── docker-compose.yaml
├── .env
├── .env.example
└── README.md
```

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/matyo-17/dev-infra.git dev-infra
cd dev-infra
```

Alternatively, navigate to your existing `dev-infra` directory.

### 2. Configure environment variables

Create your local environment file:

```bash
cp .env.example .env
```

Update `.env` with your preferred ports and strong, unique credentials.

Example:

```dotenv
# MySQL
MYSQL_PORT=3306
MYSQL_ROOT_PASSWORD=replace_with_a_strong_root_password
MYSQL_DATABASE=laravel
MYSQL_USER=laravel
MYSQL_PASSWORD=replace_with_a_strong_app_password

# Redis
REDIS_PORT=6379

# MinIO
MINIO_API_PORT=9000
MINIO_CONSOLE_PORT=9001
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=replace_with_a_strong_minio_password

# Mailpit
MAILPIT_SMTP_PORT=1025
MAILPIT_WEB_PORT=8025

# phpMyAdmin
PHPMYADMIN_PORT=8080

# RedisInsight
REDISINSIGHT_PORT=5540
```

Ensure these variables match those referenced in `docker-compose.yaml`. The database name and application user variables are required only if your Compose configuration uses them.

### 3. Start the infrastructure

Validate the Compose configuration:

```bash
docker compose config --quiet
```

Start the services:

```bash
docker compose up -d
```

Check container status:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

## Accessing Services

Once the containers are running, access the local services using the following addresses.

| Service | URL / Address |
|---|---|
| phpMyAdmin | http://127.0.0.1:8080 |
| MinIO Console | http://127.0.0.1:9001 |
| MinIO API | http://127.0.0.1:9000 |
| Mailpit Web UI | http://127.0.0.1:8025 |
| Mailpit SMTP | `127.0.0.1:1025` |
| MySQL | `127.0.0.1:3306` |
| Redis | `127.0.0.1:6379` |
| RedisInsight | http://127.0.0.1:5540 |

Use the credentials configured in `.env` when logging into services that require authentication.

## Using with Laravel

Laravel applications running directly on the host machine can connect to the services using `127.0.0.1` and the published ports.

Add the relevant settings to each Laravel application's own `.env` file.

### MySQL

```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_project_database
DB_USERNAME=your_project_user
DB_PASSWORD=your_project_password
```

Create a separate database and restricted database user for each application.

### Redis

```dotenv
REDIS_CLIENT=phpredis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
REDIS_PREFIX=your_project_database_
```

The `phpredis` client requires the PHP Redis extension. Use a different client setting if your application uses Predis.

Use distinct prefixes for applications sharing the same Redis instance, and verify the effective configuration for cache, queues, and sessions.

### MinIO

```dotenv
AWS_ACCESS_KEY_ID=your_minio_access_key
AWS_SECRET_ACCESS_KEY=your_minio_secret_key
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=your-project-bucket
AWS_ENDPOINT=http://127.0.0.1:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

Create a separate bucket for each application and use application-specific credentials with limited permissions.

Ensure the Laravel S3 filesystem adapter is installed and configured.

### Mailpit

```dotenv
MAIL_MAILER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

All emails sent through the shared Mailpit SMTP service can be viewed at http://127.0.0.1:8025.

Mailpit is intended for development and testing, not production email delivery.

## Sharing Infrastructure Across Projects

A single running infrastructure stack can support multiple Laravel applications.

Recommended isolation:

- **MySQL:** Use a separate database and database user for each application.
- **Redis:** Use unique key prefixes and appropriate connection configuration.
- **MinIO:** Use a separate bucket and restricted credentials for each application.
- **Mailpit:** Share the SMTP server for local email testing, keeping in mind that the inbox is shared.

Example workspace:

```text
development/
├── dev-infra/
│   ├── docker-compose.yaml
│   ├── .env
│   └── README.md
├── shop-app/
│   └── .env
├── crm-app/
│   └── .env
└── admin-app/
    └── .env
```

Start the infrastructure once, then run the Laravel applications independently.

## Managing the Infrastructure

Start services:

```bash
docker compose up -d
```

Stop services without removing persistent data:

```bash
docker compose down
```

Restart services:

```bash
docker compose restart
```

Restart a specific service:

```bash
docker compose restart mysql
```

View service status:

```bash
docker compose ps
```

View logs for a specific service:

```bash
docker compose logs -f mysql
```

Pull updated images:

```bash
docker compose pull
```

Apply updated images by recreating containers:

```bash
docker compose up -d
```

Review release notes and compatibility before updating pinned image versions.

## Persistent Storage

The following named Docker volumes preserve service data across container recreation.

| Volume | Purpose |
|---|---|
| `mysql-data` | MySQL database files |
| `redis-data` | Redis persistence data |
| `minio-data` | Object storage files |
| `mailpit-data` | Captured email data |
| `redisinsight-data` | RedisInsight application data |

Stopping or removing containers with `docker compose down` does not normally remove these named volumes.

**Warning:** Avoid `docker compose down -v` unless you intentionally want to delete the associated persistent volumes and their data.

Changes to MySQL initialization variables do not automatically update existing database credentials or recreate databases in an already initialized data volume.

Maintain backups of important data independently of Docker volumes.

## Security

- Never commit `.env` files containing real credentials.
- Keep `.env.example` free of secrets.
- Use strong, unique passwords for administrative accounts.
- Use dedicated, least-privilege database and object-storage credentials for applications.
- Bind published ports to `127.0.0.1` for local-only access where appropriate.
- Do not expose development management interfaces directly to the public internet.
- Do not use development credentials or this local configuration as-is in production.

## Troubleshooting

### Port already in use

If a service fails to start because its host port is occupied, change its corresponding port in `.env` and restart the stack.

For example:

```dotenv
MYSQL_PORT=3307
```

If you change a published port, update the corresponding Laravel configuration as well.

### Laravel cannot connect to MySQL

Check the following:

1. Confirm MySQL is running with `docker compose ps`.
2. Inspect MySQL logs with `docker compose logs mysql`.
3. Verify the host, published port, database name, username, and password in Laravel's `.env`.
4. Confirm the database and application user exist.
5. Clear cached Laravel configuration:

   ```bash
   php artisan config:clear
   ```

### Laravel cannot connect to Redis

Verify that Redis is healthy and listening on the expected host port. Check the Redis client extension, connection settings, and application-specific prefixes.

### Credentials appear unchanged

MySQL and MinIO can retain credentials in their existing persistent data. Updating `.env` alone may not update already initialized services.

Use the appropriate administrative procedure to change existing credentials rather than deleting volumes indiscriminately.

## License

This project is licensed under the [MIT License](LICENSE).