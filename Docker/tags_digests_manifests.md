
# 🏷️ Tags, 🔐 Digests, 📜 Manifests — Everything Explained

> 🧠 Docker images are identified by **content**, but humans interact using **names**.
> Tags, digests, and manifests exist to bridge that gap.

---

## 🔑 One-Line Truths (Lock These First)

* **Tag** = human-friendly, mutable pointer
* **Digest** = cryptographic, immutable identity
* **Manifest** = metadata document that binds them together

Everything else builds on this.

---

## 1️⃣ What Is an Image *Really* Identified By?

Internally, Docker **never trusts names**.

An image is uniquely identified by:

```
sha256:<hash>
```

Example:

```
sha256:9b3f0c2d4a8e1f...
```

📌

> This hash is calculated from the **image content**, not the name.

This hash = **image digest**.

---

## 2️⃣ Tags — Human-Friendly Names

### What a tag is

A tag is just a **label**:

```
nginx:latest
nginx:1.25
myapp:prod
```

Internally, a tag maps to a digest:

```
nginx:latest ───▶ sha256:9b3f0c2d...
```

---

### Key properties of tags

| Property       | Value |
| -------------- | ----- |
| Human-readable | ✅     |
| Mutable        | ✅     |
| Unique         | ❌     |
| Secure         | ❌     |
| Content-based  | ❌     |

📌

> Tags are **convenience**, not truth.

---

### Why tags are dangerous in production

```text
nginx:latest  (today) → digest A
nginx:latest  (tomorrow) → digest B
```

Same name. Different image.

📌

> Tags can **move without warning**.

---

## 3️⃣ Digests — Immutable Identity

### What a digest is

A digest is a **SHA256 hash** of the image manifest:

```
nginx@sha256:9b3f0c2d4a8e1f...
```

This represents:

* exact image content
* exact layers
* exact configuration

---

### Key properties of digests

| Property       | Value |
| -------------- | ----- |
| Human-friendly | ❌     |
| Mutable        | ❌     |
| Unique         | ✅     |
| Secure         | ✅     |
| Content-based  | ✅     |

📌

> If the content changes, the digest **must change**.

---

### Why digests are trusted

* Used by Docker internally
* Verified on pull
* Guaranteed reproducibility
* Used in security scanning
* Used in Kubernetes deployments

📌

> Digests are the **ground truth**.

---

## 4️⃣ Tags vs Digests — Side-by-Side

| Aspect           | Tag          | Digest                |
| ---------------- | ------------ | --------------------- |
| Example          | `nginx:1.25` | `nginx@sha256:abc...` |
| Mutable          | ✅            | ❌                     |
| Human-friendly   | ✅            | ❌                     |
| Secure           | ❌            | ✅                     |
| Stable reference | ❌            | ✅                     |
| Used internally  | ❌            | ✅                     |

---

## 5️⃣ Manifests — The Missing Link

### What a manifest is

A **manifest** is a JSON document that describes:

* which layers make up the image
* the image configuration
* the target architecture / OS

Example (simplified):

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
  "config": {
    "digest": "sha256:config-hash"
  },
  "layers": [
    { "digest": "sha256:layer1" },
    { "digest": "sha256:layer2" }
  ]
}
```

📌

> The digest is calculated from **this manifest**.

---

## 6️⃣ Relationship: Tag → Manifest → Layers

```
Tag (nginx:1.25)
   ↓
Manifest (JSON)
   ↓
Config + Layer Digests
   ↓
Actual Layer Blobs
```

📌

> Tags never point to layers directly.
> They point to **manifests**.

---

## 7️⃣ Multi-Architecture Images (Manifest Lists)

This is where manifests become **critical**.

### Example

```bash
docker pull nginx
```

On:

* amd64 → pulls amd64 image
* arm64 → pulls arm64 image

Same tag. Different image.

How?

---

### Manifest List (Index)

A **manifest list** is a higher-level manifest:

```json
{
  "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
  "manifests": [
    { "platform": { "architecture": "amd64" }, "digest": "sha256:aaa" },
    { "platform": { "architecture": "arm64" }, "digest": "sha256:bbb" }
  ]
}
```

📌

> Docker selects the correct image **at pull time**.

---

## 8️⃣ Commands to Explore Tags, Digests, Manifests

### List images with digests

```bash
docker images --digests
```

Output example:

```
REPOSITORY  TAG     DIGEST
nginx       latest  sha256:9b3f...
```

---

### Inspect image (shows digest & manifest info)

```bash
docker inspect nginx
```

Look for:

* `RepoDigests`
* `RootFS.Layers`

---

### Pull by digest (best practice)

```bash
docker pull nginx@sha256:9b3f0c2d...
```

📌

> This guarantees exact content.

---

### Retagging does NOT change digest

```bash
docker tag nginx:latest nginx:prod
```

Both point to **same digest**.

---

## 9️⃣ How Registries Store This

Registries store:

* blobs (layers)
* manifests (JSON)
* tag → manifest mapping

They **do not care about names**.

📌

> Registries are **content-addressed databases**.

---

## 🔟 Promotion & Deployment (Real World)

### ❌ Bad practice

```bash
docker build -t myapp:prod .
```

Builds different images for different envs.

---

### ✅ Correct practice

```bash
docker build -t myapp:1.2.3 .
docker tag myapp:1.2.3 myapp:prod
docker push myapp:1.2.3
docker push myapp:prod
```

Same digest everywhere.

📌

> Promotion = **retagging**, not rebuilding.

---

## 1️⃣1️⃣ Kubernetes & Digests (Why This Matters)

Kubernetes **prefers digests**:

```yaml
image: nginx@sha256:9b3f0c2d...
```

Why?

* deterministic deployments
* no surprise upgrades
* strong supply-chain security

📌

> Tags are for humans. Digests are for systems.

---

## 1️⃣2️⃣ Common Misconceptions (Kill These)

❌ Tag = version
❌ Digest = compressed image
❌ Manifests are optional
❌ Retagging copies images
❌ `latest` means newest safely

All false.

---

## 🔑 Final Mental Models (Interview-Grade)

* **Image identity** = digest
* **Tag** = movable pointer
* **Manifest** = contract describing image
* **Manifest list** = multi-arch selector
* **Security & reproducibility** come from digests

If these are solid, **Docker image behavior will never confuse you again**.

---
