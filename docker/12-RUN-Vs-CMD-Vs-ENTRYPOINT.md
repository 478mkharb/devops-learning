# RUN vs CMD vs ENTRYPOINT (Docker)

This document explains the difference between **RUN**, **CMD**, and **ENTRYPOINT** in Dockerfiles, in an **interview-ready** and **practical** manner.

---

## 1. RUN

### Purpose

* Executes commands **at image build time**
* Used to install packages, update OS, compile code

### Key Points

* Runs only during `docker build`
* Creates a **new image layer**
* Output is baked into the image

### Example

```dockerfile
RUN apt-get update && apt-get install -y nginx
```

---

## 2. CMD

### Purpose

* Defines the **default command** to run when the container starts

### Key Points

* Executed at **container runtime**
* Can be **overridden** at runtime
* Only the **last CMD** instruction is effective
* Often used to provide default arguments

### Example

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

Override at runtime:

```bash
docker run myimage bash
```

---

## 3. ENTRYPOINT

### Purpose

* Defines the **main executable** of the container

### Key Points

* Executed at **container runtime**
* Not easily overridden
* Forces the container to behave like a specific command

### Example

```dockerfile
ENTRYPOINT ["nginx"]
```

---

## Using CMD and ENTRYPOINT Together (Very Important)

### Dockerfile

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

### Behavior

```bash
docker run myimage
```

Runs:

```text
python app.py
```

Override CMD:

```bash
docker run myimage script.py
```

Runs:

```text
python script.py
```

ENTRYPOINT remains unchanged.

---

## Comparison Table

| Feature             | RUN         | CMD             | ENTRYPOINT      |
| ------------------- | ----------- | --------------- | --------------- |
| Execution time      | Build time  | Runtime         | Runtime         |
| Purpose             | Build image | Default command | Main executable |
| Can be overridden   | No          | Yes             | No (mostly)     |
| Creates image layer | Yes         | No              | No              |

---

## Common Interview Traps

* RUN cannot start services at runtime
* Multiple CMD instructions → only last one works
* Prefer **exec form** over shell form

---

## Interview One-Liner

> RUN is used during image build, CMD provides a default runtime command, and ENTRYPOINT defines the fixed executable that always runs when the container starts.

---

## Best Practice

* Use **RUN** for installing dependencies
* Use **ENTRYPOINT** for the main application
* Use **CMD
