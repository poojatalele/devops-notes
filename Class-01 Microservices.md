## Monolithic vs Microservices Architecture

### DNS Flow

**Flow:**

```
Browser → DNS → returns IP to browser → Browser → call to IP → Load Balancer → Authentication/Authorization → Backend Services
```

### Step-by-step:

1. **Browser Request**
    
    The browser initiates a request to a domain (e.g., `example.com`).
    
2. **DNS Resolution**
    
    DNS (Domain Name System) resolves the domain name into an **IP address**.
    
3. **Browser receives IP**
    
    The browser stores the IP in its cache and sends the request to it.
    
4. **Request to Load Balancer**
    
    The IP usually points to a **Load Balancer**, which:
    
    - Distributes incoming traffic among multiple servers.
    - Can perform **path-based routing** (e.g., `/login` → Login Service).
5. **Authentication & Authorization**
    - Performed via middleware or a dedicated service.
    - Often use **Redis** or **Memcached** for session or token caching.

---

### Load Balancer

### Responsibilities:

- **Distribute traffic** among backend servers.
- **Path-based routing:**
    
    Example:
    
    ```
    /v1/login → Login Service
    /v1/user  → User Service
    ```
    
- **Health checks** for active instances.
- Works at **Layer 4 (TCP)** or **Layer 7 (HTTP)** of the OSI model.

---

### Instance Identification

Each running service (instance) must be **uniquely identifiable**.

### Example:

- Instance IDs or container names like:
    
    ```
    login-service-1
    login-service-2
    ```
    
- Helps load balancers and service discovery tools route traffic properly.

---

### Microservice-to-Microservice Communication

Microservices communicate internally in two main ways:

1. **Synchronous Communication**
    - Real-time communication using **HTTP/gRPC APIs**.
    - Client sends a request and waits for a response.
    - Example:
        
        `Order Service → calls → Payment Service`
        
    - **Pros:**
        - Simple and direct.
        - Easier debugging.
    - **Cons:**
        - Tight coupling (if one service fails, the other may be affected).

---

### 2. **Asynchronous Communication**

- Services communicate via **message brokers**.
- Sender and receiver operate independently.
- Two main patterns:
    1. **Pub-Sub Model**
        - One service **publishes events**, others **subscribe** to them.
        - Example:
            
            User Service publishes `user.created`, Notification Service listens to it.
            
    - Uses **Kafka**, **RabbitMQ**, or **Google Pub/Sub**.
    - **No load balancer** needed — the broker manages distribution.
    
    2.  Queue Model
    - Tasks are pushed into a queue.
    - One service picks up a task and processes it, then removes it.
    - Ensures each task is processed exactly once.
    - Example: **Kafka**, **RabbitMQ**, **SQS**.
    - Again, **no load balancer** needed since brokers manage tasks.

---

### Service Discovery

### Purpose:

To dynamically identify which microservice instances are available.

### Responsibilities:

- Keeps track of which services are **alive**, **dead**, or **unreachable**.
- Updates routing information for communication between services.

### Tools:

- **Consul**
- **Eureka**
- **Zookeeper**
- **Kubernetes Service Discovery**

---

### API Gateway

### Role:

- Acts as the **entry point** for all client requests.
- Handles:
    - **Authentication**
    - **Authorization**
    - **Rate limiting**
    - **Routing to specific services**

### Example Flow:

```
Client → API Gateway → Auth Check → Service Routing → Response
```

**Benefits:**

- Centralized security.
- Unified logging and monitoring.
- Hides internal microservice structure from clients.

---

### Event Syncing

### Definition:

Tracking and syncing of various events or analytics data across services.

### Purpose:

- Maintain **system-wide consistency**.
- Provide data for **metrics**, **analytics**, or **machine learning pipelines**.
- Example:
    
    Logging every user interaction across different services.
    

---

### Logs (Centralized Logging)

Logs are critical for debugging and monitoring microservices.

### Why **Externalized Logs**?

Each microservice runs independently (often in containers), so logs must be centralized.

### What Logs Contain:

- Stack traces
- Service status
- Request/response details
- Performance metrics

### Popular Tools:

- **ELK Stack (Elasticsearch, Logstash, Kibana)**
- **Splunk**
- **Dynatrace**
- **Datadog**

---

### The Twelve-Factor App Principles

These are best practices for building modern, scalable microservices.

---

### **1. Codebase**

- Each microservice should have **its own codebase** (e.g., separate GitHub repo).
- Independent versioning and deployment.

---

### **2. Dependencies**

- Explicitly declare and isolate dependencies.

**Types:**

1. **Service dependencies** — dependencies between microservices.
2. **Component dependencies** — dependencies within the same service.
3. **Database dependencies** — dependencies on data storage.

---

### **3. Config**

- Separate **configuration** from code.
- Store environment-specific data (API keys, URLs) in environment variables or config files.

---

### **4. Backing Services**

- Treat external services (DB, cache, message queue) as **attached resources**.
- Easily replaceable without code changes.

---

### **5. Build, Release, Run**

- Keep build, release, and run stages separate.
    - **Build:** Compile code.
    - **Release:** Combine build + config.
    - **Run:** Execute the app.
- Each service should have its own **CI/CD pipeline**.

---

### **6. Processes**

- Execute the app as **one or more stateless processes**.
- Store persistent data in backing services (not in memory).

---

### **7. Port Binding**

- Services should be **self-contained** and expose APIs via a port.
    - Example: `localhost:8080` → Login Service.

---

### **8. Concurrency**

- Scale by **running multiple processes or containers**.
- Avoid shared state between them.

---

### **9. Disposability**

- Services should start and stop quickly.
- Enables easy scaling and graceful shutdowns.

---

### **10. Dev/Prod Parity**

- Keep development, staging, and production environments as similar as possible.

---

### **11. Logs**

- Treat logs as **event streams**.
- Send them to centralized systems (ELK, Splunk, etc.) instead of writing to local files.

---

### **12. Admin Processes**

- Run administrative tasks (e.g., DB migrations, data cleanup) as **one-off processes** in the same environment as the app.

---

### Summary Diagram

```
Browser → DNS → Load Balancer → API Gateway → Auth → Microservices
                     |                      |
                 Service Discovery        Logging (ELK)
                     |                      |
                  Message Broker ←→ Event Syncing

```
