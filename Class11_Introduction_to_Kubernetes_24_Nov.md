# Introduction to Kubernetes (K8s)

## 1. What is Kubernetes?
Kubernetes is an open-source **Container Orchestration Platform**.
While Docker manages containers on a *single* machine, Kubernetes manages containers across *multiple* machines (a cluster).

### Core Features
*   **Self-Healing:** Restarts failed containers, replaces and reschedules them when nodes die.
*   **Auto-Scaling:** Automatically scales the number of containers based on CPU usage (HPA).
*   **Bin Packing:** Efficiently places containers on nodes with available resources.
*   **Load Balancing:** Distributes network traffic so that the deployment is stable.

---

## 2. Kubernetes Architecture
A cluster consists of two main types of servers:

### A. Control Plane (The "Brain")
Manages the cluster and makes global decisions.
1.  **API Server:** The front door. It handles all REST requests (CLI, UI, API). It creates/updates objects in etcd.
2.  **etcd:** The database. A consistent, distributed key-value store that holds the cluster state (secrets, config, nodes).
3.  **Scheduler:** The matchmaker. Checks for new pods and assigns them to the best-fit Worker Node based on resource availability.
4.  **Controller Manager:** The enforcer. Ensures the *Current State* matches the *Desired State* (e.g., "Keep 3 replicas running").

### B. Worker Nodes (The "Muscle")
Where your applications actually run.
1.  **Kubelet:** The primary node agent. It watches the API server and ensures that the containers described in the PodSpecs are running and healthy.
2.  **Kube-Proxy:** Maintains network rules on the node, allowing network communication to your Pods.
3.  **Container Runtime:** The software that actually runs the containers (e.g., containerd, CRI-O, Docker).

---

## 3. The Flow of a Deployment
What happens when you run `kubectl run nginx --image=nginx`?

1.  **Request:** `kubectl` sends a POST request to the **API Server**.
2.  **Store:** API Server validates the request and saves the object to **etcd**.
3.  **Schedule:** The **Scheduler** notices an "Unscheduled Web Pod". It checks available nodes and assigns it to "Node-1".
4.  **Assign:** API Server updates the record in **etcd** with the node assignment.
5.  **Execute:** The **Kubelet** on Node-1 sees it has been assigned a task.
6.  **Run:** Kubelet tells the **Runtime** to start the Nginx container.
7.  **Report:** Kubelet updates the API Server that the Pod is "Running".

---

## 4. Key Concepts

### Desired State Configuration
Kubernetes is **Declarative**. You don't tell it *how* to do things; you tell it *what* you want.
*   *You say:* "I want 3 Nginx pods."
*   *K8s says:* "I see 0. I will create 3."
*   *Later:* If one crashes, K8s says: "I see 2. Desired is 3. I will create 1 more."

### The Pod
The smallest deployable unit in K8s. A wrapper around one or more containers. K8s does not run containers directly; it runs Pods.

---

## 5. Basic Commands
*   `kubectl get nodes` - List all servers in the cluster.
*   `kubectl run <name> --image=<image>` - Create a single pod.
*   `kubectl describe pod <name>` - detailed debugging info (events, errors).
*   `kubectl logs <name>` - View container logs.
