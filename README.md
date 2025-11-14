# Docker Basics Nginx Practical

## Lab 5A – Docker Basics Nginx Practical

**Step-by-step student guide**

---
## Before You Start

1. **Make sure Docker is installed and running**

   * On **Windows / macOS**: Open **Docker Desktop** and ensure it says “Running”.
   * On **Linux**: Ensure the Docker service is running (`sudo systemctl status docker` if you know how).

2. **Open a terminal**

   * **Windows**: PowerShell or Command Prompt
   * **macOS**: Terminal
   * **Linux**: Any terminal (GNOME Terminal, Konsole, etc.)

3. **Check that Docker works:**

   ```bash
   docker --version
   docker info
   ```

   * If you see version information and no big error messages, you’re ready.

---

## Step 1: Pulling an Nginx Image

### 1.1 Pull the official Nginx image

In your terminal, type:

```bash
docker pull nginx
```

**What this does:**

* `docker pull` tells Docker to **download an image** from Docker Hub (the public Docker image repository).
* `nginx` is the name of the image (the official Nginx web server).

**What you should see:**

* Several lines showing “Downloading”, “Pull complete”, and finally something like:

  ```text
  Status: Downloaded newer image for nginx:latest
  ```

### 1.2 Verify that the image is downloaded

Run:

```bash
docker images
```

**What this does:**

* Lists all Docker images stored on your machine.

**What to look for:**

* A row where `REPOSITORY` is `nginx`, `TAG` is usually `latest`.
* Example:

  ```text
  REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
  nginx        latest    a23c15a5b5d0   2 weeks ago    188MB
  ```

If you see `nginx` there, Step 1 is successful.

---

## Step 2: Running Nginx as a Container

### 2.1 Start a container from the Nginx image

Run:

```bash
docker run -d --name my-nginx-container nginx
```

**Explanation of the command:**

* `docker run` – create and start a container from an image.
* `-d` – detached mode (container runs in the background, terminal is free for other commands).
* `--name my-nginx-container` – gives the container a friendly name instead of a random one.
* `nginx` – the image to use.

If successful, Docker will output a **long string** (the container ID).

### 2.2 Check that the container is running

Run:

```bash
docker ps
```

**What this does:**

* Shows all **running** containers.

**What to look for:**

* A row where `NAMES` is `my-nginx-container`.
* `STATUS` should be something like `Up 5 seconds`.

If you see `my-nginx-container` with `Up ...`, your container is running correctly.

---

## Step 3: Managing the Container Lifecycle

This step teaches you how to **stop**, **start**, and **restart** a container.

### 3.1 Stop the container

Run:

```bash
docker stop my-nginx-container
```

**What this does:**

* Politely asks the container to stop running.

You should see the container name printed back:

```text
my-nginx-container
```

### 3.2 Check that it has stopped

Run:

```bash
docker ps
```

* You should **not** see `my-nginx-container` now (because it’s stopped).

Then run:

```bash
docker ps -a
```

* `-a` shows **all containers**, including stopped ones.
* Now you should see `my-nginx-container` with a `STATUS` like `Exited (0) ...`.

### 3.3 Start the container again

Run:

```bash
docker start my-nginx-container
```

Then verify:

```bash
docker ps
```

* You should see `my-nginx-container` with `STATUS` starting with `Up`.

### 3.4 Restart the container

Restart = stop + start in one command.

Run:

```bash
docker restart my-nginx-container
```

Then check again:

```bash
docker ps
```

* You should still see it running (status resets to `Up a few seconds`).

At this point, you know how to control the **lifecycle** of a container.

---

## Step 4: Exposing Nginx as a Web App

Now you will run Nginx so that it serves a web page that you can access from your browser.

### 4.1 Remove the previous container (to avoid port conflicts)

First, stop it (if it’s still running):

```bash
docker stop my-nginx-container
```

Then remove it:

```bash
docker rm my-nginx-container
```

* `docker rm` deletes the container (but not the image).

### 4.2 Run a new container with port mapping

Run:

```bash
docker run -d --name my-web-nginx -p 8080:80 nginx
```

**Explanation:**

* `-p 8080:80` means:

  * Map **host port 8080** → **container port 80**.
  * Port 80 is the default HTTP port inside the Nginx container.
  * You will access it on your machine at `http://localhost:8080`.

### 4.3 Check that it’s running and mapped correctly

Run:

```bash
docker ps
```

Look at the `PORTS` column. You should see something like:

```text
0.0.0.0:8080->80/tcp
```

This confirms that port 8080 on your computer maps to port 80 in the container.

### 4.4 Test in your browser

Open your browser and go to:

```text
http://localhost:8080
```

You should see the **default Nginx welcome page** (usually a white page with “Welcome to nginx!”).

If you don’t see it:

* Check that the container is running (`docker ps`).
* Make sure nothing else is using port 8080.
* Try a different browser or incognito mode.

---

## Step 5: Mounting a Volume for Persistence

In this step, you will:

1. Create your **own HTML page** on your computer.
2. Mount that folder into the Nginx container so Nginx serves *your* page instead of the default one.

### 5.1 Stop and remove the current web container

Run:

