# Multi-Stage Docker Build — Hands-On Practice

This hands-on demonstrates how to:

* Build a web application using a **multi-stage Dockerfile**
* Separate the **build stage** from the **runtime stage**
* Build and run the Docker image
* Push the image to Docker Hub
* Pull and run the image on another environment
* Inspect and troubleshoot the container
* Clean up Docker resources

---

## 1. Clone the Sample Application

Clone the sample Todo application:

```bash
git clone https://github.com/rahulkumar75/todoapp-docker.git
```

Move into the project:

```bash
cd todoapp-docker/
```

Check the application files:

```bash
ls
```

You should see files such as:

```text
package.json
src/
public/
...
```

---

## 2. Create the Dockerfile

Create an empty Dockerfile:

```bash
touch Dockerfile
```

Open it using your preferred editor:

```bash
vi Dockerfile
```

or:

```bash
nano Dockerfile
```

Add:

```dockerfile
# -------------------------
# Stage 1: Build
# -------------------------
FROM node:18-alpine AS installer

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build


# -------------------------
# Stage 2: Runtime
# -------------------------
FROM nginx:latest AS deployer

COPY --from=installer /app/build /usr/share/nginx/html
```

---

## 3. Understand the Dockerfile

### Stage 1 — `installer`

```dockerfile
FROM node:18-alpine AS installer
```

This stage contains Node.js and is responsible for **building the application**.

```dockerfile
WORKDIR /app
```

Sets `/app` as the working directory.

```dockerfile
COPY package*.json ./
```

Copies the dependency files first.

```dockerfile
RUN npm install
```

Installs Node.js dependencies.

```dockerfile
COPY . .
```

Copies the application source code.

```dockerfile
RUN npm run build
```

Creates the production build, usually inside:

```text
/app/build
```

---

### Stage 2 — `deployer`

```dockerfile
FROM nginx:latest AS deployer
```

This is the **runtime stage**.

The application doesn't need Node.js anymore. It only needs a web server to serve the generated static files.

```dockerfile
COPY --from=installer /app/build /usr/share/nginx/html
```

Copies only the generated build from the first stage.

```text
installer
   |
   | /app/build
   ↓
deployer
   |
   | /usr/share/nginx/html
   ↓
Nginx serves application
```

This is the main idea of a multi-stage build:

> **Build with Node.js → Run with Nginx.**

---

# 4. Build the Docker Image

Run from the project directory:

```bash
docker build -t todoapp-docker .
```

Explanation:

```text
docker build
    ↓
-t todoapp-docker
    ↓
.  → current directory / build context
```

You should see Docker execute both stages.

---

# 5. Verify the Image

Check locally created images:

```bash
docker images
```

Look for:

```text
todoapp-docker
```

You can also use:

```bash
docker image ls
```

---

# 6. Run the Container

Start the application:

```bash
docker run -dp 3000:80 todoapp-docker
```

Explanation:

```text
3000:80

Host Port 3000
      ↓
Container Port 80
      ↓
Nginx
```

Check running containers:

```bash
docker ps
```

Open:

```text
http://localhost:3000
```

Your Todo application should be accessible.

---

# 7. Verify the Container

Get the container ID:

```bash
docker ps
```

Example:

```text
CONTAINER ID   IMAGE            PORTS
a1b2c3d4       todoapp-docker   0.0.0.0:3000->80/tcp
```

You can use either the ID or container name in Docker commands.

---

# 8. Enter the Container

Using container name:

```bash
docker exec -it <container-name> sh
```

or using container ID:

```bash
docker exec -it <container-id> sh
```

Inside the container:

```bash
ls
```

Check the Nginx web directory:

```bash
ls /usr/share/nginx/html
```

You should see the generated application files.

This is a good way to verify that:

```dockerfile
COPY --from=installer /app/build /usr/share/nginx/html
```

worked correctly.

Exit:

```bash
exit
```

---

