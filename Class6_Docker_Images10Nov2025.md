# Docker vs. Virtualization Architecture

## 1. Bare Metal Architecture
In a traditional **Bare Metal** setup, the Operating System runs directly on the physical hardware.
*   **Stack:** Hardware → Kernel → Libraries → Applications.
*   **Pros:** Maximum performance, no overhead.
*   **Cons:** Inflexible. Usually limited to one OS and difficult to isolate multiple apps efficiently.

---

## 2. Virtual Machines (Hypervisor-Based)
Virtualization allows running multiple OS instances on a single physical server by using a **Hypervisor**.

### What is a Hypervisor?
Software that abstracts the physical hardware, allowing it to be shared among multiple Virtual Machines (VMs).

*   **Type 1 (Bare Metal):** Installs directly on hardware (e.g., VMware ESXi, Hyper-V). Efficient.
*   **Type 2 (Hosted):** Installs on an existing OS (e.g., VirtualBox). Good for personal testing.

### The Problem with VMs
Each VM is a complete computer. It requires:
1.  Dedicated Hardware allocation.
2.  A full **Guest Operating System**.
3.  Its own Kernel, Binaries, and Boot process.

**Why is it heavy?**
Booting a VM is like turning on a physical computer. It goes through BIOS, Bootloader, Kernel load, and Service initialization. This takes minutes and consumes GBs of RAM.

---

## 3. Container Architecture (Docker)
Docker creates isolated environments called **Containers** that share the **Host OS Kernel**. They do **not** have their own kernel or full OS.

### The Flow
```
Hardware → Host Kernel → Docker Engine → Containers
```

### Why Docker is Lightweight
*   **No OS Boot:** Containers start as essentially just another process on the host.
*   **Shared Kernel:** All containers use the same Linux kernel as the host.
*   **Minimal Footprint:** Only the application and its specific libraries are packaged.
*   **Speed:** Startup time is measured in **milliseconds**, not minutes.

---

## 4. Docker Run Lifecycle
Understanding exactly what happens when `docker run nginx` is executed:

1.  **User Command:** Docker Client sends API request to Daemon.
2.  **Image Check:** Daemon checks the local cache. If missing, it downloads (pulls) layers from Docker Hub.
3.  **Container Creation:**
    *   Allocates a read-write filesystem layer.
    *   Creates a network interface.
    *   Isolates the process namespaces.
4.  **Startup:** The application (e.g., Nginx) starts immediately. There is no BIOS or OS boot sequence.

---

## 5. Essential Docker Commands

| Action | Command |
| :--- | :--- |
| **List Running** | `docker ps` |
| **List All (incl. stopped)** | `docker ps -a` |
| **Run (Foreground)** | `docker run nginx` |
| **Run (Background/Detached)** | `docker run -d nginx` |
| **Enter Container** | `docker exec -it <container_id> bash` |
| **Stop Container** | `docker stop <container_id>` |
| **Remove Container** | `docker rm <container_id>` |
| **Remove Image** | `docker rmi <image_id>` |

### Creating a Custom Image
You can save a container's state as a new image:
```bash
docker commit -m "added new tools" <container_id> my-new-image
```
