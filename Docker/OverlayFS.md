OverlayFS – The Filesystem Behind Docker Images & Containers

> *“If Docker images are Lego blocks, OverlayFS is the invisible table that stacks them perfectly.”*

OverlayFS (Overlay File System) is one of the most critical but least understood components of Docker.  
This chapter explains **what OverlayFS is**, **how Docker uses it**, **where data lives on the host**, and **how it directly affects Docker build performance and caching**.

---

## 1. What is OverlayFS?

OverlayFS is a **Linux Union File System**.

A *union filesystem* allows multiple directories (called *layers*) to be **merged and presented as a single directory**.

Docker uses OverlayFS to:
- Combine multiple image layers into one filesystem
- Avoid copying files unnecessarily
- Enable fast container startup
- Enable Docker build cache

OverlayFS is the **default and recommended storage driver** for Docker on modern Linux systems (`overlay2`).

---

## 2. Why Docker Needs OverlayFS

Without OverlayFS:
- Every container would need a full copy of the image
- Disk usage would explode
- Containers would start slowly
- Build caching would be inefficient

OverlayFS solves this by:
- Sharing **read-only layers**
- Allowing containers to write only where needed
- Reusing layers across images and containers

---

## 3. Core OverlayFS Concepts

OverlayFS works using four main directories:

| Component | Description |
|---------|-------------|
| `lowerdir` | Read-only layers (Docker image layers) |
| `upperdir` | Writable layer (container-specific changes) |
| `workdir` | Internal working directory for OverlayFS |
| `merged` | Final unified filesystem shown to the container |

📌 **Only `upperdir` is writable**  
All image layers remain read-only.

---

## 4. Where OverlayFS Lives on the Host

Docker stores OverlayFS data under:

```text
/var/lib/docker/overlay2/
````

Inside this directory, Docker creates one folder per layer.

Typical structure:

```text
/var/lib/docker/overlay2/
├── <layer-id>/
│   ├── diff/      # Actual files of this layer
│   ├── link
│   └── lower      # References to parent layers
├── <container-id>/
│   ├── diff/      # upperdir (container writable layer)
│   ├── work/      # workdir
│   └── merged/    # merged filesystem
```

### Mapping to OverlayFS terms

| OverlayFS Term | Docker Directory                               |
| -------------- | ---------------------------------------------- |
| `lowerdir`     | Multiple `diff/` directories from image layers |
| `upperdir`     | Container `diff/` directory                    |
| `workdir`      | Container `work/` directory                    |
| `merged`       | Container `merged/` directory                  |

---

## 5. Docker Image Layers and OverlayFS

Each Dockerfile instruction usually creates **one filesystem layer**.

Example Dockerfile:

```dockerfile
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y nginx
COPY app /app
```

OverlayFS representation:

```
lowerdir:
  ├── ubuntu base layer
  ├── apt-get update layer
  ├── nginx install layer
  └── app copy layer

upperdir:
  └── container-specific changes

merged:
  └── what the container sees
```

📌 Image layers are **immutable** once created.

---

## 6. Copy-on-Write (CoW) Explained

OverlayFS uses **Copy-on-Write** when a container modifies a file.

### Example

File exists in image layer:

```text
/etc/nginx/nginx.conf
```

Container modifies it.

### What OverlayFS does:

1. Copies the file from `lowerdir` → `upperdir`
2. Modifies the copy in `upperdir`
3. Original file remains untouched

📌 This ensures:

* Image layers stay unchanged
* Containers remain isolated

⚠️ **Cost of Copy-on-Write**

* First write is expensive
* Heavy write workloads reduce performance

---

## 7. OverlayFS and Linux Page Cache

OverlayFS relies heavily on the **Linux page cache**.

### Read behavior

* Read-only layers are cached efficiently
* Multiple containers can share cached layers
* Reads are fast

### Write behavior

* Writes go to `upperdir`
* Cache sharing breaks
* Copy-on-Write overhead applies

📌 **Reads are cheap, writes are costly**

---

## 8. OverlayFS and Docker Build Cache

Docker build cache operates **layer by layer**.

A layer is reused **only if**:

* Dockerfile instruction is unchanged
* Files used by that instruction are unchanged

If a layer changes:

* All layers above it are rebuilt
* OverlayFS creates new directories under `overlay2`

---

## 9. How Dockerfile Order Affects Build Performance

### ❌ Poorly Ordered Dockerfile

```dockerfile
FROM python:3.11
COPY . /app
RUN pip install -r requirements.txt
```

Problem:

* Any source code change invalidates `COPY`
* `pip install` runs again
* Cache is wasted
* New OverlayFS layers are created every build

---

### ✅ Optimized Dockerfile

```dockerfile
FROM python:3.11
COPY requirements.txt /app/
RUN pip install -r requirements.txt
COPY . /app
```

Why this is fast:

* Dependencies change less frequently
* `pip install` layer is reused
* Only final layer changes
* OverlayFS reuses lower layers

📌 **Always place frequently changing files at the bottom**

---

## 10. What Actually Slows Docker Builds

| Action                 | Performance Impact       |
| ---------------------- | ------------------------ |
| Cached layer reuse     | ⚡ Very fast              |
| Creating new layer     | ⚠️ Moderate              |
| Large file COPY        | 🐢 Slow                  |
| Many small files       | 🐢🐢 Slower              |
| Writing existing files | 🐌 Copy-on-Write penalty |

---

## 11. Why OverlayFS Is Ideal for Containers

Compared to older drivers:

| Feature             | OverlayFS |
| ------------------- | --------- |
| Disk efficiency     | Excellent |
| Layer sharing       | High      |
| Startup time        | Instant   |
| Build cache support | Strong    |
| Kernel support      | Native    |

This is why Docker standardized on **overlay2**.

---

## 12. Inspecting OverlayFS on a Live System

### Check Docker storage driver

```bash
docker info | grep -i storage
```

### View OverlayFS mounts

```bash
mount | grep overlay
```

Example output:

```text
overlay on /var/lib/docker/overlay2/.../merged
lowerdir=...,upperdir=...,workdir=...
```

---

## 13. Mental Model to Remember

> Docker images are **immutable layers**
> OverlayFS **stacks them**
> Containers **write only on top**
> Build cache = **reusing lower layers**

If you understand this, Dockerfile optimization becomes intuitive.

---

## 14. Key Takeaways

* OverlayFS is the backbone of Docker storage
* Image layers are read-only and shared
* Containers write using Copy-on-Write
* Docker build cache depends on layer order
* Dockerfile structure directly affects performance

---
