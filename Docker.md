# Docker essentials
- docker build -t [tag] . : Build an image from a Dockerfile
- docker create [image]: Create a new container from a particular image.
- docker login: Log into an Docker registry.
- docker pull [image]: Pull an image from an Docker registry.
- docker push [username/image]: Push an image to an Docker registry.
- docker search [term]: Search an Docker registry for a particular term.
- docker tag [source] [target]: Create a target tag or alias that refers to a source image.

# Running Docker Containers
- docker start [container]: Start a particular container.
- docker stop [container]: Stop a particular container.
- docker exec -ti [container] [command]: Run a shell command inside a particular container.
- docker run -ti — image [image] [container] [command]: Create and start a container at the same time, and then run a command inside it.
- docker run -ti — rm — image [image] [container] [command]: Create and start a container at the same time, run a command inside it, and then remove the container - after executing the command.
- docker pause [container]: Pause all processes running within a particular container.

# Using Docker Utilities
- docker stats [container]: Display a live stream of container(s) resource usage statistics
- docker logs [container]: Fetch the logs of a container, add minus -f to follow log output
- docker history [image]: Show  the history of a particular image.
- docker image ls: List all of the images that are currently stored on the system.
- docker inspect [object]: Show  low-level information about a particular Docker object.
- docker ps: List all of the containers that are currently running.
- docker version: Show  the version of Docker that is currently installed on the system.
- docker events --since 5m: Show  all events within the last 5 minutes

# Cleaning Up Your Docker Environment
- docker image prune --all: Remove all unused images, not just dangling ones
- docker kill [container]: Kill a particular container.
- docker kill $(docker ps -q): Kill all containers that are currently running.
- docker rm [container]: Delete a particular container that is not currently running.
- docker rm $(docker ps -a -q): Delete all containers that are not currently running.