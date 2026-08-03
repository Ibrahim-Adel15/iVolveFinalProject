# DevOps Roadmap Web Application

A simple microservices-based web application designed as a DevOps practice project.

The application provides user registration and authentication, followed by a main DevOps Roadmap page after successful login. The project is implemented using multiple microservices, with each service responsible for a specific part of the application.

The application is containerized using Docker. Each microservice contains its own `Dockerfile` and can be built and deployed independently.

> **Note:** `docker-compose.yml` is intentionally not included in this repository.

---

## 1. Architecture

The application follows a microservices architecture consisting of:

* **Frontend Service** – Provides the web interface and handles user interaction.
* **Auth Service** – Handles user registration and login.
* **Roadmap Service** – Provides the DevOps Roadmap/main application functionality.
* **MySQL** – Stores user account information.
* **NoSQL Database** – Stores application/roadmap-related data.

### High-Level Architecture

```text
                         ┌─────────────────────┐
                         │       Browser       │
                         └──────────┬──────────┘
                                    │
                                    │ HTTP
                                    ▼
                         ┌─────────────────────┐
                         │   Frontend Service  │
                         │      Node.js        │
                         └──────────┬──────────┘
                                    │
                         ┌──────────┴──────────┐
                         │                     │
                         │ HTTP                │ HTTP
                         ▼                     ▼
                ┌─────────────────┐   ┌──────────────────┐
                │   Auth Service  │   │ Roadmap Service  │
                │      Java       │   │     Python       │
                └────────┬────────┘   └─────────┬────────┘
                         │                      │
                         │                      │
                         ▼                      ▼
                ┌─────────────────┐   ┌──────────────────┐
                │      MySQL      │   │    NoSQL DB      │
                │   User Data     │   │ Roadmap Data     │
                └─────────────────┘   └──────────────────┘
```

---

## 2. Services

The project contains three application microservices.

| Service         | Technology              | Responsibility                                 |
| --------------- | ------------------------ | ----------------------------------------------- |
| Frontend        | Node.js / Express / EJS  | Web UI and communication with backend services |
| Auth Service    | Java                      | User registration and authentication           |
| Roadmap Service | Python                    | DevOps roadmap/application data                |

The databases are external dependencies of the microservices.

| Database       | Type       | Used By         | Purpose                         |
| -------------- | ---------- | ---------------- | -------------------------------- |
| MySQL          | Relational | Auth Service     | Stores registered users         |
| NoSQL Database | NoSQL      | Roadmap Service  | Stores roadmap/application data |

---

## 3. Frontend Service

The frontend is responsible for providing the user interface.

It contains two main pages:

### Authentication Page

Users can:

* Create an account
* Log in
* Submit their credentials to the Auth Service

### Roadmap Page

After successful authentication, the user is redirected to the main application page containing the DevOps Roadmap.

### Technology

* Node.js
* Express.js
* EJS
* HTML
* CSS

### Directory Structure

```text
frontend/
├── Dockerfile
├── package.json
├── server.js
├── public/
│   └── css/
│       └── style.css
└── views/
    ├── auth.ejs
    ├── error.ejs
    └── roadmap.ejs
```

### Port

```text
Frontend: <FRONTEND_PORT>
```

The frontend listens for incoming HTTP requests on the configured port.

### Environment Variables

The frontend requires the following environment variables:

```text
AUTH_SERVICE_URL=<AUTH_SERVICE_URL>
ROADMAP_SERVICE_URL=<ROADMAP_SERVICE_URL>
PORT=<FRONTEND_PORT>
```

Example:

```text
AUTH_SERVICE_URL=http://auth-service:<AUTH_SERVICE_PORT>
ROADMAP_SERVICE_URL=http://roadmap-service:<ROADMAP_SERVICE_PORT>
PORT=<FRONTEND_PORT>
```

> The actual values should be provided through the deployment environment and should not be hardcoded into the application.

---

## 4. Auth Service

The Auth Service is responsible for user authentication and account management.

It handles:

* User registration
* User login
* Credential validation
* Password storage
* Communication with MySQL

The Auth Service is the **only service that communicates directly with the MySQL database** for user account information.

### Responsibilities

```text
Signup
   │
   ▼
Auth Service
   │
   ▼
MySQL
   │
   ▼
User Created
```

For login:

```text
User
 │
 ▼
Frontend
 │
 ▼
Auth Service
 │
 ▼
MySQL
 │
 ▼
Credentials Validated
```

### Technology

* Java
* Spring Boot
* MySQL

### Port

```text
Auth Service: <AUTH_SERVICE_PORT>
```

### Environment Variables

The Auth Service requires database connection information.

