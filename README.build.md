# Docker in Docker

Docker image with Docker.

## Synopsis

This script will create a Docker image with Docker installed and with all
of the required initialization scripts.

The Docker image resulting from this script should be the one used to
instantiate a Docker daemon.

## Getting Started

There are a couple of things needed for the script to work.

### Prerequisites

Docker, either the Community Edition (CE) or Enterprise Edition (EE), needs to
be installed on your local computer.

#### Docker

Docker installation instructions can be found
[here](https://docs.docker.com/install/).

### Usage

In order to create a Docker image using this Dockerfiles you need to run the
`docker` command with a few options.

```
docker image build --force-rm --no-cache --quiet --file Dockerfile.<VARIANT> --tag <USER>/<IMAGE>:<TAG> <PATH>
```

* `<USER>` - *[required]* The user that will own the container image (e.g.: "johndoe").
* `<IMAGE>` - *[required]* The container name (e.g.: "docker").
* `<TAG>` - *[required]* The container tag (e.g.: "latest").
* `<PATH>` - *[required]* The location of the Dockerfile folder.
* `<VARIANT>` - *[required]* The variant that is being build (`centos` or `debian`).

A build example:

```
docker image build --force-rm --no-cache --quiet --file Dockerfile.debian --tag johndoe/my_docker:debian .
```

To clean any _<none>_ image(s) left by the build process the following
command can be used:

```
docker image rm `docker image ls --filter "dangling=true" --quiet`
```

You can also use the following command to achieve the same result:

```
docker image prune -f
```

### Instantiate a Container

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

To be able to make all of the TLS configuration settings persistent, the same
will have to be stored on a different volume.

Creating volumes can be done using the `docker` tool. To create a volume use
the following command:

```
docker volume create --name <VOLUME_NAME>
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
docker container run --volume <DOCKER_VOL>:/data:rw --rm <USER>/<IMAGE>:<TAG> [options] init
```

* `-t` - Enable TLS by creating the required keys.

After this step the Docker daemon should be configured and ready to be used.

An example on how to configure the Docker daemon:

```
docker container run --volume my_docker:/data:rw --rm johndoe/my_docker -t init
```

**Note:** All the configuration files will be created and placed on the Docker
volume.

#### Start the Docker daemon

After configuring the Docker daemon the same can now be started.

Starting the Docker daemon can be done with the `start` command.

```
docker container run --volume <DOCKER_VOL>:/data:rw --detach --publish 2375:2375/tcp  --publish 2376:2376/tcp <USER>/<IMAGE>:<TAG> start
```

To help managing the container and the Docker daemon instance a name can be
given to the container. To do this use the `--name <NAME>` docker option when
starting the server

An example on how the Docker daemon can be started:

```
docker container --volume my_docker:/data:rw --detach --publish 2375:2375/tcp  --publish 2376:2376/tcp --name my_docker johndoe/my_docker start
```

To see the output of the container that was started use the following command:

```
docker container attach <CONTAINER_ID>
```

Use the `ctrl+p` `ctrl+q` command sequence to detach from the container.

#### Stop the Docker daemon

If needed the Docker daemon can be stoped and later started again (as long as
the command used to perform the initial start was as indicated before).

To stop the server use the following command:

```
docker container stop <CONTAINER_ID>
```

To start the server again use the following command:

```
docker container start <CONTAINER_ID>
```

### Docker daemon Status

The Docker daemon status can be check by looking at the Docker daemon output
data using the docker command:

```
docker container logs <CONTAINER_ID>
```

### Add Tags to the Docker Image

Additional tags can be added to the image using the following command:

```
docker image tag <image_id> <user>/<image>:<extra_tag>
```

### Push the image to Docker Hub

After adding an image to Docker, that image can be pushed to a Docker registry... Like Docker Hub.

Make sure that you are logged in to the service.

```
docker login
```

When logged in, an image can be pushed using the following command:

```
docker image push <user>/<image>:<tag>
```

Extra tags can also be pushed.

```
docker image push <user>/<image>:<extra_tag>
```

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request

Please read the [CONTRIBUTING.md](CONTRIBUTING.md) file for more details on how
to contribute to this project.

## Versioning

This project uses [SemVer](http://semver.org/) for versioning. For the versions
available, see the [tags on this repository](https://github.com/fscm/docker-docker/tags).

## Authors

* **Frederico Martins** - [fscm](https://github.com/fscm)

See also the list of [contributors](https://github.com/fscm/docker-docker/contributors)
who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE)
file for details
