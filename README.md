# Microshop: A Microservices-Based E-commerce Platform

## Overview

Microshop is a microservices-based e-commerce platform designed to demonstrate a modern, scalable, and resilient application architecture. It's built using Spring Boot and a variety of other technologies to showcase best practices in microservice design, deployment, and operation.

## Technologies Used

### Core Technologies

*   **Spring Boot:** The foundation of our microservices. Spring Boot simplifies the development of Java-based applications by providing convention-over-configuration, embedded servers, and autoconfiguration features. It is used in the `api-gateway`, `inventory-service` and `product-service`.
*   **Java:** The primary programming language used for building the microservices. It ensures type safety and good performance.
*   **Maven:** Dependency management and build automation tool. It's used to manage project dependencies, build the application, and package it for deployment.
*   **Spring WebFlux:** Used in the `api-gateway` for a reactive and non-blocking approach to handling web requests.
*   **Spring Security:** Leveraged within `api-gateway` for securing the application endpoints.
*   **Springdoc OpenAPI (Swagger):**  Used to generate API documentation for each microservice. It provides a UI to interact with and test the endpoints.

### Microservice Architecture

*   **API Gateway (Spring Cloud Gateway):** The entry point for all external requests. It handles routing, load balancing, security, and cross-cutting concerns. The `api-gateway` is configured to use `Spring WebFlux`, `Spring Security`, and interacts with the `Eureka` server.
*   **Eureka (Service Discovery):** A service registry that allows microservices to find and communicate with each other dynamically. Each service registers itself with Eureka, making service discovery straightforward. The project is using a discovery server image: `240800/discovery-server:latest`.
*   **Synchronous Communication (Spring Cloud OpenFeign):** A declarative web service client that simplifies the interaction between services using HTTP. The API Gateway uses this to make calls to downstream services like `inventory-service` and `product-service`.
*   **Zipkin (Distributed Tracing):** It helps to gather timing data needed to troubleshoot latency problems in microservice architectures. It manages both the collection and lookup of this data and is used by all microservices.
*   **Circuit Breaker (Resilience4j with Spring Cloud Circuit Breaker):** Prevents cascading failures by monitoring service calls and temporarily stopping requests to a failing service. Implemented in the API Gateway for better microservices communications.

### Containerization and Orchestration

*   **Docker:** Used for containerizing the microservices and their dependencies, ensuring consistent environments across development, testing, and production. Each service has a Docker image built using `mvn spring-boot:build-image`.
*   **Kubernetes (k8s):** Container orchestration platform used to manage the deployment, scaling, and operations of the microservices in a cluster. Kubernetes is used for orchestrating the `api-gateway` service.

### Resiliency and Fault Tolerance

*   **Circuit Breaker (Resilience4j with Spring Cloud Circuit Breaker):** Prevents cascading failures by monitoring service calls and temporarily stopping requests to a failing service.
*   **Retry:** Allows for automatic retries of failed requests, enhancing resilience in transient network issues.
*   **Timeout:** Limits the waiting time for a service response, preventing indefinite delays.

### Data Management

*   **PostgreSQL:** Relational database used for the `inventory-service`. Configured in `docker-compose.yaml` to connect to the `inventory-service`.
*   **MongoDB:** A NoSQL document database used by the `product-service`. Configured in `product-service/src/main/resources/application.properties`. It is well-suited for flexible data modeling and horizontal scalability, ideal for scenarios where data can have variable or semi-structured structures.
*   **MySQL:** Another type of database used in the project, for the Keycloak database. See the `mysql_keycloak_data` folder.

### Communication

*   **Synchronous Communication:** Implemented using `Spring Cloud OpenFeign` for direct service-to-service calls.
*   **API Gateway:** All external communication is routed through the API Gateway, acting as a reverse proxy.

## Project Structure

The project is divided into the following modules:

*   **inventory-service:** Manages the product inventory. It stores product information and availability. It uses PostgreSQL as the database.
*   **product-service:** Manages product data, using MongoDB. The connection details are in the `product-service/src/main/resources/application.properties`.
*   **api-gateway:** The API gateway handles external requests, routing them to the appropriate services. It uses `Spring WebFlux`, `Spring Security`, `Eureka`, `Circuit Breaker`, and `OpenFeign`.
*   **k8s:** Contains the configuration files for deploying the services to Kubernetes, including a specific configuration for `api-gateway` (`api-gateway-deployment.yaml`).
*   **mysql_keycloak_data:** Contains configuration for the MySQL database used by Keycloak.

## Setting Up the Development Environment

### Prerequisites

