# Docker Networking Deep Dive

## 1. Networking Fundamentals

### Essential Concepts
*   **IP Address:** Logical address of a device (e.g., `192.168.1.5`).
*   **MAC Address:** Physical address of the network card (NIC).
*   **Subnetting:** Breaking a large network into smaller segments (e.g., `/24` gives 256 IPs).
*   **NAT (Network Address Translation):** Allows private IPs to talk to the public internet.
    *   **SNAT:** Source NAT (Outbound traffic).
    *   **DNAT:** Destination NAT (Inbound traffic / Port Forwarding).

---

## 2. Docker's Internal Network
When you install Docker, it creates a default bridge network interface called `docker0`.
*   **Default Range:** Usually `172.17.0.0/16`.
*   **Function:** It acts as a virtual switch connecting all containers on the host.

### How Containers Connect
Every container gets:
1.  A valid **Private IP** (e.g., `172.17.0.2`).
2.  A virtual ethernet interface (`eth0`) inside the container.
3.  A corresponding `veth` pair on the host machine connected to the `docker0` bridge.

---

## 3. Docker Network Drivers

| Driver | Description | Command |
| :--- | :--- | :--- |
| **Bridge** | **Default.** Private internal network. Containers can talk to each other. Isolated from host. | `docker run --net bridge` |
| **Host** | Removes isolation. Container uses Host's IP and ports directly. | `docker run --net host` |
| **None** | No networking. Loopback only. Secure sandbox. | `docker run --net none` |
| **Overlay** | Used for multi-host networking (Docker Swarm/Kubernetes). | `docker network create -d overlay ...` |

---

## 4. Container Communication & DNS

### The "Default" Bridge Limitation
On the default `bridge`, containers can communicate by **IP address only**. They cannot resolve each other by name.

### Custom Bridge (User-Defined)
If you create a custom network:
```bash
docker network create my-app-net
```
And run containers on it:
```bash
docker run --name db --net my-app-net mysql
docker run --name web --net my-app-net nginx
```
**Magic happens:** The `web` container can reach the `db` container simply by using the hostname `db`. Docker provides built-in **DNS resolution** for user-defined networks.

---

## 5. Port Mapping
Since containers have private IPs, they are unreachable from the outside world by default. We use **Port Mapping** to expose them.

**Syntax:** `-p <HostPort>:<ContainerPort>`
```bash
docker run -p 8080:80 nginx
```
*   Traffic hitting `localhost:8080` is forwarded to `container:80`.
*   This uses **DNAT** (Destination NAT).

---

## 6. Practical Commands
```bash
# List networks
docker network ls

# Inspect a network (see connected containers and IPs)
docker network inspect bridge

# Create a dedicated network
docker network create my-net

# Connect a running container to a network
docker network connect my-net my-container
```
