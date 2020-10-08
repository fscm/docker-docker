# Docker in Docker

A small Docker in Docker image that can be used to start a Docker daemon.

## Supported tags

- `19.03.9-centos`
- `19.03.9-debian`, `19.03.9`
- `19.03.12-centos`, `centos`
- `19.03.12-debian`, `19.03.12`, `debian`, `latest`

## What is Docker?

> Docker is a set of platform as a service (PaaS) products that uses OS-level virtualization to deliver software in packages called containers.

*from* [wikipedia.org](https://en.wikipedia.org/wiki/Docker_%28software%29)

## Getting Started

There are a couple of things needed for the script to work.

### Prerequisites

Docker, either the Community Edition (CE) or Enterprise Edition (EE), needs to
be installed on your local computer.

#### Docker

Docker installation instructions can be found
[here](https://docs.docker.com/install/).

### Usage

In order to end up with a functional Docker daemon - after having build
the container - some configurations have to be performed.

To help perform those configurations a small set of commands is included on the
Docker container.

- `help` - Usage help.
- `init` - Configure the Docker daemon.
- `start` - Start the Docker daemon.

To store the TLS configuration of the Docker daemon a volume should be created
and added to the container when running the same.

#### Creating Volumes

To be able to make all of the Docker daemon configuration settings persistent,
the same will have to be stored on a different volume.

Creating volumes can be done using the `docker` tool. To create a volume use
the following command:

```
docker volume create --name VOLUME_NAME
```

Two create the required volume the following command can be used:

```
docker volume create --name my_docker
```

**Note:** A local folder can also be used instead of a volume. Use the path of
the folder in place of the volume name.

#### Configuring the Docker daemon

To configure the Docker daemon the `init` command must be used.

```
docker container run --volume DOCKER_VOL:/data:rw --rm fscm/docker [options] init
```

* `-t` - Enable TLS by creating the required keys.

After this step the Docker daemon should be configured and ready to be
used.

An example on how to configure the Docker daemon:

```
docker container run --volume my_docker:/data:rw --rm fscm/docker -t init
```

**Note:** All the configuration files will be created and placed on the Docker
volume.

#### Start the Docker daemon

After configuring the Docker daemon the same can now be started.

Starting the Docker daemon can be done with the `start` command.

```
docker container run --volume DOCKER_VOL:/data:rw --detach --privileged --publish 2375:2375/tcp --publish 2376:2376/tcp fscm/docker [options] start
```

* `-t` - Enable TLS.

To help managing the container and the Docker daemon instance a name can be
given to the container. To do this use the `--name <NAME>` docker option when
starting the server

An example on how the Docker daemon can be started:

```
docker container run --volume my_docker:/data:rw --detach --privileged --publish 2375:2375/tcp --publish 2376:2376/tcp  --name my_docker fscm/docker start
```

To see the output of the container that was started use the following command:

```
docker container attach CONTAINER_ID
```

Use the `ctrl+p` `ctrl+q` command sequence to detach from the container.

#### Stop the Docker daemon

If needed the Docker daemon server can be stoped and later started again (as
long as the command used to perform the initial start was as indicated before).

To stop the server use the following command:

```
docker container stop CONTAINER_ID
```

To start the server again use the following command:

```
docker container start CONTAINER_ID
```

### Docker daemon Status

The Docker daemon status can be check by looking at the Docker daemon output
data using the docker command:

```
docker container logs CONTAINER_ID
```

## Build

Build instructions can be found
[here](https://github.com/fscm/docker-docker/blob/master/README.build.md).

## Versioning

This project uses [SemVer](http://semver.org/) for versioning. For the versions
available, see the [tags on this repository](https://github.com/fscm/docker-docker/tags).

## Authors

* **Frederico Martins** - [fscm](https://github.com/fscm)

See also the list of [contributors](https://github.com/fscm/docker-docker/contributors)
who participated in this project.
