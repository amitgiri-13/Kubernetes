# Docker Commands, Arguments, CMD, and ENTRYPOINT 

# 1. Containers and Processes

## Important Concept

Containers are designed to run:

* A specific process
* A task
* An application

Examples:

* Web server
* Database
* API service
* Computation task

---

## Container Lifecycle

A container runs only while its main process is running.

```text id="m4u3t1"
Process exits → Container exits
```

---

# 2. Why Ubuntu Container Exits Immediately

## Example

```bash id="j5k2la"
docker run ubuntu
```

Result:

```text id="c4z7we"
Container exits immediately
```

---

## Why?

Ubuntu image uses:

```text id="m7v9qx"
bash
```

as the default command.

Bash:

* Requires an interactive terminal
* Exits if no terminal is attached

Since Docker does not attach a terminal automatically:

```text id="h8u1np"
bash exits → container exits
```

---

# 3. Check Stopped Containers

## Running Containers Only

```bash id="0qz2du"
docker ps
```

---

## All Containers Including Stopped

```bash id="cx9wlp"
docker ps -a
```

You will see:

```text id="v8j3om"
Exited containers
```

---

# 4. Default Process Inside Container

Defined using:

```dockerfile id="91k3aa"
CMD
```

instruction in Dockerfile.

---

# Examples

## NGINX Image

```dockerfile id="ntg3l7"
CMD ["nginx"]
```

---

## MySQL Image

```dockerfile id="s2yxvl"
CMD ["mysqld"]
```

---

# 5. Override Default Command

## Example

```bash id="9a8eqt"
docker run ubuntu sleep 5
```

Docker overrides:

```text id="p8tvxs"
Default CMD
```

and runs:

```text id="s4tqyo"
sleep 5
```

Container:

* Sleeps for 5 seconds
* Exits afterward

---

# 6. Create Custom Docker Image

## Dockerfile Example

```dockerfile id="8wjlwm"
FROM ubuntu

CMD ["sleep", "5"]
```

---

## Build Image

```bash id="0f4w3j"
docker build -t ubuntu-sleeper .
```

---

## Run Image

```bash id="2w5bkl"
docker run ubuntu-sleeper
```

Behavior:

```text id="lh93yp"
Container sleeps 5 seconds and exits
```

---

# 7. CMD Formats

## Shell Form

```dockerfile id="v6u4eo"
CMD sleep 5
```

---

## JSON Array Form (Preferred)

```dockerfile id="4mv6hj"
CMD ["sleep", "5"]
```

---

# Important Rule

In JSON form:

```text id="t7q1bd"
Command and arguments must be separate array elements
```

Correct:

```dockerfile id="u3m1po"
CMD ["sleep", "5"]
```

Wrong:

```dockerfile id="w4x8ka"
CMD ["sleep 5"]
```

---

# 8. Problem with Hardcoded CMD

Current image always runs:

```text id="9h3slm"
sleep 5
```

Cannot dynamically change duration easily.

---

# 9. ENTRYPOINT Instruction

## Purpose

ENTRYPOINT defines:

```text id="o5w2lr"
Executable command
```

CMD defines:

```text id="j7x9dp"
Default arguments
```

---

# ENTRYPOINT Example

```dockerfile id="q9z7fa"
FROM ubuntu

ENTRYPOINT ["sleep"]
```

---

## Run Container

```bash id="4h2lzp"
docker run ubuntu-sleeper 10
```

Final executed command:

```text id="o9s7yr"
sleep 10
```

---

# 10. Difference Between CMD and ENTRYPOINT

| Feature             | CMD                  | ENTRYPOINT         |
| ------------------- | -------------------- | ------------------ |
| Purpose             | Default command/args | Main executable    |
| Override behavior   | Fully replaced       | Arguments appended |
| Runtime flexibility | High                 | Fixed executable   |

---

# CMD Override Behavior

## Dockerfile

```dockerfile id="r5z0cv"
CMD ["sleep", "5"]
```

## Run

```bash id="2q4lkh"
docker run image sleep 10
```

Final command:

```text id="p4x7er"
sleep 10
```

CMD completely replaced.

---

# ENTRYPOINT Append Behavior

## Dockerfile

```dockerfile id="e6m2ys"
ENTRYPOINT ["sleep"]
```

## Run

```bash id="8y1ovm"
docker run image 10
```

Final command:

```text id="f3w7tk"
sleep 10
```

Arguments appended to ENTRYPOINT.

---

# 11. Combine ENTRYPOINT and CMD

## Best Practice

```dockerfile id="s7c5nt"
FROM ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

---

# Behavior

## Without Argument

```bash id="y0r8hv"
docker run ubuntu-sleeper
```

Runs:

```text id="3s8vke"
sleep 5
```

---

## With Argument

```bash id="a8t6wu"
docker run ubuntu-sleeper 10
```

Runs:

```text id="0u7nzd"
sleep 10
```

CMD gets overridden.

---

# 12. Missing Argument Problem

If only ENTRYPOINT exists:

```dockerfile id="t3v8ga"
ENTRYPOINT ["sleep"]
```

and container runs without argument:

```bash id="s0x1pk"
docker run ubuntu-sleeper
```

Error occurs:

```text id="m5l8wd"
sleep: missing operand
```

Because:

```text id="f0n7je"
No default argument supplied
```

---

# 13. Override ENTRYPOINT at Runtime

## Command

```bash id="n4x7ya"
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```

Final command:

```text id="a9r2cl"
sleep2.0 10
```

---

# 14. Execution Flow Summary

## CMD Only

```dockerfile id="5n0xwh"
CMD ["sleep", "5"]
```

Run:

```bash id="z4r6ls"
docker run image
```

Executes:

```text id="d2y8mv"
sleep 5
```

---

## ENTRYPOINT Only

```dockerfile id="r8q1dk"
ENTRYPOINT ["sleep"]
```

Run:

```bash id="v6k0up"
docker run image 10
```

Executes:

```text id="i1m4za"
sleep 10
```

---

## ENTRYPOINT + CMD

```dockerfile id="q7w9fn"
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Run:

```bash id="g3x2mc"
docker run image
```

Executes:

```text id="o8z6yj"
sleep 5
```

Run:

```bash id="l0v7kt"
docker run image 20
```

Executes:

```text id="p6j1sb"
sleep 20
```

---

# 15. Best Practices

## Use JSON Array Format

Preferred:

```dockerfile id="e2v5uh"
CMD ["nginx"]
```

Avoid:

```dockerfile id="s8o7ra"
CMD nginx
```

---

## Use ENTRYPOINT For

* Fixed executable
* Utility containers
* Standardized startup behavior

Examples:

* sleep
* python
* java
* nginx

---

## Use CMD For

* Default arguments
* Configurable runtime parameters

---

# Key Interview Questions

## Why do containers stop automatically?

Because:

```text id="t0z6rq"
Containers live as long as their main process runs
```

---

## Difference Between CMD and ENTRYPOINT?

### CMD

* Default command/arguments
* Replaced entirely at runtime

### ENTRYPOINT

* Main executable
* Runtime arguments appended

---

## Why JSON Array Syntax?

Advantages:

* Proper argument parsing
* Avoids shell interpretation issues
* Recommended Docker best practice
