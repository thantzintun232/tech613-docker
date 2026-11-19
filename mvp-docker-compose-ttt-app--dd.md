# TicTacToe App + MongoDB with Docker Compose

## What I built

| Item | Detail |
|---|---|
| Services | `mongo` (database) and `app` (Sparta Node app), plus an optional `seed` service in v2 |
| App image | `thantzintun234/tech613-tttapp:1.2.0` (from Docker Hub) |
| Database image | `mongo:7.0` |
| Data storage | Volume `mongo-data`, so data survives restarts |
| Ports | App `3000:3000`, Mongo `27017:27017` |
| Link between them | Env var `MONGODB_URI=mongodb://mongo:27017/tictactoe` |

Inside Compose, the app reaches Mongo by its **service name** (`mongo`), not `localhost`.



## Version 1: manual seed

`v1-manual-seed/docker-compose.yaml`:

```yaml
services:
  mongo:
    image: mongo:7.0
    container_name: ttt-mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  app:
    image: thantzintun234/tech613-tttapp:1.2.0
    container_name: ttt-app
    ports:
      - "3000:3000"
    environment:
      - MONGODB_URI=mongodb://mongo:27017/tictactoe
      - STATEFUL_MODE=server
    depends_on:
      - mongo

volumes:
  mongo-data:
```

| Part | Meaning |
|---|---|
| `volumes: mongo-data:/data/db` | Keeps database files outside the container |
| `depends_on: mongo` | Starts Mongo before the app |

### Steps

```bash
cd v1-manual-seed
docker compose up -d
docker compose ps
```

Open `http://localhost:3000`. The database is empty.

Seed it by hand:

```bash
docker exec -it ttt-app node seeds/seed.js
```

| Part | Meaning |
|---|---|
| `docker exec -it ttt-app` | Run a command inside the running app container |
| `node seeds/seed.js` | The seed script (the working folder is `/app`) |


## Version 2: automatic seed (separate seed service)

`v2-auto-seed/docker-compose.yaml`:

```yaml
services:
  mongo:
    image: mongo:7.0
    container_name: ttt-mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  app:
    image: thantzintun234/tech613-tttapp:1.2.0
    container_name: ttt-app
    ports:
      - "3000:3000"
    environment:
      - MONGODB_URI=mongodb://mongo:27017/tictactoe
      - STATEFUL_MODE=server
    depends_on:
      - mongo

  seed:
    image: thantzintun234/tech613-tttapp:1.2.0
    container_name: ttt-seed
    command: sh -c "sleep 5 && node seeds/seed.js"
    environment:
      - MONGODB_URI=mongodb://mongo:27017/tictactoe
      - SEED_BASE_URL=http://app:3000
    depends_on:
      - app
      - mongo
    restart: "no"

volumes:
  mongo-data:
```

| Part | Meaning |
|---|---|
| Same image as the app | No extra build needed |
| `command: sh -c "sleep 5 && ..."` | Replaces the start command. Waits for the app, then runs the seed |
| `SEED_BASE_URL=http://app:3000` | Reach the app by service name, not `127.0.0.1` |
| `restart: "no"` | Runs once, then stops |

### Steps

```bash
cd v2-auto-seed
docker compose down -v
docker compose up -d
docker compose ps -a
docker compose logs seed
```

`ttt-seed` should show `Exited (0)`, which means the seed finished. Open `http://localhost:3000` to see the data.

## Version 3: automatic seed (inside the app container)

`v3-seed-in-app/docker-compose.yaml`:

```yaml
services:
  mongo:
    image: mongo:7.0
    container_name: ttt-mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  app:
    image: thantzintun234/tech613-tttapp:1.2.0
    container_name: ttt-app
    command: sh -c "node index.js & sleep 5 && node seeds/seed.js; wait"
    ports:
      - "3000:3000"
    environment:
      - MONGODB_URI=mongodb://mongo:27017/tictactoe
    depends_on:
      - mongo

volumes:
  mongo-data:
```

| Part | Meaning |
|---|---|
| `command:` | Replaces the image's start command, so I must start the app myself |
| `node index.js &` | Start the app in the background |
| `sleep 5` | Give the app time to start |
| `node seeds/seed.js` | Seed the database. The default `SEED_BASE_URL` (`127.0.0.1:3000`) works because the app is in the same container |
| `; wait` | Keep the container alive while the app runs, even if the seed fails |

### Steps

```bash
cd v3-seed-in-app
docker compose down -v
docker compose up -d
docker compose logs app
```

The logs show the app starting and the seed output.

## Comparing the versions

| | v1 manual | v2 separate seed service | v3 seed inside app |
|---|---|---|---|
| Services | `mongo`, `app` | `mongo`, `app`, `seed` | `mongo`, `app` |
| Seeding | I run `docker exec ... seed.js` | Runs by itself at startup | Runs by itself at startup |
| Extra step after `up` | Yes | None | None |
| Seed runs again when `app` restarts | No | No (runs once) | Yes, may duplicate data |
| Design | Simple, hands-on | Cleaner, one job per container | Shortcut, two processes in one container |

## Is `bindIp` needed?

No. I checked:

```bash
docker exec -it ttt-mongo mongosh --quiet --eval "db.adminCommand({getCmdLineOpts:1}).parsed"
```

It returned `{ net: { bindIp: '*' } }`, so Mongo already listens on all addresses and the app can reach it.

## Useful commands

| Command | Purpose |
|---|---|
| `docker compose up -d` | Start all services in the background |
| `docker compose ps -a` | List services, including stopped ones |
| `docker compose logs <service>` | See a service's output |
| `docker compose down` | Stop and remove containers (the volume stays) |
| `docker compose down -v` | Also delete the volume (empties the database) |
| `docker exec -it <container> <cmd>` | Run a command inside a container |

## Problems I hit and fixes

| Problem | Cause | Fix |
|---|---|---|
| `image ... not found` on `compose up` | The image was not on Docker Hub under that name | Tag it as `tech613-tttapp` and push it |
| Image build failed at `npm ci` | The `postinstall` seed script ran too early and needs a database | Add `--ignore-scripts` in the Dockerfile |
| `mongo:7.0` showed `Interrupted` | A side effect of the first error | Fix the app image error |
| `container name "/ttt-mongo" is already in use` | An older Compose project still had containers with that name | `docker rm -f ttt-mongo ttt-app ttt-seed`, or `docker compose down -v` in the old folder |
| `command:` on `app` stopped the web app | `command:` replaces the normal start command, so the app never started | Start the app in the command too (v3), or use a separate seed service (v2) |
| Seed needs the app running | The script calls the app through `SEED_BASE_URL` | Wait for the app and use the service name |
| Seeding twice may duplicate data | The script may insert every time | Use `docker compose down -v` for a clean start |
| Port or container name already in use | Another version is still running | Run `docker compose down` in that folder first |

## Key takeaways

- Services reach each other by **service name** on the Compose network.
- A **volume** keeps Mongo data. `down -v` deletes it.
- `depends_on` sets the start order, but does not wait until the app is ready.
- `command:` replaces the image's default start command.
- A one-shot service is a simple way to automate setup tasks.
- One `docker-compose.yaml` per folder, so each version gets its own folder.
