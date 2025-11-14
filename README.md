# Docker Basics Nginx Practical

## Lab 5A – Docker Basics Nginx Practical

**Step-by-step student guide**

---

## Before You Start

**You should already have:**

* Docker Desktop / Docker Engine installed and running
* Basic ability to type commands in **Terminal** (macOS/Linux) or **Command Prompt / PowerShell** (Windows)

**Key ideas you’ll meet:**

* **Image** – a “blueprint” or template for a container
* **Container** – a running instance created from an image
* **Port mapping** – allowing your browser to talk to the app in the container
* **Volume** – a folder on your computer “linked” into the container so data can persist

---

## Step 1 – Pull the Nginx Image

**Goal:** Download the official Nginx image from Docker Hub to your computer.

1. Open your **terminal / command prompt**.

2. Type this command:

   ```bash
   docker pull nginx
   ```

   **What this does:**

   * `docker pull` tells Docker to download an image.
   * `nginx` is the name of the image (an official web server image).

3. After it finishes, check that the image is stored locally:

   ```bash
   docker images
   ```

   **You should see:**
   A table with a row where **REPOSITORY** is `nginx`.
   That means the image is now available for you to use.

---

## Step 2 – Run Nginx as a Container

**Goal:** Start a container (a running instance) from the Nginx image.

1. Run this command:

   ```bash
   docker run -d --name my-nginx-container nginx
   ```

   **Explanation of options:**

   * `docker run` – create and start a new container.
   * `-d` – run in **detached mode** (in the background, so your terminal is free).
   * `--name my-nginx-container` – give the container a friendly name.
   * `nginx` – use the `nginx` image you pulled earlier.

2. Check that it’s running:

   ```bash
   docker ps
   ```

   **You should see:**
   A table showing a container with **NAMES** column `my-nginx-container` and **STATUS** like `Up ...`.

**Concept check:**

* Image = template.
* Container = running process created from the template.

---

## Step 3 – Manage the Container Lifecycle

**Goal:** Learn how to **stop**, **start**, and **restart** a container.

### 3.1 Stop the container

1. Run:

   ```bash
   docker stop my-nginx-container
   ```

   **What this does:**

   * Sends a stop signal to gracefully shut down the container.

2. Check active containers:

   ```bash
   docker ps
   ```

   You **should NOT** see `my-nginx-container` here (because it’s stopped).

3. List **all** containers (including stopped):

   ```bash
   docker ps -a
   ```

   You **should** see `my-nginx-container` with STATUS like `Exited`.

---

### 3.2 Start the container again

1. Run:

   ```bash
   docker start my-nginx-container
   ```

2. Verify it’s running:

   ```bash
   docker ps
   ```

   Now you should see it again with STATUS `Up ...`.

---

### 3.3 Restart the container

1. Run:

   ```bash
   docker restart my-nginx-container
   ```

   This is equivalent to stop + start in one command.

2. Check:

   ```bash
   docker ps
   ```

   It should still be **Up**, maybe with a new uptime.

**Concept check:**

* `stop` – pause / shut down the container process.
* `start` – bring a stopped container back.
* `restart` – stop then start in one go.

---

## Step 4 – Expose Nginx as a Web App (Port Mapping)

Right now your container is running, but we’re not controlling which **host port** it uses. We’ll start fresh and map ports properly.

**Goal:** Make Nginx accessible from your browser at `http://localhost:8080`.

### 4.1 Remove the old container

1. Stop and remove the original container to avoid conflicts:

   ```bash
   docker stop my-nginx-container
   docker rm my-nginx-container
   ```

   **Note:** `rm` deletes the container, but **not** the image.

---

### 4.2 Run Nginx with port mapping

1. Start a new container with port mapping:

   ```bash
   docker run -d --name my-web-nginx -p 8080:80 nginx
   ```

   **Explanation of `-p 8080:80`:**

   * `80` is the port **inside** the container (Nginx’s default HTTP port).
   * `8080` is the port on **your computer (host)**.
   * So when your browser goes to `localhost:8080`, Docker forwards traffic to port `80` inside the container.

2. Verify:

   ```bash
   docker ps
   ```

   Check:

   * The container name: `my-web-nginx`
   * The **PORTS** column should show something like `0.0.0.0:8080->80/tcp`.

3. Open your browser and go to:

   ```text
   http://localhost:8080
   ```

   You should see the default **Nginx welcome page**.

