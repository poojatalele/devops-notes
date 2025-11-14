# Deep Dive: Docker Containers

## 1. Defining a Container
A common misconception is that a container is a mini-virtual machine. **It is not.**
A container is simply a standard Linux process that has been isolated from the rest of the system using specific kernel features.
*   **Formula:** Process + Namespaces + Control Groups (cgroups) + Temporary Filesystem.
*   It uses the **Host's Kernel**. It does not boot its own OS.

---

## 2. Containers vs. Hypervisors (VMs)

### Virtual Machines (The Heavyweight)
A VM, managed by a Hypervisor, is a complete emulation of a computer.
*   **Boot Process:** BIOS → MBR → Bootloader → Kernel → Init.
*   **Resources:** Dedicated (and often wasted) CPU and RAM.
*   **Isolation:** Hard isolation (separate kernel).
*   **Startup:** Slow (minutes).

### Containers (The Lightweight)
Docker containers skip the entire hardware emulation and OS boot process.
*   **Boot Process:** None. It just starts the application process.
*   **Resources:** Shared with the host (efficient).
*   **Isolation:** Logical isolation (Namespaces).
*   **Startup:** Instant (milliseconds).

---

## 3. Internal Architecture: How Docker Starts
When you run `docker run`, a complex pipeline executes:

1.  **Docker Client:** Sends request to Daemon.
2.  **Containerd:** The high-level runtime that manages the container's lifecycle (pulling images, storage, networking).
3.  **Containerd-Shim:** A process that sits between `containerd` and the container, allowing `containerd` to restart without killing the container.
4.  **Runc:** The low-level runtime that actually interacts with the Kernel to create the container.
5.  **Process:** Your application (e.g., Nginx) starts.

**The exit flow:** The container remains alive only as long as its main process (PID 1) is running. If PID 1 stops, the container dies.

---

## 4. The Magic of Isolation (Namespaces)
Containers feel like separate machines because Linux **Namespaces** lie to the process about what it can see.

| Namespace | What it isolates |
| :--- | :--- |
| **PID** | Process IDs. The container sees itself as PID 1, but it's just PID 9999 on the host. |
| **NET** | Networking. Passing its own network stack (IP, localhost, ports). |
| **MNT** | Mounts. It sees a separate filesystem root. |
| **UTS** | Hostname. It has its own hostname. |
| **IPC** | Inter-Process Communication. |
| **USER** | User IDs. Root inside container can be a non-root user outside. |

---

## 5. Ephemeral Storage
Containers are **ephemeral** (temporary).
They consist of:
1.  **Read-Only Image Layers.**
2.  **A Thin Writable Layer.**

When a container is deleted, the **writable layer is destroyed**. Any data saved there (logs, database files) is lost forever unless you use **Volumes**.
