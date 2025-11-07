# Docker Basics on AWS EC2

## 1. Getting Access to Your Server
Once you've launched an EC2 instance, the first step is to remotely connect to it using SSH.

### Step-by-Step Guide
1.  Navigate to the AWS Console, select your running instance, and hit **Connect**.
2.  Select the **SSH Client** tab.
3.  You'll see a pre-generated SSH command (something like `ssh -i "key.pem" ubuntu@ec2...`). Copy it.
4.  On your own computer, open a terminal and `cd` into the folder where you saved your `.pem` key file.
5.  Paste and run the command:
    ```bash
    ssh -i your-key.pem ubuntu@ec2-xx-xx-xx-xx.compute.amazonaws.com
    ```
6.  You are now logged into the remote Ubuntu server!

---

## 2. Your First Container
Let's verify everything is working by running the classic "Hello World" example.
Run the following command:
```bash
docker run hello-world
```

If Docker isn't installed, you'll see a "command not found" error. If it is installed, Docker will spring into action, download the necessary files, and run them.

**What you'll see:**
```
Unable to find image 'hello-world:latest' locally
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

## 3. Decoding the Output
The output above actually tells us a lot about how Docker behaves.

### Does `docker run` reuse containers?
No. Every time you type `docker run <image>`, Docker spins up a **brand new container**. It doesn't restart an old one; it creates a fresh instance from scratch.

---

## 4. The Blueprint: Images
You can't have a container without an image.
*   **The Concept:** Think of an **Image** as a Class (the blueprint) and a **Container** as an Object (the actual instance).
*   **Storage:** Images can live on your hard drive (**Local**) or in a cloud registry like Docker Hub (**Remote**).

When you ran the command earlier, Docker looked for the image on your computer. It complained that it couldn't find it locally, so it automatically reached out to the internet (Docker Hub) and downloaded ("pulled") it for you.

---

## 5. The Container Concept
What exactly is a container?
*   It's like a **lightweight Virtual Machine**.
*   It has its own isolated file system and processes.
*   **The Magic:** Unlike a VM, it doesn't need its own full Operating System. It shares the **Kernel** of the host machine, which makes it incredibly fast and efficient.

---

## 6. Under the Hood: Client & Daemon
When you type a command, two main components are working together:

### The Docker Client
This is the CLI tool (`docker` command) you type into. It's just a messenger. It sends your specific instructions to the engine.

### The Docker Daemon
This is the heavy lifter. It runs quietly in the background (as a service) and handles all the hard work:
*   Downloading images.
*   Starting and stopping containers.
*   Managing the entire lifecycle.

**The Workflow:**
The Client says "Run this." -> The Daemon says "On it," grabs the image, starts the container, and sends the output back to the Client.

---

## 7. Linux Background Services
In the Linux world, background services are often called **daemons**.
You can check if the Docker daemon is actually running with:
```bash
systemctl status docker
```

---

## 8. The Two Pieces of Docker
So, when you install Docker, you are really installing two things:
1.  **The Client:** The interface you talk to.
2.  **The Daemon:** The engine that does the work.

You can confirm this by running:
```bash
docker version
```
(You'll see separate entries for Client and Server/Engine.)

---

## 9. Stepping Inside a Container
Running a container is cool, but sometimes you want to actually "go inside" it and run commands.
We do this with the interactive flag:
```bash
docker run -it ubuntu bash
```

**Breaking it down:**
*   `-i`: Interactive (keep the input stream open).
*   `-t`: Allocate a pseudo-Terminal (so it looks like a regular shell).
*   `ubuntu`: The image we want to use.
*   `bash`: The command we want to run inside (the shell).

**Notice the change:**
Your command prompt will switch from `ubuntu@ip-172...` to something like `root@f21106bca534:/#`. You are now inside the container, and that random string is your **Container ID**.

---

## 10. Process Isolation
While inside that container, try checking the running processes:
```bash
ps -ef
```
You will notice the list is very short!
This is because the container is **isolated**. It can't see the hundreds of processes running on the host machine; it can only see what's running inside its own little bubble.

---

## 11. The Big Picture
Here is the full lifecycle of a Docker command:
1.  User types `docker run hello-world`.
2.  **Client** forwards the request to the **Daemon**.
3.  **Daemon** checks its local cache for the image.
4.  If missing, it **pulls** it from the Docker Hub registry.
5.  **Daemon** creates a new container from that image.
6.  **Daemon** launches the container executable.
7.  The output is streamed back to your **Client** terminal.

---

## 12. Key Takeaways
*   **Images** are the blueprints; **Containers** are the running instances.
*   `docker run` always creates a new container.
*   The **Daemon** does the actual work; the **Client** is just how you talk to it.
*   Use `docker run -it` to open an interactive shell inside a container.
*   Containers are isolated environments with their own process tree.
