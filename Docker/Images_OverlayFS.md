**Docker images** covering:

* 🧱 what an image *really* is
* 📁 files & directories on disk (Linux focus)
* 🗂️ layers, metadata, cache
* 🔄 image states & lifecycle
* 🧰 **all practical image commands** and what they prove
* 🧠 mental models you can reuse in debugging & interviews


---

## 🧠 One-Line Truth (Lock This First)

> **A Docker image is an immutable, layered filesystem + metadata, stored and managed by the Docker engine.**

Everything else is detail around this.

---

## 1️⃣ What a Docker Image REALLY Contains

A Docker image is **NOT**:

* ❌ a running thing
* ❌ a VM
* ❌ an OS

It **IS**:

### 📦 Image = 3 things

1. **Read-only filesystem layers**
2. **Metadata (JSON config)**
3. **Content-addressed identity (digest)**

---

## 2️⃣ Image Layers (Filesystem Part)

Each Dockerfile instruction (`FROM`, `RUN`, `COPY`, …) creates a **layer**.

Each layer:

* is **read-only**
* represents a **filesystem diff**
* is **content-addressed** (hash)
* can be **shared across images**

Example mental stack:

```
Layer 4  ← COPY app/
Layer 3  ← RUN apt install nginx
Layer 2  ← Base OS updates
Layer 1  ← ubuntu:22.04
```

📌

> Docker never modifies layers. It only stacks them.

---

## 3️⃣ Where Docker Images Live on Disk (Linux)

> ⚠️ **Important**: Paths below are for **Docker Engine on Linux**
> (Docker Desktop hides this inside a VM)

### Root Docker directory

```text
/var/lib/docker/
```

This is Docker’s **entire universe**.

---

### Inside `/var/lib/docker/`

```
/var/lib/docker/
├── overlay2/        ← image + container filesystems
├── image/           ← image metadata
├── containers/      ← container-specific metadata
├── volumes/         ← named volumes
├── network/         ← network state
└── tmp/
```

We care mainly about **images**, so let’s zoom in.

---

## 4️⃣ `overlay2/` — Where Image Layers Actually Live

![Image](https://docs.docker.com/engine/storage/drivers/images/overlay_constructs.webp)

![Image](https://jvns.ca/images/overlay.jpeg)

```text
/var/lib/docker/overlay2/
├── <layer-id>/
│   ├── diff/        ← actual files in this layer
│   ├── link
│   └── lower
```

### 🔹 `diff/`

* Contains **real files** added/changed in that layer
* Example:

  ```text
  diff/usr/bin/nginx
  diff/etc/nginx/nginx.conf
  ```

### 🔹 `lower`

* References **parent layers**
* This is how stacking works

📌

> OverlayFS merges multiple `diff/` directories into one view.

---

## 5️⃣ `image/overlay2/` — Image Metadata

```text
/var/lib/docker/image/overlay2/
├── imagedb/
│   └── content/sha256/
│       └── <image-config-hash>
├── layerdb/
│   └── sha256/
│       └── <layer-hash>/
│           ├── diff
│           ├── size
│           ├── parent
```

This is where Docker stores:

* image configuration JSON
* layer relationships
* sizes
* parent-child mapping

📌

> Filesystem = `overlay2/`
> Metadata = `image/overlay2/`

---

## 6️⃣ Image Metadata (What Docker Knows About an Image)

Stored as JSON (viewable via `docker inspect`):

```json
{
  "Id": "sha256:abcd...",
  "RepoTags": ["nginx:latest"],
  "Config": {
    "Cmd": ["nginx", "-g", "daemon off;"],
    "Env": ["PATH=/usr/local/bin"],
    "WorkingDir": "/"
  },
  "RootFS": {
    "Type": "layers",
    "Layers": [
      "sha256:layer1",
      "sha256:layer2"
    ]
  }
}
```

This metadata defines:

* default command
* env vars
* exposed ports
* entrypoint
* layer list

📌

> Image behavior comes from metadata, not magic.

---

## 7️⃣ Image Identity: Tags vs Digests

### 🏷️ Tag

```text
nginx:latest
nginx:1.25
```

* Human-friendly
* **Mutable**
* Can point to different images over time

---

### 🔐 Digest

```text
nginx@sha256:9b3f...
```

* Cryptographic hash
* **Immutable**
* Content-addressed

📌

> **Tags move. Digests don’t.**

---

## 8️⃣ Image States (Important Concept)

Images don’t “run”, but they do have **lifecycle states**:

### Image lifecycle

```
Dockerfile
   ↓ build
Local Image
   ↓ tag
Tagged Image
   ↓ push
Remote Registry
   ↓ pull
Local Image
   ↓ remove
Dangling / Deleted
```

---

### 🟡 Dangling Images

```text
<none>:<none>
```

Caused by:

* rebuilding with same tag
* old layers no longer referenced

List them:

```bash
docker images -f dangling=true
```

Remove:

```bash
docker image prune
```

📌

> Dangling ≠ unused layers (subtle but important).

---

## 9️⃣ Relationship: Image ↔ Container

```
Image (read-only layers)
        ↓
Container adds
  writable layer
```

When container is deleted:

* writable layer is destroyed
* image layers remain untouched

📌

> Images outlive containers by design.

---

## 🔟 ALL Practical Docker Image Commands (With WHY)

### 📋 List images

```bash
docker images
```

Shows:

* repository
* tag
* image ID
* size

---

### 🔍 Inspect image

```bash
docker inspect nginx
```

Truth source:

* CMD
* ENV
* layers
* architecture

---

### 🧱 Image history (layers)

```bash
docker history nginx
```

Shows:

* each layer
* command that created it
* size contribution

---

### ⬇️ Pull image

```bash
docker pull nginx:1.25
```

Downloads:

* missing layers only
* verifies digest

---

### ⬆️ Push image

```bash
docker push myrepo/myimage:1.0
```

Uploads:

* only layers not already in registry

---

### 🏷️ Tag image

```bash
docker tag nginx:latest nginx:prod
```

No copy happens.
Only metadata changes.

---

### ❌ Remove image

```bash
docker rmi nginx
```

Fails if:

* container exists using it

Force:

```bash
docker rmi -f nginx
```

---

### 🧹 Prune unused images

```bash
docker image prune
docker image prune -a
```

📌

> `-a` removes **all unused images**, not just dangling.

---

## 1️⃣1️⃣ How Docker Decides Image Reuse

Docker uses:

* layer hashes
* content-addressable storage

If two images share layers:

* stored once
* referenced many times

That’s why:

* pulling is fast
* disk usage is efficient

---

## 1️⃣2️⃣ Common Image Misconceptions (Kill These)

❌ “Images are heavy like VMs”
❌ “Each image duplicates everything”
❌ “Rebuilding wastes space”
❌ “Deleting containers deletes images”

All false.

---

## 🔑 Final Mental Models (Interview-Grade)

* **Image** = immutable filesystem + metadata
* **Layer** = filesystem diff
* **Tag** = mutable pointer
* **Digest** = immutable identity
* **Container** = image + writable layer
* **overlay2** = how Linux merges layers

