# Docker Volumes & Storage

## 1. The Persistence Problem
By design, Docker containers are ephemeral.
*   **Read-Only Layers:** Provide the OS and code.
*   **Writable Layer:** Stores runtime changes.
*   **Issue:** When you `docker rm` a container, the writable layer is deleted. All data inside `/var/lib/mysql` or `/app/data` vanishes.

**Solution:** Docker Volumes. These are storage locations outside the container's writable layer, managed by Docker, and immune to container deletion.

---

## 2. Storage Types

### A. Anonymous Volumes
Created when you don't specify a name.
*   **Syntax:** `docker run -v /data nginx`
*   **Storage:** `/var/lib/docker/volumes/random-hash/_data`
*   **Use Case:** Quick, temporary storage where you don't care about the location name.

### B. Named Volumes (Recommended)
Explicitly named and managed by Docker.
*   **Create:** `docker volume create my-data`
*   **Run:** `docker run -v my-data:/data nginx`
*   **Behavior:** Even if the container is destroyed, `my-data` persists. You can attach it to a new container to restore state.
*   **Use Case:** Databases (Postgres, MySQL), Application state.

### C. Bind Mounts
Directly linking a folder on your Host OS to the Container.
*   **Syntax:** `docker run -v /home/user/project:/app nginx`
*   **Behavior:** Changes on the host appear instantly in the container, and vice versa.
*   **Use Case:** Development (Code reloading), Injecting Config files.

### D. tmpfs Mounts
Stored **only in RAM**. Never written to disk.
*   **Syntax:** `--tmpfs /app/cache`
*   **Use Case:** Secrets, High-speed caching, Security-sensitive data.

---

## 3. Managing Volumes

| Command | Description |
| :--- | :--- |
| `docker volume create <name>` | Create a new named volume. |
| `docker volume ls` | List all volumes. |
| `docker volume inspect <name>` | See details (creation time, mount path). |
| `docker volume rm <name>` | Delete a volume (must not be in use). |
| `docker volume prune` | Delete all unused volumes. |

---

## 4. Key Takeaways
1.  **Stop vs Remove:** Stopping a container (`docker stop`) preserves data. Removing it (`docker rm`) destroys the writable layer.
2.  **Location:** Docker volumes live in `/var/lib/docker/volumes/`.
3.  **Sharing:** Multiple containers can mount the same volume simultaneously, enabling data sharing.
