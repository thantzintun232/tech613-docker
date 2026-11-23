## Notes

- An **image** is a blueprint. A **container** is a running copy of it.
- Docker pulls an image automatically if it isn't on my machine.
- A container's behaviour comes from its image. `hello-world` prints a message and exits by itself.

## 2. Setup

- Docker Desktop must be running (it shows **Engine running**).
- On Windows, use **Git Bash**.
- Help: `docker --help` or `docker <command> --help`.

## 3. Commands

### Images

| Command | What it does |
|---|---|
| `docker images` | List images on my machine |
| `docker pull nginx` | Download the latest official Nginx image |
| `docker pull nginx:<tag>` | Download a specific version |
| `docker rmi <image>` | Remove an image |

### Containers

| Command | What it does |
|---|---|
| `docker run hello-world` | Test that Docker works |
| `docker run -d -p 80:80 nginx` | Run Nginx in the background on port 80 |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers, including stopped ones |
| `docker stop <name or id>` | Stop a container |
| `docker start <name or id>` | Start a stopped container |
| `docker rm <name or id>` | Remove a stopped container |
| `docker rm -f <name or id>` | Force remove a running container |
| `docker exec -it <name> sh` | Open a shell inside a container |
| `exit` | Leave the container |

### Sharing images

| Command | What it does |
|---|---|
| `docker commit <container> <username>/<image>:<tag>` | Create an image from a running container |
| `docker login` | Log in to Docker Hub |
| `docker push <username>/<image>:<tag>` | Upload the image to Docker Hub |

## 4. Port mapping

Format: `-p <my machine>:<inside container>`

| Example | Open in browser |
|---|---|
| `-p 80:80` | `http://localhost` |
| `-p 90:80` | `http://localhost:90` |

- The first number can be changed. The second is the port Nginx uses inside the container (80).
- `-d` runs the container in the background.
- Two containers can't use the same host port.

## 5. Editing the Nginx page inside a container

1. `docker exec -it <name> sh`
2. `apt-get update -y` and `apt-get install nano -y` (containers are cut-down Linux)
3. `cd /usr/share/nginx/html`
4. `nano index.html`, edit the heading, save and exit
5. Refresh the browser to see the change

## 6. Publishing my own image

```bash
docker ps
docker commit <container> thantzintun234/nginx-610:v1
docker login
docker push thantzintun234/nginx-610:v1
```

Anyone can then run it with:

```bash
docker run -d -p 90:80 thantzintun234/nginx-610:v1
```

**Checklist:** the repo is public, and the image name starts with my Docker Hub username.

