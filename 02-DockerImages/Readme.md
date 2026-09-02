# How To Dockerize a Project 

## Docker — Build, Push, Pull & Run an Application

## 1. Clone the Sample Repository

Clone the sample application repository:

```bash
git clone https://github.com/docker/getting-started-app.git
```

Move into the project directory:

```bash
cd getting-started-app/
```

> You can also use your own project instead of the sample repository.

---

## 2. Create a Dockerfile

Create an empty file named `Dockerfile`:

```bash
touch Dockerfile
```

Open the `Dockerfile` using your preferred text editor and add:

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY . .

RUN yarn install --production

CMD ["node", "src/index.js"]

EXPOSE 3000
```

### What each instruction does

| Instruction                     | Purpose                                                   |
| ------------------------------- | --------------------------------------------------------- |
| `FROM node:18-alpine`           | Uses Node.js 18 on Alpine Linux as the base image         |
| `WORKDIR /app`                  | Sets `/app` as the working directory inside the container |
| `COPY . .`                      | Copies the application files into the container           |
| `RUN yarn install --production` | Installs production dependencies                          |
| `CMD [...]`                     | Defines the command used to start the application         |
| `EXPOSE 3000`                   | Documents that the application listens on port `3000`     |

---

# 3. Build the Docker Image

Build the Docker image using the application code and `Dockerfile`:

```bash
docker build -t todo .
```

### Understanding the command

```text
docker build       → Build an image
-t todo      → Give the image the name "todo"
.                  → Use the current directory as the build context
```

---

# 4. Verify the Docker Image

Check whether the image was successfully created and stored locally:

```bash
docker images
```

You should see an image similar to:

```text
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
todo    latest    xxxxxxxxxxxx   ...           ...
```

---

# 5. Login to Docker Hub

Before pushing the image to Docker Hub, log in:

```bash
docker login
```

Enter your Docker Hub credentials when prompted.

---

# 6. Create a Repository on Docker Hub

Go to Docker Hub and create a **public repository**.

For example:

```text
Repository name: my-todo-app
```

Your Docker Hub username might be:

```text
username
```

So your complete image name can be:

```text
username/my-todo-app:latest
```

---

# 7. Tag the Docker Image

Tag your local image with the Docker Hub repository name:

```bash
docker tag todo:latest username/my-todo-app:latest
```

Replace:

```text
username
```

with your actual Docker Hub username.

Verify the image:

```bash
docker images
```

You should now see something similar to:

```text
REPOSITORY              TAG       IMAGE ID
todo              latest    xxxxxxxxx
username/my-todo-app    latest    xxxxxxxxx
```

Notice that both tags point to the same image ID.

---

# 8. Push the Image to Docker Hub

Push the tagged image to the remote Docker Hub repository:

```bash
docker push username/my-todo-app:latest
```

After the push completes, the image will be available in your Docker Hub repository.

---

# 9. Pull the Image from Another Environment

In another machine/environment, you can download the image from Docker Hub:

```bash
docker pull username/my-todo-app:latest
```

This demonstrates an important Docker workflow:

```text
Local Machine
     |
     | docker build
     ↓
Docker Image
     |
     | docker tag
     ↓
Docker Hub
     |
     | docker pull
     ↓
Another Environment
```

---

# 10. Run the Docker Container

Start a container from the image:

```bash
docker run -dp 3000:3000 username/my-todo-app:latest
```

### Understanding the command

```text
docker run
    ↓
Create and start a container

-d
    ↓
Run in detached/background mode

-p 3000:3000
    ↓
Host Port : Container Port

username/my-todo-app:latest
    ↓
Docker image to use
```

The port mapping means:

```text
Your Machine
localhost:3000
      |
      ↓
Container
port 3000
```

---

# 11. Verify the Application

Open:

```text
http://localhost:3000
```

If everything is configured correctly, the application should be accessible on port `3000`.

You can also verify the running container:

```bash
docker ps
```

---

# 12. Enter Inside the Running Container

First, identify the container:

```bash
docker ps
```

Then use either the **container name**:

```bash
docker exec -it containername sh
```

or the **container ID**:

```bash
docker exec -it containerid sh
```

For example:

```bash
docker exec -it abc123 sh
```

Once inside the container, you can run Linux commands such as:

```bash
ls
pwd
env
```

Exit the container shell with:

```bash
exit
```

---

# 13. View Container Logs

To view logs using the container name:

```bash
docker logs containername
```

Or using the container ID:

```bash
docker logs containerid
```

For example:

```bash
docker logs abc123
```

For continuously streaming logs:

```bash
docker logs -f containername
```

---

# Complete Docker Workflow

The complete flow you practiced is:

```text
1. Clone Application
       ↓
2. Create Dockerfile
       ↓
3. docker build
       ↓
4. Docker Image
       ↓
5. docker login
       ↓
6. Create Docker Hub Repository
       ↓
7. docker tag
       ↓
8. docker push
       ↓
9. Docker Hub
       ↓
10. docker pull
       ↓
11. docker run
       ↓
12. Application
       ↓
13. docker exec / docker logs
```

## Commands Cheat Sheet

```bash
# Clone
git clone https://github.com/docker/getting-started-app.git
cd getting-started-app/

# Create Dockerfile
touch Dockerfile

# Build image
docker build -t todo .

# List images
docker images

# Login
docker login

# Tag image
docker tag todo:latest username/my-todo-app:latest

# Push image
docker push username/my-todo-app:latest

# Pull image
docker pull username/my-todo-app:latest

# Run container
docker run -dp 3000:3000 username/my-todo-app:latest

# List running containers
docker ps

# Enter container
docker exec -it containername sh

# View logs
docker logs containername

# Follow logs
docker logs -f containername
```
