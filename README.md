# Fullstack API

A REST API written in Go for user accounts and posts. It uses Gorilla Mux for
routing, GORM for database access, and JSON for request and response bodies.

## Features

- Create, list, read, update, and delete users.
- Log in with an email and password to receive a JWT.
- Create, list, read, update, and delete posts.
- Restrict user and post updates and deletes to authenticated users.
- MySQL development database and a PostgreSQL test Compose configuration.

## Requirements

- Go version specified in [`go.mod`](./go.mod).
- Docker Engine and the Docker Compose plugin for the container workflows.

## Configuration

The application and tests load configuration from a `.env` file in the
repository root. The file is ignored by Git. Create one with values appropriate
to your local database; for example:

```dotenv
API_SECRET=replace-with-a-long-random-secret

# Main application database
DB_DRIVER=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=fullstack
DB_PASSWORD=change-me
DB_NAME=fullstack

# Test database
TestApiSecret=replace-with-a-test-secret
TestDbDriver=mysql
TestDbHost=127.0.0.1
TestDbPort=3306
TestDbUser=fullstack_test
TestDbPassword=change-me
TestDbName=fullstack_test
```

For the app running in Docker Compose, set `DB_HOST=fullstack-mysql` so it can
reach the MySQL service over the Compose network. For the app or tests running
directly on the VM, use the host name/address and port where the database is
reachable (for example, `127.0.0.1`).

Keep real credentials and signing secrets out of version control. The included
Compose files use the corresponding `.env` values when creating database
containers.

## Run with Docker Compose

Configure `.env` for MySQL, including `DB_HOST=fullstack-mysql`, then run:

```sh
docker compose up --build
```

The API listens on `http://localhost:8080`. phpMyAdmin is available at
`http://localhost:9090`. MySQL is published on port `3306`.

To stop the services:

```sh
docker compose down
```

## Run locally

Make sure a MySQL server is running and that the `DB_*` values in `.env` allow
the application to connect to it. Then run:

```sh
go run .
```

The server listens on port `8080`.

> **Warning:** On startup, the application drops and recreates the `users` and
> `posts` tables, then inserts sample records. Do not point it at a database
> containing data you need to keep.

## API

All responses use JSON. User and post list endpoints return up to 100 records.

| Method | Path | Description | Authentication |
| --- | --- | --- | --- |
| `GET` | `/` | Welcome response | No |
| `POST` | `/login` | Log in and receive a JWT | No |
| `POST` | `/users` | Create a user | No |
| `GET` | `/users` | List users | No |
| `GET` | `/users/{id}` | Get a user | No |
| `PUT` | `/users/{id}` | Update the authenticated user's account | JWT |
| `DELETE` | `/users/{id}` | Delete a user | JWT |
| `POST` | `/posts` | Create a post | JWT |
| `GET` | `/posts` | List posts | No |
| `GET` | `/posts/{id}` | Get a post | No |
| `PUT` | `/posts/{id}` | Update the authenticated user's post | JWT |
| `DELETE` | `/posts/{id}` | Delete the authenticated user's post | JWT |

Send a JWT in the `Authorization` header for protected operations:

```http
Authorization: Bearer <token>
```

Tokens expire after one hour. The token signing key is read from `API_SECRET`.

Example user creation request:

```sh
curl -X POST http://localhost:8080/users \
  -H 'Content-Type: application/json' \
  -d '{"nickname":"Ada","email":"ada@example.com","password":"change-me"}'
```

Example login request:

```sh
curl -X POST http://localhost:8080/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"ada@example.com","password":"change-me"}'
```

## Tests

The test packages read `TestDb*` settings from `.env` and connect to a
database. Run them with:

```sh
go test ./...
```

The test helpers recreate database tables, so use a disposable test database,
not a database containing data you need. A PostgreSQL test container is defined
in `docker-compose.test.yml`; its container-network hostname is
`postgres_test` and its host-published port is `5555`. The current PostgreSQL
test connection string in the test helpers contains a placeholder in place of
the password, so PostgreSQL-backed tests need that connection setup corrected
before they can connect successfully. MySQL-backed tests use the `TestDb*`
values shown in the configuration example.

## License

This project is distributed under the MIT License. See [`LICENSE`](./LICENSE).