```text
DB_HOST=<MYSQL_HOST>
DB_PORT=<MYSQL_PORT>
DB_NAME=<MYSQL_DATABASE>
DB_USERNAME=<MYSQL_USERNAME>
DB_PASSWORD=<MYSQL_PASSWORD>
```

Example:

```text
DB_HOST=mysql
DB_PORT=3306
DB_NAME=users
DB_USERNAME=<username>
DB_PASSWORD=<password>
```

**Never commit the actual database password to GitHub.**

---

## 5. Roadmap Service

The Roadmap Service provides the data and functionality used by the main DevOps Roadmap page.

It is separated from the authentication functionality so that roadmap/application functionality can be developed and deployed independently from user authentication.

### Responsibilities

* Provide roadmap data
* Process roadmap-related requests
* Communicate with the NoSQL database
* Return roadmap information to the frontend

### Technology

* Python
* Flask/FastAPI
* NoSQL database

### Port

```text
Roadmap Service: <ROADMAP_SERVICE_PORT>
```

### Environment Variables

```text
DB_HOST=<NOSQL_HOST>
DB_PORT=<NOSQL_PORT>
DB_NAME=<NOSQL_DATABASE>
```

Additional variables should be added here if the service requires authentication keys, database URLs, or other configuration.

---

## 6. Database Architecture

The application uses two different database technologies.

### MySQL

MySQL is used for structured user information.

The Auth Service communicates directly with MySQL.

**Database**

```text
users
```

**User Information**

The user database contains information such as:

```text
firstname
lastname
email
password
```

Conceptually:

```text
MySQL
└── users
    └── users table
        ├── firstname
        ├── lastname
        ├── email
        └── password
```

The exact schema should be maintained by the Auth Service/database initialization process.

### NoSQL Database

The NoSQL database is used for application-specific information such as the DevOps Roadmap.

The Roadmap Service communicates directly with the NoSQL database.

Conceptually:

```text
NoSQL Database
└── roadmap/application data
    ├── Linux
    ├── Git
    ├── Docker
    ├── Kubernetes
    ├── CI/CD
    ├── Terraform
    └── Cloud
```

The exact collection/document structure depends on the current implementation.

---

## 7. Service Communication

The frontend acts as the entry point for users.

Backend communication follows this model:

```text
Browser
   │
   ▼
Frontend
   │
   ├──────────────► Auth Service ─────────────► MySQL
   │
   └──────────────► Roadmap Service ──────────► NoSQL DB
```

The frontend does not communicate directly with the databases.

This provides separation between:

* User interface
* Authentication
* Application functionality
* Data storage

---

## 8. Authentication Flow

### User Registration

```text
1. User opens the application.
2. User enters first name, last name, email and password.
3. Frontend receives the registration request.
4. Frontend sends the request to the Auth Service.
5. Auth Service validates the request.
6. Auth Service stores the user in MySQL.
7. Auth Service returns the result to the Frontend.
8. Frontend displays the result to the user.
```

### User Login

```text
1. User enters email and password.
2. Frontend sends the credentials to the Auth Service.
3. Auth Service checks MySQL.
4. Credentials are validated.
5. Auth Service returns the authentication result.
6. Frontend allows the user to access the main application page.
```

---

## 9. API Endpoints

The exact endpoints should match the implementation in each service.

### Auth Service

Typical endpoints:

| Method | Endpoint  | Description                   |
| ------ | --------- | ------------------------------ |
| POST   | `/signup` | Create a new user             |
| POST   | `/login`  | Authenticate an existing user |

Example signup request:

```json
{
  "firstname": "John",
  "lastname": "Doe",
  "email": "john@example.com",
  "password": "********"
}
```

Example login request:

```json
{
  "email": "john@example.com",
  "password": "********"
}
```

### Roadmap Service

Typical endpoints:

| Method | Endpoint   | Description                  |
| ------ | ---------- | ------------------------------ |
| GET    | `/roadmap` | Retrieve roadmap information |
| GET    | `/health`  | Service health check         |

> Update this section if the actual API paths differ.

---

## 10. Ports

The following table should be kept synchronized with the application configuration.

| Component       |                     Port | Protocol | Purpose              |
| --------------- | ------------------------: | -------- | --------------------- |
| Frontend        |        `<FRONTEND_PORT>` | HTTP     | Web application      |
| Auth Service    |    `<AUTH_SERVICE_PORT>` | HTTP     | Authentication API   |
| Roadmap Service | `<ROADMAP_SERVICE_PORT>` | HTTP     | Roadmap API          |
| MySQL           |                   `3306` | TCP      | User database        |
| NoSQL DB        |           `<NOSQL_PORT>` | TCP      | Application database |

