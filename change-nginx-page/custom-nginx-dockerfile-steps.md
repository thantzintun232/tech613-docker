# Automate a Custom Nginx Page with a Dockerfile


## How it works

| Piece | What it is |
|---|---|
| `index.html` | My custom web page (on my laptop) |
| `Dockerfile` | Instructions for building the image |
| Image | The built package: Nginx + my page |
| Container | The running copy of the image |

## Step 1: Create the project folder

```bash
mkdir tech613-mod-nginx-dockerfile
cd tech613-mod-nginx-dockerfile
```


## Step 2: Create `index.html`

```bash
nano index.html => 

<!DOCTYPE html>
<html>
<head>
  <title>Tech613 DevOps</title>
</head>
<body>
  <h1 style="color: green;">Welcome to Tech613 - DevOps (Thant)</h1>
  <p>This page was built automatically with a Dockerfile.</p>
</body>
</html>

```
Verify:

```bash
cat index.html
```

## Step 3: Create the `Dockerfile`

```bash
nano Dockerfile =>

FROM nginx
COPY index.html /usr/share/nginx/html/index.html

```


| Line | Meaning |
|---|---|
| `FROM nginx` | Start from the official Nginx image |
| `COPY index.html /usr/share/nginx/html/index.html` | Copy my page into the image, replacing the default Nginx page |


`COPY` takes a snapshot at build time. If I edit the file later, the running container does not change. I must **rebuild** the image.

## Step 4: Build the image

```bash
docker build -t thantzintun234/tech613-nginx-auto:1.0 .
```

| Part | Meaning |
|---|---|
| `docker build` | Build an image from a Dockerfile |
| `-t` | Name (tag) the image |
| `thantzintun234/tech613-nginx-auto` | Docker Hub username + image name |
| `:1.0` | Version tag |
| `.` | Use the current folder (don't forget this) |

Check it:

```bash
docker images
```

## Step 5: Run the container

```bash
docker run -d -p 80:80 thantzintun234/tech613-nginx-auto:1.0
```

| Part | Meaning |
|---|---|
| `-d` | Run in the background |
| `-p 80:80` | Machine port 80 connects to container port 80 |

Check it:

```bash
docker ps
```

Open `http://localhost` in the browser. I should see the green heading **"Welcome to Tech613 - DevOps (Thant)"**.

## Step 6: Push the image to Docker Hub

Make sure I'm signed in to Docker Desktop, then:

```bash
docker push thantzintun234/tech613-nginx-auto:1.0
```

Each layer should show `Pushed`, then a `digest: sha256:...` line.

## Step 7: Check it is public

Open this in an **incognito window**:

```
https://hub.docker.com/r/thantzintun234/tech613-nginx-auto
```

| Result | Meaning |
|---|---|
| Page loads | Pushed and public |
| 404 | Not pushed, or the repo is private |

To make it public: hub.docker.com, **Repositories**, my repo, **Settings**, **Visibility**, **Make public**.

## Step 8: Command to share

```bash
docker run -d -p 80:80 thantzintun234/tech613-nginx-auto:1.0
```

Anyone can run this, and Docker pulls the image automatically.

## Commands used

| Command | Purpose |
|---|---|
| `mkdir` / `cd` / `pwd` | Create, enter and check the folder |
| `cat > file <<'EOF'` | Create a file from the terminal |
| `docker build -t <name>:<tag> .` | Build an image from the Dockerfile |
| `docker images` | List images |
| `docker run -d -p 80:80 <image>` | Run a container in the background |
| `docker ps` | List running containers |
| `docker rm -f <name>` | Force remove a container |
| `docker push <image>` | Upload the image to Docker Hub |

## Key takeaways

- A Dockerfile makes the setup **repeatable and automatic**, with no manual edits in a container.
- `COPY` puts my file into the image at build time.
- Always rebuild after changing files.
- The image name must start with my Docker Hub username to push.
- Tag images with a version (e.g. `:1.0`).