1.  **Java Development Kit (JDK):** Ensure you have JDK 17+ installed.
2.  **Maven:** Install Maven and add it to your system's `PATH`.
3.  **Docker:** Install Docker Desktop for local containerization.
4.  **Kubernetes (Optional):** If you want to deploy to Kubernetes locally, install Minikube or Docker Desktop with Kubernetes enabled.

### Installation

1.  **Clone the Repository:**
    ```bash
    git clone <repository_url>
    cd microshop
    ```

2.  **Building the Projects:**
    ```bash
    # Navigate to each microservice directory (e.g., inventory-service, product-service, api-gateway)
    cd inventory-service
    mvn clean install
    cd ../product-service
    mvn clean install
    cd ../api-gateway
    mvn clean install

    # Repeat above steps for each microservice.
    ```

3.  **Create `inventory-service` database:**
    ```bash
    docker run --name postgres_db -e POSTGRES_USER=developer -e POSTGRES_PASSWORD=password -p 5432:5432 -d postgres
    ```
4. **start mongodb:**
    ```bash
    docker run --name mongodb -p 27017:27017 -d mongo
    ```

5.  **Running in Docker:**

    1.  Build the Docker images:
        ```bash
        # Navigate to each microservice directory, and run:
        cd inventory-service
        mvn spring-boot:build-image
        cd ../product-service
        mvn spring-boot:build-image
        cd ../api-gateway
        mvn spring-boot:build-image
        ```

    2.  Create the containers using docker compose:

        ```bash
        docker compose up -d
        ```

6.  **Running in Kubernetes (k8s):**

    1.  **Deploy the Services:**
        ```bash
        # Navigate to the Kubernetes configuration directory
        cd k8s/infrastructure/

        # Apply the configurations for the api-gateway
        kubectl apply -f api-gateway/api-gateway-deployment.yaml
        # Repeat the command for every deployment.yaml file inside k8s folder.
        ```

    2.  Check that all the pods are up and running:
        ```bash
        kubectl get pods -A
        ```

7.  **Access to the services:**

    *   Access to Zipkin: `http://localhost:9411/zipkin/`
    *   Access to Eureka: `http://localhost:8761/`
    *   Access to API Gateway: `http://localhost:8080/`
    *   Access to API Gateway (LoadBalancer): `http://localhost:8181/`
    *   Access to Product-service: `http://localhost:8082`

8. **Access to the API Documentation (Swagger)**
    * Access the API documentation for each service using the following URLs:
        *   Inventory Service: `http://localhost:8081/swagger-ui/index.html`
        *   Product Service: `http://localhost:8082/swagger-ui/index.html`
        * API Gateway: `http://localhost:8080/swagger-ui/index.html`
        * API Gateway: `http://localhost:8181/swagger-ui/index.html`

## Features

*   **Microservices Architecture:** The application is broken down into small, independent services that communicate with each other.
*   **Service Discovery:** Using Eureka to discover and communicate with other services.
*   **API Gateway:** Spring Cloud Gateway acts as a single point of entry for client requests, handling routing, security, and more.
*   **Database Integration:** The `inventory-service` uses PostgreSQL, the `product-service` uses MongoDB.
*   **Containerization:** Docker is used for building and running the services in containers.
*   **Distributed Tracing:** Using Zipkin for monitoring and troubleshooting.
*   **Circuit Breaker:** Implemented with Resilience4j and Spring Cloud Circuit Breaker for fault tolerance.
*   **Reactive Programming:** The API Gateway is built using Spring WebFlux for reactive request handling.
*   **Security:** The API Gateway uses Spring Security for enhanced control.
*   **API Documentation:** Interactive API documentation is available through Swagger (Springdoc OpenAPI).

## Communication Patterns

### Synchronous Communication

Synchronous communication is used in scenarios where one service needs an immediate response from another service. This is typically the case when using `Spring Cloud OpenFeign` to create a connection between services.

*   API Gateway uses `Spring Cloud OpenFeign` to create a connection between other services.

### Gateway

*   Acts as an entry point for all requests into the system.
*   Forwards requests to the downstream microservices.
*   Acts as a reverse proxy.

### Circuit Breaker

*   Implemented using `Resilience4j` and `Spring Cloud Circuit Breaker`.
*   Protects the system from cascading failures by monitoring service calls and temporarily blocking requests to failing services.
*   States: Open, Half-Open, Closed.

## Testing Circuit Breaker

*   To test, ensure that one of the services is unavailable.
*   Call the corresponding service.
*   You should see an error: "Service Unavailable, please try again later" (HTTP 503).
*   To test timeout and retry, simulate latency with `Thread.sleep()`.