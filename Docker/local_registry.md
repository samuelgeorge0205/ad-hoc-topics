
# 📦 Docker Local Registry — Everything You Need to Know

> 🧠 A local registry is **Docker, without the internet**.

It turns image distribution into something **you fully control**.

---

## 🔑 One-Line Truth (Lock This First)

> **A local Docker registry is just another container that stores image layers and metadata over HTTP.**

Nothing special.
No magic.
No new Docker concepts.

---

## 1️⃣ What Is a Local Docker Registry?

A **local registry** is:

* A Docker Registry server
* Running **as a container**
* Usually based on the image: `registry:2`
* Listening on a local port (e.g. `localhost:5000`)
* Storing images on your machine

📌

> It is the *same registry software* used by large registries — just self-hosted.

---

## 2️⃣ Why a Local Registry Exists (Real Reasons)

You use a local registry when:

* 🚫 No internet access
* 🧪 Learning & experimentation
* 🏗️ CI/CD pipelines
* 🏭 Air-gapped environments
* 🔁 Image caching
* 🧠 Understanding registry internals

📌

> A local registry removes **network and trust complexity** so you can focus on Docker mechanics.

---

## 3️⃣ What a Local Registry Is NOT

A local registry does **not**:

* ❌ Build images
* ❌ Run containers
* ❌ Replace Docker daemon
* ❌ Replace Kubernetes

It only:

* stores
* serves
* verifies image content

---

## 4️⃣ Running a Local Registry (Hands-On)

### 🔹 Start a local registry

```bash
docker run -d \
  -p 5000:5000 \
  --name local-registry \
  registry:2
```

### What each part means

| Part           | WHY                     |
| -------------- | ----------------------- |
| `registry:2`   | Official registry image |
| `-p 5000:5000` | Expose registry API     |
| `-d`           | Run in background       |
| `--name`       | Easier management       |

📌

> Your registry is now just another container.

---

## 5️⃣ Verify the Registry Is Running

```bash
docker ps
```

You should see:

```
local-registry
```

Test API:

```bash
curl http://localhost:5000/v2/_catalog
```

Expected output:

```json
{"repositories":[]}
```

📌

> Empty registry, but alive.

---

## 6️⃣ How Docker Decides Where to Push Images

Docker image reference format:

```text
REGISTRY/REPOSITORY:TAG
```

### Local registry example

```text
localhost:5000/myapp:1.0
```

📌

> The **registry is determined by the image name**, not by config.

---

## 7️⃣ Tagging an Image for Local Registry

### Step 1 — Build or pull an image

```bash
docker pull nginx
```

### Step 2 — Tag it for local registry

```bash
docker tag nginx localhost:5000/nginx:local
```

📌

> Tagging does NOT copy the image.
> It only creates a new reference.

---

## 8️⃣ Pushing to Local Registry

```bash
docker push localhost:5000/nginx:local
```

### What actually happens

1. Docker checks local layers
2. Registry checks which layers it already has
3. Missing layers are uploaded
4. Manifest is stored
5. Tag → digest mapping created

📌

> Push is **layer-based**, not image-based.

---

## 9️⃣ Listing Images in Local Registry

```bash
curl http://localhost:5000/v2/_catalog
```

Output:

```json
{"repositories":["nginx"]}
```

List tags:

```bash
curl http://localhost:5000/v2/nginx/tags/list
```

📌

> Registry API is plain HTTP + JSON.

---

## 🔟 Pulling from Local Registry

### Remove local image first

```bash
docker rmi nginx
docker rmi localhost:5000/nginx:local
```

### Pull from local registry

```bash
docker pull localhost:5000/nginx:local
```

📌

> Docker treats local registry exactly like Docker Hub.

---

## 1️⃣1️⃣ Image Flow with Local Registry (Mental Model)

```
Dockerfile
   ↓ docker build
Local Image
   ↓ docker tag
Local Registry Reference
   ↓ docker push
Local Registry
   ↓ docker pull
Same / Other Host
```

📌

> The registry is the **handoff point** between systems.

---

## 1️⃣2️⃣ Where Local Registry Stores Images (On Disk)

Inside the registry container:

```text
/var/lib/registry/
├── docker/
│   └── registry/
│       └── v2/
│           ├── blobs/
│           └── repositories/
```

### What’s stored

* **blobs/** → layer content (by digest)
* **repositories/** → tag & manifest metadata

📌

> Registry storage is **content-addressed**, just like Docker images.

---

## 1️⃣3️⃣ Persistence (Very Important)

### ❌ Without volume (bad)

If registry container dies → images lost.

---

### ✅ With volume (correct)

```bash
docker run -d \
  -p 5000:5000 \
  --name local-registry \
  -v registry-data:/var/lib/registry \
  registry:2
```

📌

> Registry data must live **outside** the container.

---

## 1️⃣4️⃣ Insecure Registry (Why Docker Complains)

Local registry runs on **HTTP**, not HTTPS.

Docker will fail with:

```
http: server gave HTTP response to HTTPS client
```

### Fix (Linux)

Edit Docker daemon config:

```json
{
  "insecure-registries": ["localhost:5000"]
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

📌

> Docker defaults to HTTPS for safety.

---

## 1️⃣5️⃣ Local Registry States

A local registry can be:

| State      | Meaning              |
| ---------- | -------------------- |
| Running    | Accepts push/pull    |
| Stopped    | Registry unavailable |
| Empty      | No repositories      |
| Populated  | Has images           |
| Persistent | Volume-backed        |
| Ephemeral  | Data lost on stop    |

---

## 1️⃣6️⃣ Common Local Registry Use Cases

### 🧪 Learning Docker internals

* Understand push/pull
* See layers
* Debug naming

### 🏗️ CI pipelines

* Cache base images
* Reduce external pulls

### 🏭 Air-gapped systems

* Transfer images once
* Reuse locally

### 🔁 Multi-host testing

* Push once
* Pull on many machines

---

## 1️⃣7️⃣ Common Mistakes (Important)

❌ Forgetting volume → data loss
❌ Not configuring insecure registry
❌ Using `latest` everywhere
❌ Treating registry as backup
❌ Assuming registry builds images

---

## 🔑 Final Mental Models (Lock These)

* **Registry = content store**
* **Local registry = same registry, closer**
* **Tag decides destination**
* **Push/Pull = layer sync**
* **Registry container is disposable; data is not**

If these are clear, **image distribution stops being scary**.

---