# 9. View Container Logs

```bash
docker logs <container-name>
```

or:

```bash
docker logs <container-id>
```

For continuously following logs:

```bash
docker logs -f <container-name>
```

Press:

```text
Ctrl + C
```

to stop following the logs.

---

# 10. Inspect the Container

Use:

```bash
docker inspect <container-name>
```

or:

```bash
docker inspect <container-id>
```

This provides detailed information such as:

* Container configuration
* Network settings
* Mounts
* Environment
* Port mappings
* Container state
* Image information

For image information:

```bash
docker image inspect todoapp-docker
```

---

# 11. Push the Image to Docker Hub

First, log in:

```bash
docker login
```

Tag the image:

```bash
docker tag todoapp-docker:latest username/new-reponame:tagname
```

For example:

```bash
docker tag todoapp-docker:latest rahulkumar/todoapp:v1
```

Verify:

```bash
docker images
```

You should now see both:

```text
todoapp-docker
username/new-reponame
```

Push:

```bash
docker push username/new-reponame:tagname
```

Example:

```bash
docker push rahulkumar/todoapp:v1
```

---

# 12. Pull the Image on Another Environment

On another machine:

```bash
docker login
```

Pull the image:

```bash
docker pull username/new-reponame:tagname
```

Example:

```bash
docker pull rahulkumar/todoapp:v1
```

Verify:

```bash
docker images
```

---

# 13. Run the Pulled Image

```bash
docker run -dp 3000:80 username/new-reponame:tagname
```

Example:

```bash
docker run -dp 3000:80 rahulkumar/todoapp:v1
```

Verify:

```bash
docker ps
```

Then open:

```text
http://localhost:3000
```

---

# 14. Troubleshooting

### Container is not running

Check:

```bash
docker ps -a
```

Then:

```bash
docker logs <container-id>
```

---

### Application is not accessible

Check:

```bash
docker ps
```

Verify:

```text
0.0.0.0:3000->80/tcp
```

Then test:

```bash
curl http://localhost:3000
```

---

### Build fails

Check the Docker build output.

Also verify:

```bash
cat package.json
```

Make sure the project has the required build script:

```json
"scripts": {
    "build": "..."
}
```

---

### `docker exec` fails

Make sure the container is running:

```bash
docker ps
```

If it has stopped:

```bash
docker ps -a
```

Check:

```bash
docker logs <container-id>
```

---

# 15. Cleanup

Stop the container:

```bash
docker stop <container-id>
```

Remove the container:

```bash
docker rm <container-id>
```

List images:

```bash
docker images
```

Remove an image:

```bash
docker image rm <image-id>
```

Or:

```bash
docker rmi <image-id>
```

---

# 16. Hands-On Verification Checklist

After completing the exercise, make sure you can verify each point:

* [ ] Clone the application repository.
* [ ] Create a Dockerfile.
* [ ] Understand the **builder stage**.
* [ ] Understand the **runtime stage**.
* [ ] Understand `COPY --from=installer`.
* [ ] Build the Docker image.
* [ ] Verify the image with `docker images`.
* [ ] Run the container with port mapping.
* [ ] Open the application on `localhost:3000`.
* [ ] Enter the container using `docker exec`.
* [ ] Verify files inside `/usr/share/nginx/html`.
* [ ] Check container logs.
* [ ] Inspect the container.
* [ ] Tag the image.
* [ ] Push the image to Docker Hub.
* [ ] Pull the image on another environment.
* [ ] Run the pulled image.
* [ ] Clean up containers and images.

---

## Key Takeaway

```text
Node.js Builder
      |
      | npm install
      | npm run build
      ↓
/app/build
      |
      | COPY --from=installer
      ↓
Nginx Runtime
      |
      ↓
Final Docker Image
      |
      ↓
Container :80
      |
      ↓
Host :3000
```

**Remember:**

> **Builder stage creates the application. Runtime stage only runs the application.**