**Concept check:**

* Port mapping makes a service inside the container available outside via a host port.

---

## Step 5 – Mount a Volume for Persistent Custom Content

**Goal:** Serve your **own HTML page** from Nginx, stored in a folder on your computer (so you can edit it without rebuilding the container).

### 5.1 Stop and remove the current web container

1. Run:

   ```bash
   docker stop my-web-nginx
   docker rm my-web-nginx
   ```

---

### 5.2 Create a folder and custom `index.html`

1. In your current working directory, create a folder:

   ```bash
   mkdir nginx-html
   ```

2. Inside that folder, create `index.html` with your own content.

* **Windows (Command Prompt):**

  ```cmd
  echo ^<!DOCTYPE html^>^<html^>^<head^>^<title^>My Custom Nginx Page^</title^>^</head^>^<body^>^<h1^>Hello from Docker!^</h1^>^<p^>This is my custom Nginx content.^</p^>^</body^>^</html^> > nginx-html\index.html
  ```

* **macOS / Linux (Bash/Zsh):**

  ```bash
  echo '<!DOCTYPE html><html><head><title>My Custom Nginx Page</title></head><body><h1>Hello from Docker!</h1><p>This is my custom Nginx content.</p></body></html>' > nginx-html/index.html
  ```

**Concept:**
This file is on your **host machine**, not inside Docker yet.

---

### 5.3 Run Nginx with a volume mount

Now we tell Docker: “use my local `nginx-html` folder as Nginx’s web root”.

1. First, find the **absolute path** to your `nginx-html` directory, e.g.:

* Windows example: `C:\Users\<YourUser>\Desktop\nginx-html`
* macOS/Linux example: `/Users/<YourUser>/nginx-html`

2. Run the container with a volume:

   ```bash
   docker run -d --name my-persistent-nginx -p 8080:80 -v <PATH_TO_YOUR_NGINX_HTML_DIRECTORY>:/usr/share/nginx/html nginx
   ```

   Replace `<PATH_TO_YOUR_NGINX_HTML_DIRECTORY>` with your actual path, e.g.:

   * **Windows:**

     ```bash
     docker run -d --name my-persistent-nginx -p 8080:80 -v C:\Users\<YourUser>\Desktop\nginx-html:/usr/share/nginx/html nginx
     ```

   * **macOS/Linux:**

     ```bash
     docker run -d --name my-persistent-nginx -p /Users/<YourUser>/nginx-html:/usr/share/nginx/html nginx
     ```

   **Explanation of `-v host_path:container_path`:**

   * `host_path` – folder on your computer.
   * `container_path` – folder inside the container.
   * Here, `/usr/share/nginx/html` is Nginx’s default web content folder.

3. Check that it’s running:

   ```bash
   docker ps
   ```

4. In your browser, go to:

   ```text
   http://localhost:8080
   ```

   You should now see your **custom HTML page** (“Hello from Docker!”), not the default Nginx page.

**Concept check:**

* A **volume mount** links a host folder into the container.
* Changes you make to files in `nginx-html` will immediately affect what Nginx serves (persistent and easy to edit).

---

## Step 6 – Clean Up

**Goal:** Remove everything created in this lab so your system is clean.

1. Stop the running container:

   ```bash
   docker stop my-persistent-nginx
   ```

2. Remove the container:

   ```bash
   docker rm my-persistent-nginx
   ```

3. Remove the Nginx image:

   ```bash
   docker rmi nginx
   ```

4. Check that no containers or images from this lab remain:

   ```bash
   docker ps -a
   docker images
   ```

   * `docker ps -a` should not list `my-persistent-nginx` or the earlier containers.
   * `docker images` should not list `nginx` (unless you have other tags).

---

## Quick Recap for Students

By the end of this lab, you have learned how to:

1. **Pull** an image from Docker Hub (`docker pull`).
2. **Run** a container and see it in `docker ps`.
3. **Stop**, **start**, and **restart** containers (lifecycle management).
4. Use **port mapping** (`-p 8080:80`) to access a web server in the container.
5. Use **volumes** (`-v host:container`) to serve your own HTML content from Nginx.
6. **Clean up** containers and images when you are done (`docker stop`, `docker rm`, `docker rmi`).

If you’d like, I can next turn this into a **GitHub-style README** or a **student lab sheet** with checkboxes and spaces for screenshots.

