
# 📦 Docker Registries & Image Distribution — End to End

> 🧠 If images are the **artifact**,
> registries are the **supply chain**.

Until you understand registries, Docker feels local and incomplete.

---

## 🔑 One-Line Truth (Lock This First)

> **A Docker registry is a content-addressed storage and distribution system for images.**

It does **not** run containers.
It does **not** build images.
It only **stores and serves** them.

---

## 🧠 Big Picture — Image Distribution Flow

![Image](https://miro.medium.com/1%2AuuZ-h5EH76LOtJ614z-qDA.png)

![Image](https://www.tutorialspoint.com/docker/images/docker_hub_1.jpg)

![Image](https://www.researchgate.net/publication/308050257/figure/fig1/AS%3A433709594746881%401480415833510/High-level-overview-of-Docker-architecture.png)

```
Dockerfile
   ↓ docker build
Local Image
   ↓ docker tag
Named Image
   ↓ docker push
Registry
   ↓ docker pull
Remote Host
   ↓ docker run
Container
```

This is the **entire Docker supply chain**.

---

## 1️⃣ What a Docker Registry Really Is

A registry provides:

* 🗂️ **Blob storage** for image layers
* 🧾 **Metadata APIs** (manifests, tags)
* 🔐 **Authentication & authorization**
* 🔄 **Layer deduplication**

It stores:

* layers (by SHA256 digest)
* image manifests
* tag → digest mappings

📌

> Registries store **content**, not “images” as files.

---

## 2️⃣ Registry vs Repository vs Image (Common Confusion)

Let’s clean this up clearly.

### 🏢 Registry

* Server/service
* Hosts repositories
* Example: Docker Hub

---

### 📁 Repository

* Logical collection of image versions
* Example:

  ```
  nginx
  myorg/payment-service
  ```

---

### 🏷️ Image (Tag)

* Specific version in a repository
* Example:

  ```
  nginx:1.25
  nginx:latest
  ```

📌

> Registry → Repository → Image (tag)

---

## 3️⃣ Public Registries

Public registries allow **anonymous pull** (sometimes rate-limited).

### Common Public Registries

* Docker Hub
* GitHub Container Registry
* Quay

### Characteristics

✅ Easy to use
✅ Free tiers
✅ Huge ecosystem
❌ Rate limits
❌ Public visibility by default
❌ Trust must be verified

📌

> Public ≠ trusted by default.

---

## 4️⃣ Private Registries

Private registries require **authentication for pull & push**.

### Common Private Registries

* Amazon Elastic Container Registry
* Azure Container Registry
* Google Artifact Registry
* Self-hosted Docker Registry (`registry:2`)

### Characteristics

✅ Access control
✅ Private images
✅ No public rate limits
✅ Enterprise security
❌ Cost
❌ More setup

📌

> Private registries are about **control and trust**.

---

## 5️⃣ Image Naming & Registry Resolution (Very Important)

### Full image reference format

```text
REGISTRY/REPOSITORY:TAG
```

Examples:

```text
nginx:latest                  → Docker Hub (implicit)
docker.io/library/nginx:latest
ghcr.io/myorg/api:1.0
123456789012.dkr.ecr.us-east-1.amazonaws.com/app:prod
```

### Default behavior

If **registry is omitted**:

```bash
docker pull nginx
```

Docker assumes:

```
docker.io/library/nginx:latest
```

📌

> Docker Hub is the **default registry**.

---

## 6️⃣ Image Distribution — What REALLY Happens on `docker push`

```bash
docker push myorg/app:1.0
```

### Step-by-step internally

1. Docker checks local image layers
2. Calculates SHA256 for each layer
3. Contacts registry
4. Uploads **only missing layers**
5. Uploads image manifest
6. Updates tag → digest mapping

📌

> Pushing is **incremental**, not full upload.

---

## 7️⃣ Image Distribution — What REALLY Happens on `docker pull`

```bash
docker pull myorg/app:1.0
```

Docker:

1. Fetches manifest
2. Checks local layer cache
3. Downloads only missing layers
4. Verifies digests
5. Assembles image locally

📌

> Pulling is also **incremental and verified**.

---

## 8️⃣ Tags vs Digests in Registries (Critical)

### 🏷️ Tags

* Human-friendly
* Mutable
* Can move

```text
myapp:latest
```

---

### 🔐 Digests

* Content hash
* Immutable
* Verifiable

```text
myapp@sha256:abc123...
```

📌

> **Registries store by digest. Tags are pointers.**

---

## 9️⃣ Authentication & Authorization

### Login

```bash
docker login
docker login ghcr.io
docker login 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

Creates:

```text
~/.docker/config.json
```

Contains:

* registry URL
* auth token (base64 / credential helper)

---

### Permissions

Registries enforce:

* who can **push**
* who can **pull**
* who can **delete**

📌

> Registry security is **separate** from Docker daemon security.

---

## 🔁 The Full Distribution Lifecycle (CI/CD View)

```
Developer
  ↓ docker build
Local Image
  ↓ docker tag
Versioned Image
  ↓ docker push
Registry
  ↓ docker pull
CI / QA / Prod
  ↓ docker run
Container
```

### Key principle

> **Build once, run everywhere.**

* No builds in production
* Same image promoted across environments
* Only configuration changes

---

## 1️⃣0️⃣ Promotion Strategy (Real-World)

Example:

```text
myapp:1.0.0-dev
myapp:1.0.0-qa
myapp:1.0.0-prod
```

All point to **same digest**.

📌

> Promotion = retagging, not rebuilding.

---

## 1️⃣1️⃣ Registry Garbage Collection (Advanced Insight)

When tags move:

* old digests may become unreferenced
* layers may still exist

Registries periodically:

* clean unreferenced blobs
* reclaim storage

📌

> Deleting tags ≠ deleting layers immediately.

---

## 1️⃣2️⃣ Common Registry Mistakes (Very Important)

❌ Using `latest` in production
❌ Rebuilding instead of retagging
❌ Trusting public images blindly
❌ Pushing secrets into images
❌ Letting CI push unversioned tags

All lead to **non-reproducible systems**.

---

## 🔑 Final Mental Models (Interview-Grade)

* **Registry** = content store + API
* **Repository** = image namespace
* **Tag** = mutable pointer
* **Digest** = immutable truth
* **Push/Pull** = layer sync, not copy
* **Distribution** = promotion, not rebuild

If these are clear, **Docker at scale makes sense**.

---