Ports should be configurable through environment variables where possible.

---

## 11. Environment Variables

Configuration should be provided through environment variables instead of hardcoding values into the application.

### Frontend

```text
PORT=<FRONTEND_PORT>
AUTH_SERVICE_URL=<AUTH_SERVICE_URL>
ROADMAP_SERVICE_URL=<ROADMAP_SERVICE_URL>
```

### Auth Service

```text
DB_HOST=<MYSQL_HOST>
DB_PORT=<MYSQL_PORT>
DB_NAME=<MYSQL_DATABASE>
DB_USERNAME=<MYSQL_USERNAME>
DB_PASSWORD=<MYSQL_PASSWORD>
```

### Roadmap Service

```text
DB_HOST=<NOSQL_HOST>
DB_PORT=<NOSQL_PORT>
DB_NAME=<NOSQL_DATABASE>
```

### Important

Do not commit files containing real secrets.

For example, do not commit:

```text
DB_PASSWORD=MyRealPassword
```

Instead, use:

```text
DB_PASSWORD=<password>
```

or provide the value through the deployment environment/secrets manager.

---

## 12. Docker

Each microservice has its own Dockerfile.

The services are designed to be containerized independently.

Example repository structure:

```text
project/
│
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js
│   └── ...
│
├── auth-service/
│   ├── Dockerfile
│   ├── pom.xml
│   └── ...
│
├── roadmap-service/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── ...
│
└── README.md
```

Each Dockerfile is responsible for:

1. Selecting the required base image.
2. Installing application dependencies.
3. Copying the application source code.
4. Configuring the application.
5. Exposing the required application port.
6. Starting the microservice.

### Building an Image

From a service directory:

```bash
docker build -t <service-name>:latest .
```

Example:

```bash
cd frontend
docker build -t frontend:latest .
```

The same approach can be used for the other services.

---

## 13. Running the Services Independently

Because the services are independent, each service can be built and run separately.

Example:

```bash
docker build -t frontend:latest ./frontend
docker build -t auth-service:latest ./auth-service
docker build -t roadmap-service:latest ./roadmap-service
```

When running the containers, provide the required environment variables.

Example:

```bash
docker run \
  -p <FRONTEND_PORT>:<FRONTEND_PORT> \
  -e AUTH_SERVICE_URL=<AUTH_SERVICE_URL> \
  -e ROADMAP_SERVICE_URL=<ROADMAP_SERVICE_URL> \
  frontend:latest
```

Database services must be available before starting the microservices that depend on them.

---

## 14. Repository Structure

The repository is organized by microservice.

```text
.
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js
│   ├── public/
│   │   └── css/
│   │       └── style.css
│   └── views/
│       ├── auth.ejs
│       ├── error.ejs
│       └── roadmap.ejs
│
├── auth-service/
│   ├── Dockerfile
│   └── ...
│
├── roadmap-service/
│   ├── Dockerfile
│   └── ...
│
└── README.md
```

---

## 15. Development Guidelines

When modifying the application:

### Frontend

Frontend-related changes should be made inside:

```text
frontend/
```

### Authentication

Authentication and user/database-related changes should be made inside:

```text
auth-service/
```

### Roadmap

Roadmap/application functionality should be implemented inside:

```text
roadmap-service/
```

Each service should remain independently deployable.

Avoid introducing direct database access from the frontend.

---

## 16. Configuration Guidelines

Application configuration should be externalized.

Do not hardcode:

* Database passwords
* API keys
* Authentication secrets
* Database hostnames
* Production URLs
* Environment-specific configuration

Use environment variables instead.

This allows the same Docker image to be used in different environments such as:

```text
Development
     │
     ▼
Testing
     │
     ▼
Staging
     │
     ▼
Production
```

without rebuilding the application for every environment.

---

## 17. Security Considerations

This project is intended primarily as a DevOps/microservices practice application.

For a production implementation, additional security controls should be implemented.

Important considerations include:

* Never commit passwords to Git.
* Never commit API keys or tokens.
* Store secrets using a secrets-management solution.
* Passwords should be securely hashed before being stored.
* Use HTTPS in production.
* Validate all user input.
* Implement authentication and authorization properly.
* Add rate limiting to authentication endpoints.
* Restrict database network access.
* Use non-root users inside containers where possible.
* Keep base images and dependencies updated.
* Do not expose databases directly to the public internet.

---

## 18. Health Checks

Each microservice should provide a health endpoint where possible.

Example:

```text
GET /health
```

Expected response:

```json
{
  "status": "UP"
}
```

Health checks can be used by container orchestration platforms and monitoring systems to determine whether a service is available.

