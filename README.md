# RoboShop Docker

A containerized, microservices-based e-commerce application ("RoboShop") built with Docker, Docker Compose, and Kubernetes. The project splits a typical online store into independent services — written in Node.js, Java, and Python — each with its own Dockerfile, dependencies, and data store, orchestrated together via `docker-compose.yaml` and deployable to Kubernetes.

## Architecture

| Service      | Language / Stack        | Description                                              | Depends On                          |
|--------------|--------------------------|------------------------------------------------------------|--------------------------------------|
| `frontend`   | Nginx (static + reverse proxy) | Serves the static web UI and routes API traffic to backend services | `catalogue`, `user`, `cart`, `shipping`, `payment` |
| `catalogue`  | Node.js (Express)       | Product catalogue / listing REST API                       | `mongodb`                            |
| `user`       | Node.js (Express)       | User accounts, authentication, and profiles                | `redis`, `mongodb`                   |
| `cart`       | Node.js (Express)       | Shopping cart REST API                                      | `redis`                              |
| `shipping`   | Java (Spring Boot)      | Shipping cost calculation and order fulfillment              | `mysql`, `cart`                      |
| `payment`    | Python                  | Payment processing, publishes events via RabbitMQ            | `rabbitmq`, `cart`, `user`           |
| `mysql`      | MySQL                   | Relational database (used by `shipping`)                    | —                                     |
| `mongodb`    | MongoDB                 | Document database (used by `catalogue`, `user`), seeded via `master-data.js` | —                     |
| `redis`      | Redis 7.0                | In-memory store/cache (used by `user`, `cart`)               | —                                     |
| `rabbitmq`   | RabbitMQ 3               | Message broker for async communication (used by `payment`)   | —                                     |
| `debug`      | Utility                  | Debug/troubleshooting tooling for the deployment              | —                                     |

## Repository Structure

```
roboshop-docker/
├── cart/                  # Cart service (Node.js/Express)
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── catalogue/              # Catalogue service (Node.js/Express)
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── debug/                  # Debug/troubleshooting utilities
│   └── Dockerfile
├── frontend/                # Frontend web app (Nginx)
│   ├── static/
│   ├── Dockerfile
│   └── nginx.conf
├── mongodb/                 # MongoDB service + seed data (Terraform provisioning)
│   ├── Dockerfile
│   └── master-data.js
├── mysql/                   # MySQL service
│   ├── db/
│   └── Dockerfile
├── payment/                  # Payment service (Python)
│   ├── Dockerfile
│   ├── payment.ini
│   ├── payment.py
│   ├── rabbitmq.py
│   └── requirements.txt
├── shipping/                  # Shipping service (Java/Spring Boot, Maven)
│   ├── src/main/java/com/instana/robotshop/shipping/
│   ├── Dockerfile
│   └── pom.xml
├── user/                      # User service (Node.js/Express)
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yaml         # Local multi-container orchestration
└── README.md
```

## Tech Stack

- **Languages:** JavaScript (Node.js/Express), Java (Spring Boot), Python
- **Web Server:** Nginx (frontend, alpine-slim image)
- **Containerization:** Docker (multi-stage builds), Docker Compose
- **Orchestration:** Kubernetes (per-service manifests)
- **Infrastructure as Code:** Terraform (MongoDB provisioning)
- **Databases / Stores:** MySQL, MongoDB, Redis
- **Messaging:** RabbitMQ

## Getting Started

### Prerequisites

- [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/)
- [Kubernetes](https://kubernetes.io/) cluster (e.g., Minikube, EKS, GKE) for K8s deployments
- [Terraform](https://www.terraform.io/) (for MongoDB provisioning)

### Run Locally with Docker Compose

Clone the repository and start all services:

```bash
git clone https://github.com/e-shekharreddy/roboshop-docker.git
cd roboshop-docker
docker-compose -f docker-compose.yaml up -d --build
```

This builds each service's image from its local Dockerfile and starts the full stack — frontend, backend services, MySQL, MongoDB, Redis, and RabbitMQ — connected on a shared bridge network.

Once running, the frontend is available at:

```
http://localhost:80
```

To stop and remove the containers:

```bash
docker-compose down
```

### Environment Variables

Some services (e.g. `payment`) expect connection details for their dependencies, such as:

| Variable       | Description                  | Example         |
|-----------------|-------------------------------|-------------------|
| `CART_HOST`     | Hostname of the cart service   | `cart`            |
| `CART_PORT`     | Port of the cart service       | `8080`            |
| `USER_HOST`     | Hostname of the user service   | `user`            |
| `USER_PORT`     | Port of the user service       | `8080`            |
| `AMQP_HOST`     | RabbitMQ hostname                | `rabbitmq`        |
| `AMQP_USER`     | RabbitMQ username                 | `roboshop`        |
| `AMQP_PASS`     | RabbitMQ password                 | `roboshop123`     |

In Kubernetes deployments, these are managed via ConfigMaps rather than hardcoded in the Dockerfiles.

### Deploy to Kubernetes

Each service folder contains its own Kubernetes manifests. Apply them individually or as a group:

```bash
kubectl apply -f cart/
kubectl apply -f catalogue/
kubectl apply -f frontend/
kubectl apply -f payment/
kubectl apply -f shipping/
kubectl apply -f user/
```

### Provision MongoDB with Terraform

```bash
cd mongodb
terraform init
terraform plan
terraform apply
```

## Notable Implementation Details

- Node.js services (`cart`, `catalogue`, `user`) use **multi-stage Docker builds** (`node:20.20.2-alpine3.23`) to keep final images small, and run as a non-root `roboshop` user.
- The `frontend` service uses a hardened **Nginx alpine-slim** image with custom cache directories, permissions, and a non-root `nginx` user.
- The `shipping` service is a Java/Spring Boot app built with Maven, backed by MySQL.
- The `payment` service is written in Python and communicates with `cart` and `user` over HTTP, and publishes messages to RabbitMQ.
- `mongodb` includes a `master-data.js` script to seed initial data.

## Contributing

Contributions are welcome! Feel free to open an issue or submit a pull request for bug fixes, improvements, or new features.

## License

This project currently has no license specified. Add a `LICENSE` file to clarify usage terms.
