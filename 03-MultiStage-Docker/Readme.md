# Multi-Stage Docker Builds

## 1. What is a Multi-Stage Build?

A **multi-stage Docker build** uses multiple `FROM` statements in one Dockerfile.

The main idea is:

> **Use one stage to build the application and another smaller stage to run it.**

```text
Source Code
    ↓
Builder Stage
    ↓
Build Artifact
    ↓
Runtime Stage
    ↓
Final Small Image
```

The builder can contain compilers, development tools, and dependencies that are **not required at runtime**.

---

## 2. Why Use Multi-Stage Builds?

Without multi-stage builds:

```text
Final Image
├── Source code
├── Compiler
├── Build tools
├── Development dependencies
└── Application
```

With multi-stage builds:

```text
Builder Image
├── Compiler
├── Build tools
├── Source code
└── Application artifact
          ↓
          ↓ COPY --from
          ↓
Runtime Image
├── Runtime
└── Application artifact
```

### Benefits

* Smaller Docker images
* Faster image transfer/deployment
* Lower attack surface
* Build and runtime environments are separated
* Build tools don't need to exist in production

---

# 3. Basic Syntax

```dockerfile
FROM <build-image> AS builder

# Build application


FROM <runtime-image>

COPY --from=builder <source> <destination>

CMD [...]
```

`AS builder` gives the build stage a name.

`COPY --from=builder` copies files from that stage into the final stage.

---

# 4. Example: Python Application

```dockerfile
# Stage 1: Builder
FROM python:3.12 AS builder

WORKDIR /build

COPY requirements.txt .

RUN pip install \
    --no-cache-dir \
    --prefix=/install \
    -r requirements.txt


# Stage 2: Runtime
FROM python:3.12-slim

WORKDIR /app

COPY --from=builder /install /usr/local

COPY app ./app

CMD ["python", "app/main.py"]
```

### What happens?

```text
python:3.12
      ↓
Install dependencies
      ↓
Builder
      ↓
Copy dependencies
      ↓
python:3.12-slim
      ↓
Run application
```

The final image doesn't need the full Python build environment.

---

# 5. Non-Root User

By default, a container may run as `root`.

For production applications, prefer a **non-root user**.

### Why?

If an application is compromised, running as root can increase the potential impact.

Use the principle:

> **Least privilege — give the application only the permissions it needs.**

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN useradd \
    --create-home \
    --uid 10001 \
    appuser

COPY --chown=appuser:appuser app ./app

USER appuser

CMD ["python", "app/main.py"]
```

Important:

```dockerfile
USER appuser
```

means the application runs as `appuser`, not root.

---

# 6. `USER` vs `COPY --chown`

These solve different problems.

### `USER`

Defines **who runs the application**:

```dockerfile
USER appuser
```

### `COPY --chown`

Defines **who owns the copied files**:

```dockerfile
COPY --chown=appuser:appuser app ./app
```

Usually, both are useful together.

---

# 7. `.dockerignore`

Use `.dockerignore` to prevent unnecessary files from entering the Docker build context.

Example:

```text
.git
.env
.venv
__pycache__
*.pyc
node_modules
*.log
Dockerfile
README.md
```

Especially avoid sending:

```text
.env
credentials
private keys
large unnecessary files
```

Remember:

> `.dockerignore` controls the **build context**, while multi-stage builds control what goes into the **final image**.

---

# 8. Docker Layer Caching

Order Dockerfile instructions carefully.

### ❌ Less efficient

```dockerfile
COPY . .

RUN pip install -r requirements.txt
```

Any source-code change may invalidate the dependency layer.

### ✅ Better

```dockerfile
COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .
```

Now Docker can reuse the dependency layer when only application code changes.

### General rule

```text
Less frequently changing
        ↓
Base image
        ↓
Dependencies
        ↓
Application code
        ↓
More frequently changing
```

---

# 9. Production Best Practices

### Image

* Use a small runtime image such as `slim` when appropriate.
* Avoid `latest`; use a controlled version.
* Consider digest pinning for stronger reproducibility.

### Build

* Use multi-stage builds.
* Keep build tools out of the runtime image.
* Use dependency lock files/pinned versions.
* Take advantage of Docker layer caching.

### Security

* Run as a non-root user.
* Never bake secrets into the image.
* Use `.dockerignore`.
* Keep unnecessary packages out of the runtime image.
* Scan images for vulnerabilities.

### Runtime

* Use exec-form `CMD`:

```dockerfile
CMD ["python", "app.py"]
```

* Make the filesystem read-only where practical.
* Drop unnecessary Linux capabilities in production/Kubernetes.

---

# 10. Common Mistakes

### Running as root

```dockerfile
CMD ["python", "app.py"]
```

without configuring a non-root user.

### Copying everything

```dockerfile
COPY . .
```

without a proper `.dockerignore`.

### Installing build tools in runtime

```text
gcc
git
make
development headers
```

when the application doesn't need them.

### Poor layer ordering

Installing dependencies after copying the entire source tree.

### Putting secrets in Dockerfile

Avoid:

```dockerfile
ENV PASSWORD=secret
```

or hardcoding credentials.

---

# 11. Multi-Stage + Kubernetes

Multi-stage builds create a clean production image.

Kubernetes can then add runtime security:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
```

Think of it as:

```text
Dockerfile
   ↓
Small + secure image
   ↓
Kubernetes
   ↓
Runtime security controls
```

---

# 12. Interview Answer

**What is a multi-stage Docker build?**

> A multi-stage Docker build separates the build environment from the runtime environment using multiple `FROM` stages. The builder stage contains the tools and dependencies required to build the application, while the final stage contains only the runtime dependencies and application artifact. This helps reduce image size, attack surface, and deployment overhead. In production, I also prefer running the application as a non-root user and following Docker security and caching best practices.

---

# Quick Revision

```text
Multi-stage build
        ↓
Builder Stage
        ↓
Build artifact
        ↓
Runtime Stage
        ↓
Small final image
```

Remember these **6 points**:

1. **Build in one stage, run in another.**
2. Use `COPY --from=builder` to transfer artifacts.
3. Keep build tools out of the final image.
4. Run applications as a **non-root user**.
5. Use `.dockerignore` and proper layer ordering.
6. Keep the runtime image **small, secure, and reproducible**.

### Core principle

> **Build big, run small, and run with least privilege.**
