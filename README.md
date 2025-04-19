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

---

Feel free to copy this file into your GitHub repository as `README.md` or documentation!

