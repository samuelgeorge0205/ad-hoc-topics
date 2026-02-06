

---

# 🧪 Step-by-Step: From Full Inspect → Digest File

---

## 🧩 STEP 0 — Raw `docker inspect` (Full Truth)

### Command

```bash
docker inspect myapp:stable
```

### Output (your sample, shortened for focus)

```json
[
  {
    "Id": "sha256:af3f0f48a24edb84e94aff6f44f5d089203453719d3b2328486d311e61db9b09",

    "RepoTags": [
      "busybox:latest",
      "myapp:a1b2c3d",
      "myapp:stable",
      "myapp:v1.2.3"
    ],

    "RepoDigests": [
      "busybox@sha256:b3255e7dfbcd10cb367af0d409747d511aeb66dfac98cf30e97e87e4207dd76f"
    ],

    ...
  }
]
```

### What to notice here (VERY IMPORTANT)

| Field         | Meaning                                            |
| ------------- | -------------------------------------------------- |
| `Id`          | Local image ID (internal, not registry identity)   |
| `RepoTags`    | All **human-friendly tags** pointing to this image |
| `RepoDigests` | **Registry-verified immutable identity**           |
| Array `[...]` | `inspect` always returns a list                    |

📌 **Key insight**

> Tags are many → digest is one → content is one

---

## 🧩 STEP 1 — Extract ONLY `RepoDigests`

### Command

```bash
docker inspect -f '{{.RepoDigests}}' myapp:stable
```

### Output

```text
[busybox@sha256:b3255e7dfbcd10cb367af0d409747d511aeb66dfac98cf30e97e87e4207dd76f]
```

### What changed

* Full JSON ❌
* Only `RepoDigests` ✅
* Still wrapped in:

  * square brackets `[]` (array)
  * repository prefix `busybox@`

📌 **Mental note**

> We are now working with **registry identity**, not local metadata.

---

## 🧩 STEP 2 — Extract FIRST digest element only

### Command

```bash
docker inspect -f '{{index .RepoDigests 0}}' myapp:stable
```

### Output

```text
busybox@sha256:b3255e7dfbcd10cb367af0d409747d511aeb66dfac98cf30e97e87e4207dd76f
```

### What changed

* Removed `[ ]`
* Selected **first element** of the list
* Still includes repository name before `@`

📌 **Why index 0?**

* `RepoDigests` is a list
* Most images have exactly one digest

---

## 🧩 STEP 3 — Pipe output into `awk`

### Command

```bash
docker inspect -f '{{index .RepoDigests 0}}' myapp:stable | awk -F'@' '{print $2}'
```

### Input to `awk`

```text
busybox@sha256:b3255e7dfbcd10cb367af0d409747d511aeb66dfac98cf30e97e87e4207dd76f
```

### How `awk` splits it

Separator:

```text
@
```

Resulting fields:

| Field | Value                                                                     |
| ----- | ------------------------------------------------------------------------- |
| `$1`  | `busybox`                                                                 |
| `$2`  | `sha256:b3255e7dfbcd10cb367af0d409747d511aeb66dfac98cf30e97e87e4207dd76f` |

### Output

```text
sha256:b3255e7dfbcd10cb367af0d409747d511aeb66dfac98cf30e97e87e4207dd76f
```

📌 **This is the exact value you want**

---

## 🧩 STEP 4 — Redirect Output to a File

### Command

```bash
docker inspect -f '{{index .RepoDigests 0}}' myapp:stable \
| awk -F'@' '{print $2}' \
> /home/user/image_digest.txt
```

### Terminal output

```text
(no output)
```

Why?

* STDOUT is redirected to a file

---

## 🧩 STEP 5 — Verify File Contents

### Command

```bash
cat /home/user/image_digest.txt
```

### Output

```text
sha256:b3255e7dfbcd10cb367af0d409747d511aeb66dfac98cf30e97e87e4207dd76f
```

✅ **Mission accomplished**

---

# 🔁 End-to-End Transformation Summary

```
FULL INSPECT JSON
        ↓
RepoDigests array
        ↓
Single digest entry
        ↓
Strip repository name
        ↓
sha256:....
        ↓
Saved to file
```

---

# 🧠 Critical Reality Checks (Don’t Skip)

### If STEP 1 outputs:

```text
[]
```

That means:

* image exists locally
* **but has never been pushed**
* registry digest doesn’t exist yet

Fix:

```bash
docker push myapp:stable
```

---

# 🔑 Final Locked Mental Model

* `RepoTags` → human labels (many)
* `RepoDigests` → immutable truth (one)
* `inspect` → Docker’s brain
* `index` → array access
* `awk` → text slicing
* `>` → persistence

You are now reading Docker metadata **like an engineer**, not guessing.
