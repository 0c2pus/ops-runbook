# Docker Troubleshooting & Administration

Essential commands for managing containers and diagnosing microservice failures.

## 1. Container Status & Inspection
* `docker ps -a` — List all containers (running and stopped).
* `docker inspect <container_id>` — Get detailed JSON metadata (IPs, Mounts, Env vars).
* `docker stats` — Real-time CPU, Memory, and Network I/O usage per container.
* `docker port <container_id>` — Mapping between host and container ports.

## 2. Logs & Debugging
* `docker logs --tail 100 -f <container_id>` — Follow the last 100 lines of logs.
* `docker exec -it <container_id> /bin/bash` — Open an interactive shell inside a running container.
* `docker cp <container_id>:/path/to/log ./local_dir` — Copy files from container to host for analysis.
* `docker top <container_id>` — Display the running processes of a container.

## 3. Network & Volume Management
* `docker network ls` — List all Docker networks.
* `docker network inspect <network_name>` — See which containers are connected to a specific network.
* `docker volume ls` — List all persistent volumes (check for orphaned data).

## 4. Cleanup & Recovery
* `docker stop $(docker ps -q)` — Stop all running containers.
* `docker system prune -a` — **Use with caution:** Delete all unused containers, networks, and images to free up space.

## 5. Building & Running Containers
* `docker build -t <image-name> .` - Build an image from Dockerfile in the current directory.
* `docker build -t <image-name> /path/` - Build an image from Dockerfile in a specific directory.
* `docker run <image-name>` - Run a container from an image.
* `docker run -p <host-port>:<container-port> <image-name>` - Run container with port mapping to host.
* `docker run -d <image-name>` - Run container in detached (background) mode.
* `docker run -d -p 8888:8888 --name <name> <image-name>` - Run detached with port mapping and custom name.
* `docker stop <container_id>` - Gracefully stop a running container.
* `docker rm <container_id>` - Remove a stopped container.
* `docker rmi <image_id>` - Remove an image.