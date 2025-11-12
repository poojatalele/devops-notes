# Mastery of Dockerfiles and Building Images

## 1. What is a Dockerfile?
A **Dockerfile** is a text script containing a series of instructions. Docker reads this file to build a custom Image automatically. Each instruction creates a new layer in the image stack.

---

## 2. Key Dockerfile Instructions

### `FROM`
**Purpose:** Sets the Base Image.
**Rule:** Must be the first non-comment instruction.
```dockerfile
FROM python:3.9-slim
```
*Tip: Use minimal images like `alpine` or `slim` versions to reduce size.*

### `RUN`
**Purpose:** Executes commands *during the build process*.
**Use Case:** Installing packages, libraries, or setting up the environment.
```dockerfile
RUN apt-get update && apt-get install -y curl
```
*Note: This runs once, when you build the image.*

### `CMD` vs. `ENTRYPOINT`
These define what happens when the container *starts*.

| Instruction | Behavior | Overridable? |
| :--- | :--- | :--- |
| **CMD** | Default command/arguments. | Yes, easily overridden by adding arguments after `docker run`. |
| **ENTRYPOINT** | The main executable. | No, difficult to override. It creates a fixed binary behavior. |

**Common Pattern:**
Use `ENTRYPOINT` for the executable and `CMD` for default arguments.
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```
*Effect: `docker run image` executes `python app.py`. `docker run image script.py` executes `python script.py`.*

### `COPY` vs. `ADD`
**Purpose:** Moving files from your host into the image.

*   **COPY:** Simple file copy. Preferred/Best Practice.
    ```dockerfile
    COPY requirements.txt /app/
    ```
*   **ADD:** Advanced copy. Can extract `.tar` files automatically and download from URLs. Use only if necessary.

### `WORKDIR`
**Purpose:** Sets the working directory for subsequent instructions.
```dockerfile
WORKDIR /app
```
*Acts like `cd /app` inside the image.*

---

## 3. The Build Process
To create an image from a Dockerfile, use the `docker build` command.

**Syntax:**
```bash
docker build -t <image-name>:<tag> <context-path>
```

**Example:**
```bash
docker build -t my-app:v1 .
```
*   `-t`: Tags the image (names it).
*   `.`: Specifies the current directory as the build context.

**Internal Workflow:**
1.  Docker reads the Dockerfile.
2.  It sends the "Context" (files in the folder) to the Daemon.
3.  It executes instructions line-by-line.
4.  Each instruction creates a cached layer.
5.  The final result is the Image ID.

---

## 4. Docker Build vs. Run Time

It is critical to distinguish when commands happen:

*   **Build Time:** When you run `docker build`.
    *   Commands: `FROM`, `RUN`, `COPY`, `WORKDIR`.
    *   *Example: Installing Python dependencies.*

*   **Runtime:** When you run `docker run`.
    *   Commands: `CMD`, `ENTRYPOINT`.
    *   *Example: Starting the web server.*

---

## 5. Image Inspection
Once built, you can inspect your images:

*   **View Layers:** `docker history <image_name>`
*   **View Details:** `docker inspect <image_name>`
*   **Scan Security:** `docker scan <image_name>`