```bash
docker stop my-web-nginx
docker rm my-web-nginx
```

### 5.2 Create a folder for your custom HTML

In your **current working directory**, create a folder:

```bash
mkdir nginx-html
```

Now you have a folder called `nginx-html` where you will store web files.

> You can check with:
>
> ```bash
> ls
> ```
>
> or on Windows:
>
> ```bash
> dir
> ```

### 5.3 Create `index.html` with custom content

You have two options:

#### Option A: Use the provided one-line command

**Windows (Command Prompt):**

```bat
echo ^<!DOCTYPE html^>^<html^>^<head^>^<title^>My Custom Nginx Page^</title^>^</head^>^<body^>^<h1^>Hello from Docker!^</h1^>^<p^>This is my custom Nginx content.^</p^>^</body^>^</html^> > nginx-html\index.html
```

**macOS / Linux (Bash):**

```bash
echo '<!DOCTYPE html><html><head><title>My Custom Nginx Page</title></head><body><h1>Hello from Docker!</h1><p>This is my custom Nginx content.</p></body></html>' > nginx-html/index.html
```

#### Option B: Create the file using a text editor (recommended for beginners)

1. Open a text editor:

   * Windows: Notepad / VS Code
   * macOS: TextEdit / VS Code
   * Linux: gedit / VS Code / nano

2. Paste this HTML:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My Custom Nginx Page</title>
   </head>
   <body>
       <h1>Hello from Docker!</h1>
       <p>This is my custom Nginx content.</p>
   </body>
   </html>
   ```

3. Save it as **`index.html`** **inside the `nginx-html` folder** you just created.

### 5.4 Find the absolute path to `nginx-html`

You will need the **full path** to the folder for the `-v` option.

* **macOS / Linux**:

  ```bash
  pwd
  ```

  If it shows `/Users/yourname/projects`, then your folder path is:

  ```text
  /Users/yourname/projects/nginx-html
  ```

* **Windows (PowerShell)**:

  ```powershell
  (Get-Location).Path
  ```

  If it shows `C:\Users\YourUser\Desktop`, your folder path is:

  ```text
  C:\Users\YourUser\Desktop\nginx-html
  ```

### 5.5 Run Nginx with a mounted volume

Use the **absolute path** in this command.

#### Windows example:

```bash
docker run -d --name my-persistent-nginx -p 8080:80 -v C:\Users\YourUser\Desktop\nginx-html:/usr/share/nginx/html nginx
```

#### macOS / Linux example:

```bash
docker run -d --name my-persistent-nginx -p 8080:80 -v /Users/yourname/projects/nginx-html:/usr/share/nginx/html nginx
```

**Explanation:**

* `-v host_path:container_path`

  * `host_path` is your folder on your machine.
  * `container_path` is where Nginx looks for web files (`/usr/share/nginx/html`).
* This means Nginx will now serve **your `index.html`** instead of its own default page.

### 5.6 Test in the browser

Again, go to:

```text
http://localhost:8080
```

You should now see your **custom page**:

* Title: “My Custom Nginx Page”
* Heading: “Hello from Docker!”
* Custom paragraph text.

If you edit the `index.html` file on your machine and refresh the page in your browser, you should see the changes **immediately**. That’s the benefit of using a volume.

---

## Step 6: Cleaning Up

After finishing the lab, you can clean up your containers and images to free disk space.

### 6.1 Stop the container

```bash
docker stop my-persistent-nginx
```

### 6.2 Remove the container

```bash
docker rm my-persistent-nginx
```

### 6.3 Remove the Nginx image

```bash
docker rmi nginx
```

> If Docker says the image is still in use, it means some container using `nginx` is still present. Make sure all `nginx` containers are stopped and removed.

### 6.4 Verify everything is cleaned up

Check containers:

```bash
docker ps -a
```

* You should **not** see `my-persistent-nginx` or any other lab container.

Check images:

```bash
docker images
```

* You should **not** see the `nginx` image (unless you are using it for something else).

---
## Step 7: Practice Exercise: Customise the Webpage with Student Name

Modify the content and insert the student’s name.

**Example:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Student Webpage</title>
</head>
<body>
    <h1>Hello from Docker!</h1>

    <h2>Student Name: <span style="color: blue;">YOUR NAME HERE</span></h2>

    <p>This webpage is customised as part of the Docker Practical Lab.</p>
</body>
</html>
```

You are required to **replace “YOUR NAME HERE” with their own name**.

---

# 📸 **Submission Requirement for Step 7**

### **1. Zipped the nginx-html and submit**

### **2. Screenshot of your customized webpage**

Take a screenshot of your browser showing your name and student ID


## Quick Summary of What You Learned

By completing this lab, you have:

* Pulled an image from Docker Hub (`docker pull nginx`).
* Created and managed containers (`docker run`, `stop`, `start`, `restart`, `rm`).
* Exposed a containerized web server to your host (`-p 8080:80` and `http://localhost:8080`).
* Mounted a host directory into a container as a volume (`-v host_path:/usr/share/nginx/html`).
* Cleaned up containers and images (`docker rm`, `docker rmi`).

You can now reuse these patterns for other images and web apps, not just Nginx.

