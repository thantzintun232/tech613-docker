# Run the Sparta Node App in a Docker Container


## Goal

Run the Sparta test Node.js app in a Docker container, push the image to Docker Hub, and share a `docker run` command that works for anyone.

## Overview

| Step | What I do |
|---|---|
| 1 | Create a folder and put the `app` folder in it |
| 2 | Write a `Dockerfile` |
| 3 | Build the image |
| 4 | Run the container and check port 3000 |
| 5 | Push the image to Docker Hub |
| 6 | Verify it works for everyone |

## Step 1: Folder and app

```bash
mkdir docker-run-sparta-app
cd docker-run-sparta-app
```

Copy the `app` folder in. Only one Dockerfile is allowed per folder, which is why I use a separate folder.

Create a `.dockerignore` so my local `node_modules` is not copied in:

```bash
echo "app/node_modules" > .dockerignore
```

## Step 2: Create the Dockerfile

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY app/package.json app/package-lock.json ./
RUN npm ci --omit=dev --ignore-scripts
COPY app/ .
USER node
EXPOSE 3000
CMD ["node", "index.js"]
```

| Line | Meaning |
|---|---|
| `FROM node:20-alpine` | Official Node image on a small Linux (about 150 MB, not 1 GB) |
| `WORKDIR /app` | Working folder inside the container |
| `COPY ... ./` | Copy only the package files first, so Docker can cache the install step |
| `RUN npm ci --omit=dev --ignore-scripts` | Install exact versions from the lock file, without dev tools, and without running the `postinstall` script |
| `COPY app/ .` | Copy the app code (the Dockerfile sits outside `app`, so the path is `app/`) |
| `USER node` | Run as the `node` user instead of root, which is safer |
| `EXPOSE 3000` | Documents the app port (the real mapping is done with `-p`) |
| `CMD ["node", "index.js"]` | Start the app |

## Step 3: Build the image

```bash
docker build -t thantzintun234/tech613-tttapp:1.2.0 .
docker images
```

| Part | Meaning |
|---|---|
| `-t` | Name (tag) the image |
| `thantzintun234/tech613-tttapp` | Docker Hub username + image name |
| `:1.2.0` | Version tag |
| `.` | Use the current folder |

## Step 4: Run the container

```bash
docker run -d -p 3000:3000 thantzintun234/tech613-tttapp:1.2.0
docker ps
```

Open `http://localhost:3000` in the browser.

### Port mapping

Format: `-p <my machine>:<inside container>`

| Command | Open in browser |
|---|---|
| `-p 3000:3000` | `http://localhost:3000` |
| `-p 80:3000` | `http://localhost` |
| `-p 8080:3000` | `http://localhost:8080` |

The second number must match the port the app listens on (3000). Only the first number can change. `-p 3000:80` would fail because nothing listens on port 80 in this container.

## Step 5: Push to Docker Hub

```bash
docker push thantzintun234/tech613-tttapp:1.2.0
```

Each layer should show `Pushed`, then a `digest: sha256:...` line.

## Step 6: Verify

Open this in an incognito window:

```
https://hub.docker.com/r/thantzintun234/tech613-tttapp
```

If the page loads, it is public. To make it public: Docker Hub, **Repositories**, my repo, **Settings**, **Visibility**, **Make public**.

To test like a stranger:

```bash
docker rm -f <container-name>
docker rmi thantzintun234/tech613-tttapp:1.2.0
docker run -d -p 3000:3000 thantzintun234/tech613-tttapp:1.2.0
```

Docker should say it can't find the image locally and download it.

## Command to share

```bash
docker run -d -p 3000:3000 thantzintun234/tech613-tttapp:1.2.0
```

## Commands used

| Command | Purpose |
|---|---|
| `docker build -t <name>:<tag> .` | Build an image from a Dockerfile |
| `docker images` | List images |
| `docker run -d -p 3000:3000 <image>` | Run a container in the background |
| `docker ps` / `docker ps -a` | List running / all containers |
| `docker logs <name>` | See container output |
| `docker rm -f <name>` | Force remove a container |
| `docker push <image>` | Upload the image to Docker Hub |

## Problems I hit and fixes

| Problem | Cause | Fix |
|---|---|---|
| `npm ci` failed during build | `package.json` has `"postinstall": "node seeds/seed.js"`. It ran before the `seeds` folder was copied, and needs a database | Add `--ignore-scripts` to `npm ci` |
| Removing `--omit=dev` did not help | It is unrelated. It only skips dev dependencies | Keep both flags |
| Compose later said the image was `not found` | The image had not been pushed under that name | Tag and push it again |
| Image named `tech610` instead of `tech613` | The task example used another cohort's name | `docker tag` it with the new name and push |
| Push or login timeouts | Unstable network | Retry (finished layers are skipped), restart Docker Desktop, check DNS |

## Key takeaways

- Copy the package files first, then the code. Docker caches layers, so rebuilds are faster.
- `postinstall` scripts run during `npm ci`. Skip them in a build when they need files or services that aren't available yet.
- The container port in `-p` must match the port the app listens on.
- The image name must start with my Docker Hub username to push.
