# 🐧 Linux STDIN, STDOUT & STDERR — Visual Practical Guide

> A beginner → advanced guide for Linux, Shell, DevOps, Docker & Automation 🚀

---

# 🌟 Big Picture First

```
⌨️ Keyboard  →  STDIN (0)
                    ↓
                🧠 Program
                    ↓
🖥 STDOUT (1) → Screen
❌ STDERR (2) → Screen (errors)
```

Every Linux process automatically gets **3 standard streams**.

---

# 🧠 The 3 Standard Streams

| Stream | FD | Default | Purpose |
|--------|------|-----------|------------|
| ⌨️ STDIN | 0 | Keyboard | Input |
| 🖥 STDOUT | 1 | Terminal | Normal output |
| ❌ STDERR | 2 | Terminal | Errors |

👉 FD = File Descriptor (internal number used by Linux)

---

---

# ⌨️ STDIN (Standard Input)

## What is it?
Input going INTO a program.

Default = keyboard.

---

## Example

```bash
cat
```

Type:
```
hello
```

Output:
```
hello
```

Flow:
```
stdin → cat → stdout
```

---

## Input from file

```bash
cat < file.txt
```

Meaning:
```
file.txt → stdin → program
```

---

---

# 🖥 STDOUT (Standard Output)

## What is it?
Normal output from a program.

Default = screen.

---

## Example

```bash
ls
```

Shows files on terminal.

---

## Save output to file

### Overwrite
```bash
ls > out.txt
```

### Append
```bash
ls >> out.txt
```

---

---

# ❌ STDERR (Standard Error)

## What is it?
Only error messages.

---

## Example

```bash
ls wrongfile
```

Output:
```
No such file or directory
```

This goes to **stderr**, not stdout.

---

---

# 🔢 File Descriptors (Important)

Linux treats streams as numbers:

| Number | Meaning |
|-----------|-------------|
| 0 | stdin |
| 1 | stdout |
| 2 | stderr |

These numbers are used in redirection.

---

---

# 🔁 Redirection Operators Cheat Sheet

| Operator | Meaning |
|-------------|----------------|
| > | stdout overwrite |
| >> | stdout append |
| 2> | stderr overwrite |
| 2>> | stderr append |
| < | stdin from file |
| \| | pipe |
| &> | stdout + stderr |
| 2>&1 | merge stderr → stdout |

---

---

# 💾 Saving Output to Files

## Only STDOUT

```bash
command > out.txt
```

---

## Only STDERR

```bash
command 2> err.txt
```

---

## Both separately

```bash
command > out.txt 2> err.txt
```

---

## Both together (most common)

```bash
command > all.txt 2>&1
```

OR

```bash
command &> all.txt
```

---

---

# 🔥 Understanding 2>&1 (Very Important)

Break it:

```
2   >   &1
↑   ↑    ↑
stderr redirect stdout
```

Meaning:

👉 Send **stderr** to the same place as **stdout**

---

## Correct order

```bash
command > file.txt 2>&1
```

Result:
```
stdout → file
stderr → file
```

---

## Wrong order

```bash
command 2>&1 > file.txt
```

Result:
```
stdout → file
stderr → screen
```

⚠️ ORDER MATTERS!

---

💡 Memory trick:
> stderr follows stdout

---

---

# 🔗 Pipes ( | )

## What is pipe?
Send stdout of one command → stdin of another.

---

## Example

```bash
ls | grep txt
```

Flow:
```
ls → grep → filtered output
```

Used heavily in Linux + DevOps.

---

---

# 🧃 tee (show + save)

Sometimes you want:
- show on screen
- AND save to file

---

## Example

```bash
command | tee file.txt
```

Append:
```bash
command | tee -a file.txt
```

---

---

# 🗑 /dev/null (black hole)

Discard output.

---

## Ignore errors

```bash
command 2>/dev/null
```

---

## Ignore everything

```bash
command > /dev/null 2>&1
```

Common in:
- cron jobs
- scripts
- silent automation

---

---

# ⚙️ Background Jobs (&)

Run command in background.

```bash
sleep 60 &
```

Check:
```bash
jobs
```

Bring back:
```bash
fg
```

---

---

# 🔗 Logical AND (&&)

Run next command only if first succeeds.

```bash
mkdir test && cd test
```

Used in:
- scripts
- Dockerfiles
- CI/CD pipelines

---

---

# 🚀 Real DevOps Examples

## Save docker logs
```bash
docker logs nginx > logs.txt
```

## Save errors only
```bash
systemctl status docker 2> err.txt
```

## Everything
```bash
kubectl get pods &> pods.txt
```

## Cron job logging
```bash
0 2 * * * /backup.sh > backup.log 2>&1
```

## Filter processes
```bash
ps aux | grep nginx > nginx.txt
```

---

---

# 🧪 Practice Lab

Try:

```bash
ls file1 wrongfile > out.txt 2> err.txt
```

Then:

```bash
cat out.txt
cat err.txt
```

Observe the difference 👀

---

---

# 🎯 Interview Quick Answers

What is STDIN? → Input  
What is STDOUT? → Normal output  
What is STDERR? → Errors  
What is 2>&1? → Merge stderr into stdout  
What is | ? → Pipe  
What is /dev/null? → Discard output  

---

---

# 🏁 Final Rule

If you master:

```
>
>>
2>
2>&1
|
tee
/dev/null
```

👉 You can handle 95% of Linux logging & scripting tasks.

---

# 🎉 Done!

Happy learning 🐧  
Practice daily → becomes muscle memory 💪
