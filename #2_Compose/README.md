# 2 - Composing your flavor

## Docker Compose 

Docker Compose is a command-line interface for Docker that can be used for a variety of goals. Similar to Docker, it uses a configuration file that helps automate a process. The file, or *composition*, is a set of information that describes how containers have to interact with the environment. However, one goal is often overlooked.
Take, for example, the following composition file.

:information_source: Docker Compose now follows the Compose Specification ([4]). Older 2.x and 3.x formats were merged, which is why modern examples usually omit a top-level `version`.

:information_source: Use `docker compose` (space) for current workflows. The standalone Compose binary is now a legacy install path ([7]).

```yaml
services:
  python:
    build: .
    volumes:
      - .:/home/myapp
```

:grey_question: What does this composition file do? (you can use the [getting started][1] guide)

With this in mind, think of Docker Compose as an extension of the Docker engine.
In the following, we will see how to actually compose a service.

## Before you start (2 minutes)

Run these checks once in this module folder:

```sh
docker compose version
docker compose config
```

If `docker compose` is not found, install/enable the Compose plugin as described in [7].

## Quick mental model

* Service: one container definition in `docker-compose.yml`.
* Project: the full app described by the Compose file.
* Network: shared by services in the same project by default.
* Volume: persistent/shared data managed by Compose or bind mounts.
* Lifecycle: `up` starts, `logs` inspects, `down` stops/removes.

## Minimal Compose command cheat sheet

* `docker compose up -d`: create/start services in background.
* `docker compose ps`: list service container status.
* `docker compose logs -f <service>`: follow service logs.
* `docker compose exec <service> <cmd>`: run a command in a running service.
* `docker compose config`: validate and render final config.
* `docker compose down`: stop and remove containers/network.
* `docker compose down -v`: also remove named volumes.

## General networking considerations

* By default, each Compose project gets a dedicated network named `<project>_default` ([9]).
* For Compose apps, this default network is typically a user-defined `bridge` network ([9], [10]).
* Containers on the same Compose network can resolve each other by service name (embedded DNS), for example `db` from `web` ([9]).
* `ports:` publishes container ports to the host; internal service-to-service traffic should use container/service ports, not host ports ([9]).
* `network_mode: host` shares the host network namespace. In this mode, `ports:` mappings are ignored ([11]).

:information_source: On Docker Desktop, networking is provided through a VM/backend layer, so host-level bridge details differ from native Linux Docker Engine behavior ([12]).

### Access host services from containers

If a container needs a service running on the host, use `host.docker.internal` when available ([12]).
On Linux engines, map it explicitly when needed:

```yaml
services:
  app:
    image: alpine
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

This avoids hardcoding the host IP and keeps the Compose file portable ([13]).

### Multiple network adapters per container

A service can attach to multiple networks (effectively multiple interfaces/routes):

```yaml
services:
  app:
    image: alpine
    command: sleep infinity
    networks:
      - frontend
      - backend

networks:
  frontend:
  backend:
```

This is useful when one container must talk to two isolated network segments ([4], [9]).

## Create a compose file

One of the most straightforward uses of a composition is a web service. It needs a stable operating system, a web server, a database, and an interpreter for an eventual server-side language like PHP or Python.
A LAMP (Linux Apache MySql PHP) is such a combination. It was trendy for a long time and is still of wide use.
The tutorial [HERE][2] shows how to create such a service using Docker and Docker Compose.

What you will encounter during this tutorial is that many options and flags remind a Dockerfile. Indeed, in many cases if specified in a compose file, these can replace or override the default values of an image.

## Resources

Resource management grows in importance as a system is used for multiple tasks. On a server, one service must not threaten the health of another.
As you may already have guessed, Docker Compose foresees parameters to manage these resources.

:grey_question: Which are they? Try to set up a memory limit for the database. 

:information_source: Startup order and readiness are different topics. `depends_on` controls startup order, while readiness is usually handled with `healthcheck` and conditions in Compose ([5]).

### Reading command output quickly

When you run `docker compose ps`, focus on `State`/`Status`, `Ports`, and service names.
When you run `docker compose logs`, focus on repeated errors and startup messages.
When you run `docker compose config`, verify image tags, volume mounts, and ports.

A thing not discussed previously is the flexibility of volumes. Beyond the traditional bind mounting of a folder, volumes can also be shared only among containers. They are created and labeled and then assigned to the instances requiring them.

:grey_question: In which cases such a shared volume might be useful?

## Swarm

An extension of a composition is a Swarm. Docker Swarm is one of several ways to use multiple nodes to provide a particular service. The [Voting App][3] example in the tutorial shows how an application can use parallelized instances. With multiple nodes, the service can be spawned on many servers or virtual instances. If you have not tried it out yet, do it now.

:information_source: For local development, Compose is usually the default choice. Swarm is more suitable when you need scheduling and scaling across multiple Docker hosts ([6]).

:information_source: Do not confuse Swarm mode with standalone "classic" Swarm, which has been removed from modern Docker Engine releases ([8]).

:grey_question: So, when is this really useful? Who might be applying this technology?

## Beginner troubleshooting

* `docker compose up` fails on port conflicts: change host ports in the Compose file.
* Service starts but app is unavailable: check `docker compose logs <service>` first.
* App container fails because DB is not ready yet: add readiness checks (`healthcheck`) and dependency conditions ([5]).
* Old data causes confusing behavior: reset with `docker compose down -v` and start again.

## 10-minute practice

1. Validate config in `#2_Compose`: `docker compose config`.
2. Build and start the Python service: `docker compose up -d --build`.
3. Check state: `docker compose ps`.
4. Inspect output: `docker compose logs python`.
5. Stop and cleanup: `docker compose down`.


[1]: <https://docs.docker.com/compose/gettingstarted/> "Docker compose, getting started"
[2]: <https://docs.docker.com/reference/samples/> "Docker samples"
[3]: <https://github.com/dockersamples/example-voting-app> "Deploying to a Swarm"
[4]: <https://docs.docker.com/reference/compose-file/> "Compose Specification reference"
[5]: <https://docs.docker.com/compose/how-tos/startup-order/> "Compose startup order and readiness"
[6]: <https://docs.docker.com/engine/swarm/> "Swarm mode overview"
[7]: <https://docs.docker.com/compose/install/> "Compose install methods"
[8]: <https://docs.docker.com/engine/deprecated/> "Engine deprecated and removed features"
[9]: <https://docs.docker.com/compose/how-tos/networking/> "Compose networking behavior"
[10]: <https://docs.docker.com/reference/compose-file/networks/> "Compose networks reference"
[11]: <https://docs.docker.com/engine/network/tutorials/host/> "Host network mode"
[12]: <https://docs.docker.com/desktop/features/networking/networking-how-tos/> "Docker Desktop networking how-tos"
[13]: <https://docs.docker.com/reference/cli/docker/container/run/> "host-gateway and add-host"
