# HNG12 DevOps Stage4 Task

This Repo Contains the code for a microservice application comprising of several components communicating to each other. In other words, this is an example of microservice. These microservices are written in different languages.

The app itself is a simple TODO app that additionally authenticates users.

## Components

1. [Frontend](/frontend) part is a Javascript application, provides UI. Created with [VueJS](http://vuejs.org)
2. [Auth API](/auth-api) is written in Go and provides authorization functionality. Generates JWT tokens to be used with other APIs.
3. [TODOs API](/todos-api) is written with NodeJS, provides CRUD functionality ove user's todo records. Also, it logs "create" and "delete" operations to Redis queue, so they can be later processed by [Log Message Processor](/log-message-processor).
4. [Users API](/users-api) is a Spring Boot project written in Java. Provides user profiles. Does not provide full CRUD for simplicity, just getting a single user and all users.
5. [Log Message Processor](/log-message-processor) is a very short queue processor written in Python. It's sole purpose is to read messages from Redis queue and print them to stdout


The diagram describes the various components and their interactions.
![microservice-app-example](https://user-images.githubusercontent.com/1905821/34918427-a931d84e-f952-11e7-85a0-ace34a2e8edb.png)

Note: 3 different login details are provided in the .env file 

## License

MIT




# **Traefik Setup for Local Development** 🚀

This guide will walk you through setting up **Traefik** as a reverse proxy for your local development environment. We'll configure **networking**, **SSL certificates**, **Docker Compose services**, and **Traefik routing**.

---

## **1. Network Configuration** 🌐

First, we need to create a **Docker network** to ensure that all our services communicate seamlessly.

### **Create the network:**
```sh
docker network create localhost_net
```
### **Define the network in `docker-compose.yml`**
```yaml
networks:
  localhost_net:
    external: true
volumes:
  traefik-data:
    driver: local
```
## 2️⃣ Domain Configuration 🌍

We will expose our application using various subdomains. Modify your `hosts` file to map these domains to your local machine (`127.0.0.1`).

### **Domains:**
| **Service**         | **URL**                                          |
|---------------------|------------------------------------------------|
| **Frontend**       | [https://loclahost.com](https://localhost.com)       |
| **Auth API**       | [https://auth.localhost.com](https://auth.localhost.com) or [https://localhost.com/api/auth](https://localhost.com/api/auth) |
| **Todos API**      | [https://localhost.domain.com](https://todos.localhost.com) or [https://localhost.com/api/todos](https://localhost.com/api/todos) |
| **Users API**      | [https://users.localhost.com](https://users.localhost.com) or [https://localhost.com/api/users](https://localhost.com/api/users) |
| **Traefik Dashboard** | [https://proxy.localhost](https://proxy.localhost) |

### **Modify Your Hosts File** 🖥️  
Edit your `hosts` file to map these domains to `127.0.0.1`:

#### **Mac/Linux (`/etc/hosts`)**  
#### **Windows (`C:\Windows\System32\drivers\etc\hosts`)**

```sh
127.0.0.1  localhost.com auth.localhost.com todos.localhost.com users.localhost.com proxy.localhost
```
## 3️⃣ Traefik Service Configuration ⚙️  

Define Traefik as a reverse proxy in `docker-compose.yml`.  

### **Traefik Service in `docker-compose.yml`**  

```yaml
services:
  traefik:
    image: traefik:v3.3
    command: "--configFile=./config/traefik.yml"
    ports:
      - "8085:8085"
      - "443:443"
    volumes:
      - "./traefik_data:/etc/traefik"
      - "./config/configuration.yml:/config/traefik.yml:ro"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./localhost.crt:/certs/localhost.crt:ro"
      - "./localhost.key:/certs/localhost.key:ro"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`proxy.localhost`)"
      - "traefik.http.routers.traefik.entrypoints=websecure"
      - "traefik.http.services.traefik.loadbalancer.server.port=8085"
    networks:
      - localhost_net

networks:
  localhost_net:
    external: true

volumes:
  traefik_data:
```
## 4️⃣ Local SSL Certificate Setup 🔒  

Generate a self-signed SSL certificate for HTTPS.  

### **Generate SSL Certificate**  

Run the following command at the **project root**:  

```sh
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout localhost.key -out localhost.crt
```
### **Generated Files:**  
- **`localhost.crt`** → SSL certificate  
- **`localhost.key`** → Private key  
## 5️⃣ Traefik Configuration File 🛠️  

Create a `config/configuration.yml` file with the following content:  

```yaml
global:
  checkNewVersion: true
  sendAnonymousUsage: false

serversTransport:
  insecureSkipVerify: true

entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https

  websecure:
    address: ":443"
    http:
      tls:
        domains:
          - main: "localhost"
            sans:
              - "*.localhost"

  traefik:
    address: ":8085"

providers:
  docker:
    watch: true
    network: localhost_net
    exposedByDefault: false

api:
  dashboard: true
  insecure: true

log:
  level: INFO

tls:
  certificates:
    - certFile: "../certs/localhost.crt"
      keyFile: "../certs/localhost.key"
```
## 6️⃣ Define Application Services 🚀  

Each service must be proxied by Traefik.  

```yaml
services:
  traefik:
    image: traefik:v3.3
    command: "--configFile=./config/traefik.yml"
    ports:
      - "8085:8085"
      - "443:443"
    volumes:
      - "./traefik_data:/etc/traefik"
      - "./config/configuration.yml:/config/traefik.yml:ro"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./localhost.crt:/certs/localhost.crt:ro"
      - "./localhost.key:/certs/localhost.key:ro"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`proxy.localhost`)"
      - "traefik.http.routers.traefik.entrypoints=websecure"
      - "traefik.http.services.traefik.loadbalancer.server.port=8085"
    networks:
      - localhost_net

  app:
    image: app
    build: ./frontend
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app.rule=Host(`localhost`)"
      - "traefik.http.routers.app.entrypoints=websecure"
      - "traefik.http.services.app.loadbalancer.server.port=8080"
    networks:
      - localhost_net
    ports:
      - "8080:8080"
    depends_on:
      - auth-api
      - todos-api
      - users-api
    environment:
      - PORT=${PORT}
      - AUTH_API_ADDRESS=${AUTH_API_ADDRESS}
      - TODOS_API_ADDRESS=${TODOS_API_ADDRESS}

  auth-api:
    image: auth-api
    build: ./auth-api
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.auth.rule=Host(`auth.localhost`)"
      - "traefik.http.routers.auth.entrypoints=websecure"
      - "traefik.http.services.auth.loadbalancer.server.port=8081"
    networks:
      - localhost_net
    ports:
      - "8081:8081"
    depends_on:
      - redis
    environment:
      - AUTH_API_PORT=${AUTH_API_PORT}
      - JWT_SECRET=${JWT_SECRET}
      - USERS_API_ADDRESS=${USERS_API_ADDRESS}

networks:
  localhost_net:
    external: true

volumes:
  traefik_data:
    driver: local
```
## 7️⃣ Start Services 🚀  

Run this command to start all services:  

```sh
docker-compose up -d
```
## 8️⃣ Access Services 🔗  

| Service         | URL                               |
|----------------|----------------------------------|
| **Frontend**   | [https://localhost.com](https://localhost.com) |
| **Auth API**   | [https://auth.localhost.com](https://auth.localhost.com) |
| **Todos API**  | [https://todos.localhost.com](https://todos.localhost.com) |
| **Users API**  | [https://users.localhost.com](https://users.localhost.com) |
| **Traefik Dashboard** | [https://proxy.localhost](https://proxy.localhost) |

---

## 🎉 Conclusion  
Now you have a fully configured **Traefik reverse proxy** for local development with **HTTPS support** and **service routing**. Enjoy! 🚀🎉  