---

## 19. Troubleshooting

### Frontend cannot reach Auth Service

Check:

```text
AUTH_SERVICE_URL
```

Verify that:

* Auth Service is running.
* The hostname is resolvable from the frontend container.
* The correct port is configured.
* Network connectivity exists between the containers/services.

### Auth Service cannot connect to MySQL

Check:

```text
DB_HOST
DB_PORT
DB_NAME
DB_USERNAME
DB_PASSWORD
```

Also verify that:

```text
MySQL is running
Database exists
User has the required permissions
Network connectivity exists
```

### Roadmap Service cannot connect to NoSQL Database

Check:

```text
DB_HOST
DB_PORT
DB_NAME
```

Verify that:

* NoSQL database is running.
* The database/collection exists if required.
* The service can resolve the database hostname.
* The configured port is correct.

### Container starts but service is unreachable

Check:

```bash
docker ps
```

Then inspect the container logs:

```bash
docker logs <container-name>
```

Check which port the application is listening on:

```bash
docker exec -it <container-name> sh
```

Then verify the application's configuration and environment variables.

---

## 20. Deployment Model

The application is designed so that each microservice can be deployed independently.

For example:

```text
                 Git Repository
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
      Frontend      Auth Service   Roadmap
          │            │            │
          ▼            ▼            ▼
       Container    Container     Container
          │            │            │
          └────────────┼────────────┘
                       │
                       ▼
                Runtime Platform
```

This architecture makes it possible to:

* Scale individual services independently.
* Deploy services independently.
* Update one service without rebuilding all services.
* Assign different resources to different services.
* Implement independent CI/CD pipelines.

---

## 21. CI/CD Considerations

Each microservice can have its own CI/CD pipeline.

A typical pipeline can contain:

```text
Checkout
   │
   ▼
Build
   │
   ▼
Unit Tests
   │
   ▼
Build Docker Image
   │
   ▼
Security Scan
   │
   ▼
Push Image to Registry
   │
   ▼
Deploy
   │
   ▼
Health Check
```

The services can also be versioned independently.

For example:

```text
frontend:v1.0.0
auth-service:v1.2.0
roadmap-service:v1.1.0
```

---

## 22. Future Improvements

Potential improvements include:

* Implement JWT-based authentication.
* Add refresh tokens.
* Add role-based access control.
* Add API Gateway/BFF.
* Add centralized logging.
* Add Prometheus metrics.
* Add Grafana dashboards.
* Add distributed tracing.
* Add automated unit and integration tests.
* Add container vulnerability scanning.
* Add Kubernetes deployment manifests.
* Add Kubernetes ConfigMaps and Secrets.
* Implement Horizontal Pod Autoscaling.
* Add centralized secret management.
* Add API documentation using OpenAPI/Swagger.
* Implement database migrations.
* Add automated CI/CD pipelines.

---

## 23. Important Project Notes

* The application consists of three independent microservices.
* Each microservice has its own Dockerfile.
* The frontend communicates with backend services through HTTP APIs.
* The frontend does not directly access either database.
* The Auth Service is responsible for user data and MySQL access.
* The Roadmap Service is responsible for roadmap/application data and NoSQL access.
* Configuration should be supplied through environment variables.
* Secrets must not be committed to GitHub.
* `docker-compose.yml` is intentionally excluded from this repository.
* Service ports and environment variables must remain synchronized with the actual application configuration.

---

## 24. Quick Reference

```text
Frontend
    │
    ├── HTTP ──► Auth Service ──► MySQL
    │
    └── HTTP ──► Roadmap Service ──► NoSQL Database
```

### Services

```text
Frontend Service
Technology: Node.js
Port: <FRONTEND_PORT>

Auth Service
Technology: Java
Port: <AUTH_SERVICE_PORT>

Roadmap Service
Technology: Python
Port: <ROADMAP_SERVICE_PORT>
```

### Databases

```text
MySQL
Purpose: User authentication/account data

NoSQL
Purpose: Roadmap/application data
```

---

## Maintainer Notes

Before deploying this project to a new environment, verify:

- [ ] All required environment variables are configured.
- [ ] MySQL is reachable from the Auth Service.
- [ ] NoSQL database is reachable from the Roadmap Service.
- [ ] Frontend can reach Auth Service.
- [ ] Frontend can reach Roadmap Service.
- [ ] Required ports are available.
- [ ] No secrets are committed to the repository.
- [ ] Docker images build successfully.
- [ ] All services pass their health checks.
- [ ] Database schemas/data are initialized correctly.
- [ ] The deployment environment provides the required networking between services.

---

## License

Add the project's license information here if applicable.
