# Docker_Learning
information about Dockerfile, single-stage, multi-stage, distroless

# Dockerfile: Before & After Multistage Build

This project demonstrates how using **multi-stage builds** in Docker can significantly improve the efficiency, size, and security of container images.

---

## ⚒️ Before: Single-stage Build (Typical Flow)

A single-stage Dockerfile might look like this:

```dockerfile
FROM ubuntu

RUN apt-get update && \
    apt-get install -y python3 python3-pip

WORKDIR /app
COPY . .

CMD ["python3", "calculator.py"]
```

### ❌ Drawbacks:
- Final image contains **both Python runtime and build tools** (apt, pip, etc.).
- **Large image size** (~1GB+) because it includes Ubuntu base, package manager, caches, and everything needed during the build process.
- Not secure or clean: more tools = larger attack surface.

---

## ✅ After: Multistage Build (Optimized Flow)

```dockerfile
# ---------- Stage 1: Builder ----------
FROM ubuntu AS build

RUN apt-get update &&\
    apt-get install -y \
        python3 \
        python3-pip

WORKDIR /app
COPY . .

# ---------- Stage 2: Final Runtime ----------
FROM python:3.9-slim

WORKDIR /app
COPY --from=build /app /app

CMD ["python3", "calculator.py"]
```

---

## 🚀 Benefits of Multistage Build

| Feature                    | Single-Stage     | Multi-Stage     |
|---------------------------|------------------|-----------------|
| 🛉 Clean final image      | ❌ No             | ✅ Yes          |
| 📦 Final image size       | ❌ ~1GB+          | ✅ ~120MB       |
| 🛡️ Security              | ❌ Large surface   | ✅ Minimal base   |
| ⚙️ Separation of concerns | ❌ Mixed build/runtime | ✅ Isolated stages |
| 📤 Deployment speed        | ❌ Slower          | ✅ Faster         |

---

## 📂 File Structure in Final Image

Only what's needed is included in the final container:

```
/app/
└── calculator.py
```

No `apt`, no `pip`, no unnecessary tools.

---

## 🧠 Summary

Using **multi-stage builds** helps:

- Minimize image size
- Improve security
- Speed up build & deploy times
- Keep build & runtime logic separate

It's the best practice for containerizing Python applications—especially when your build tools aren’t needed at runtime.



Check out the official Distroless project: https://github.com/GoogleContainerTools/distroless


# Dockerizing Python with Distroless

This guide shows how to build and run a secure, minimal Docker image for a Python app using **Distroless** containers.

---

## 🧰 What is Distroless?

Distroless images are Docker images that **do not include a package manager, shell, or OS-level tools**. They include only:

- Your application
- Its runtime (e.g., Python interpreter)

This results in:

- ✅ Smaller image sizes
- ✅ Improved security
- ✅ Faster deployments

---

## ⚒️ Use Case: Python CLI App (`calculator.py`)

This guide assumes you have a simple Python script like:

```python
# calculator.py
A = float(input("Enter first number: "))
B = float(input("Enter second number: "))
operation = input("Choose operation (+, -, *, /): ")

if operation == '+':
    print("Result:", A + B)
elif operation == '-':
    print("Result:", A - B)
elif operation == '*':
    print("Result:", A * B)
elif operation == '/':
    print("Result:", A / B)
else:
    print("Invalid operation")
```

---

## ✅ Final Dockerfile (Multistage with Distroless)

```dockerfile
# -------- Stage 1: Build --------
FROM python:3.9-slim AS build

WORKDIR /app
COPY . .

# Optional: Install dependencies
# RUN pip install -r requirements.txt --target /app

# -------- Stage 2: Runtime --------
FROM gcr.io/distroless/python3

WORKDIR /app
COPY --from=build /app /app

CMD ["calculator.py"]
```

---

## 🚀 Build & Run

```bash
# Build the image
docker build -t calculator-distroless .

# Run the container
docker run -it calculator-distroless
```

> Use `-it` to keep the terminal interactive if using `input()` in your script.

---

## 📈 Benefits of Distroless

| Feature           | Value                          |
|------------------|----------------------------------|
| Size             | ✅ Much smaller than Ubuntu-based |
| Security         | ✅ No shell, no package manager  |
| Attack surface   | ✅ Minimal                     |
| Runtime only     | ✅ Includes only Python         |
| Best practice    | ✅ For production deployments   |

---

## 🔎 Debugging Tips

- Distroless images have **no shell**, so you can't `docker exec` into them.
- To debug, add a temporary step to your Dockerfile using an image with bash:

```dockerfile
FROM python:3.9-slim  # for debugging
CMD ["bash"]
```

---

## 🧠 Summary

Using **Distroless + Multistage builds** helps you:

- Create ultra-small, secure Python containers
- Avoid bloated images with unnecessary tools
- Align with Docker best practices for production

---

## 📦 Want more?
Check out the official Distroless project: https://github.com/GoogleContainerTools/distroless

