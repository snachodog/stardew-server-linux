# Stardew Valley Dedicated Server (Docker)

This repo provides a ready-to-run Docker setup for hosting a Stardew Valley multiplayer server on Linux using a prebuilt image.

The image is published at:

```code
ghcr.io/snachodog/stardew-server-linux:latest
```

You can run it either with the Docker Compose CLI or through Portainer.


## Prerequisites

- A machine with:
  - Docker installed
  - Docker Compose  installed (usually included with Docker)
- OR Portainer set up and attached to your Docker host
- A Steam account that **owns** Stardew Valley


## Environment variables

The container expects these variables:

- STEAM_USER – your Steam username  
- STEAM_PASS – your Steam password (or app password)

Recommended approach: create a `.env` file in the same directory as your compose file.

Example `.env`:

``` yaml
STEAM_USER=your_steam_username
STEAM_PASS=your_steam_password
```

Do not commit `.env` to a public repo.



## Docker Compose configuration

Example `docker-compose.yml`:

```
services:
  stardew:
    image: ghcr.io/snachodog/stardew-server-linux:latest
    container_name: stardew-server
    restart: unless-stopped

    ports:
      - "8080:6080"
      - "5901:5901"
      - "24642:24642/udp"

    volumes:
      - ./stardew_data:/root
      - ./Mods:/root/Steam/steamapps/common/Stardew Valley/Mods

    env_file:
      - .env

volumes:
  stardew_data:
```

## Running with Docker Compose

### 1. Clone this repo or create a folder

```
git clone <https://github.com/snachodog/stardew-server-linux.git>
cd stardew-server-linux
```

### 2. Create and edit `.env`

```
nano .env
```

Add:

```
STEAM_USER=your_steam_username
STEAM_PASS=your_steam_password
```

### 3. Start the server

```
docker compose up -d
```

### 4. View logs

```
docker compose logs -f
```

### 5. Stop the server

```
docker compose down
```

Your data stays in the local folders.



## Running with Portainer (Stacks)

These steps assume your Portainer instance is already connected to your Docker host.

### Option A – Create a stack and paste the compose file

1. In Portainer: Stacks → Add Stack  
2. Name it `sdv`
3. In the editor, paste your compose:

```
services:
  stardew:
    image: ghcr.io/snachodog/stardew-server-linux:latest
    container_name: stardew-server
    restart: unless-stopped

    ports:
      - "8080:6080"
      - "5901:5901"
      - "24642:24642/udp"

    volumes:
      - ./stardew_data:/root
      - ./Mods:/root/Steam/steamapps/common/Stardew Valley/Mods

    env_file:
      - .env

volumes:
  stardew_data:
```

### Handling volume paths in Portainer

Using `./` will place folders relative to Portainer's working directory on the Docker host.  
If you prefer explicit host paths:

```
volumes:

- /srv/stardew/stardew_data:/root
- /srv/stardew/Mods:/root/Steam/steamapps/common/Stardew Valley/Mods
```

Create these directories:

```
sudo mkdir -p /srv/stardew/stardew_data /srv/stardew/Mods
sudo chown -R $(id -u):$(id -g) /srv/stardew
```

### Environment variables in Portainer

Portainer does not auto-load `.env` files for stack deployments. Options:

#### Option 1 – Hardcode variables

```
environment:

- STEAM_USER=your_steam_username
- STEAM_PASS=your_steam_password
```

#### Option 2 – Use Portainer’s environment variable UI

Add `STEAM_USER` and `STEAM_PASS` under the stack’s environment variable section.

Change `.env` to `stack.env` in the docker-compose seciton

### Deploy

Scroll down → "Deploy the stack".



## Accessing the server

- **In-game:** Use `<server-ip>:24642`  
- **noVNC web UI:** `http://<server-ip>:8080`  
- **VNC client:** `<server-ip>:5901`



## Updating the server

### Docker Compose

```
docker compose pull
docker compose up -d
```

### Portainer

- Open the stack → "Update the stack"  
or  
- Containers → stardew-server → "Recreate" (with "pull latest image" checked)



## Removing the server

Keep data:

```
docker compose down
```

Delete everything including data:

```
docker compose down
rm -rf stardew_data Mods
```
