# Docker for .NET

## Docker overview

Motivation behind using docker is having a reproducible environment for application development and deployment.

Instead of installing libraries, tools, configuration, etc. - everywhere the application runs - a [docker image](#docker-image) which contains everything application needs to run (tools, add configuration files/resources, libraries, etc.). This image is then consistent between development machine, cloud host, VPS, etc. [Docker container](#docker-container) is self-contained and isolated - other applications or different versions of some tool running on host won't affect container execution. Likewise once something is added to an image, even if the remote source is gone, the image contains all the files it needs to run.

All cloud providers support running docker images and it usually it simplifies the development/deployment/testing cycle (fewer "Works on my machine !" scenarios).

Docker images add some overhead :

- usually need to run inside of a linux virtual machine (can only run natively on a linux host)
- contain a copy of all dependencies in every image
- building and updating the image takes non-trivial amount of time and is usually slower than building the app locally
- debugging inside of the image is not as straightforward as debugging locally

Without going into details (see official docs for that), here's a basic overview of the important concepts behind Docker.

### Docker image

Image is a self-contained application environment. A good analogy for an image would be a virtual machine _snapshot_ - image build basically installs all the libraries needed for the application to run inside of a virtual machine and then creates a snapshot.

You can then use the image ("VM snapshot") as a template to start one or more instances of a [docker container](#docker-container). You can also [upload this image](#uploading-docker-images) to a registry to start [docker containers](#docker-container) on different hosts.

Instead of manually installing things into a VM docker images are built from a script called [Dockerfile](#dockerfile).

You can build a docker image from CLI :

```bash
> docker build -t <image-name> .
```

### Dockerfile

Script that specifies how a [docker image](#docker-image) will be built (eg. copy source files from project folder, run dotnet build inside of Docker image).

An important feature of dockerfile is that it can reference existing docker images, eg. :

```Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:5.0
```

This means that the docker image will have all the stuff from the referenced image preinstalled (in this case a Linux machine with a .NET 5.0 SDK installed).

Example of commands you can do in Dockerfile :

```Dockerfile
# We need dotnet SDK so we reference a dotnet/sdk:5.0
FROM mcr.microsoft.com/dotnet/sdk:5.0

# changes the working directory inside of docker image to `/app`.
WORKDIR /app

# copies everything from current directory to docker image `WORKDIR` (ie. `/app/`).
COPY . .

# command calls `dotnet publish -c Release -o out` inside of docker image working directory (builds the app).
RUN dotnet publish -c Release -o out
```

### Docker container

Container is an instance of a docker image, it can be running or stopped.

To start a new container from an image :

```bash
> docker run -d --name <container-name> -p 8000:80 -p 8001:443 <image-name>
```

This will run start a new docker container `<container-name>` and expose ports `80` and `433` from the container to ports `8000` and `8001` on host. You can then open `localhost:8000` on host to access application running inside of the container.

The `-d` switch means that docker container should run in background and not capture your current terminal.

To get a list of all containers on the machine :

```bash
> docker ps -a

CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS        PORTS              NAMES
a56253f77395   random-api    "dotnet Random.API..."    30 hours ago   Up 30 hours   :::8000->80/tcp    random-api
0cb476024407   mariadb       "docker-entrypoint.s…"   10 days ago    Up 2 days     :::3306->3306/tcp  random-db
```

`-a` switch tells it to show stopped containers as well, by default it will only print the running ones.

Stop a container :

```bash
> docker stop <container-name|container-id>
```

Removing a docker container :

```docker
> docker rm <container-name>
```

Removing a container also deletes the data inside of it. If you had a database running in this container all the data would be lost.

To persist the data and share it between different containers (eg. update the database image to a new version will require starting a new container from updated image), you need to use [volumes](https://docs.docker.com/storage/volumes/).

To share files between host and docker image (eg. when developing, files that change often don't want to rebuild the image and start a new container on each change) you can use [binding mounts](https://docs.docker.com/storage/bind-mounts/) to expose your host paths to the docker image.

## Example Dockerfile for a .NET project

```Dockerfile
# syntax=docker/dockerfile:1
# First we create a "build" image
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env
# We will be working inside of the /app folder in docker image
WORKDIR /app

# The general idea behind this step is that you first copy `.csproj` and `.sln` files because they rarely change.
# Then you run `dotnet restore` which will pull nuget packages.
# Docker will then understand that if no files changed before `RUN dotnet restore`
# it can keep using a cached version of the image
# so you only need to run restore when `.csproj` or `.sln` changes.
COPY *.sln .
COPY **/*.csproj ./
# This is a bash script that will for each `.csproj` create folders named like the project and move
# the `.csproj` inside of it - it seems strange but the line above this one `COPY **/*.csproj` does not
# work the way you expect it to - instead of copying the entire folder structure with `.csproj` files
# it will put all *.csproj files in to the target folder root
# This is a hacky workaround around an issue in docker, if your project files use nested hierarchy
# this will not work and you will need to do some other workaround.
# See this issue for more info : https://github.com/moby/moby/issues/15858
RUN for file in $(ls *.csproj); do mkdir -p ./${file%.csproj}/ && mv $file ./${file%.csproj}/; done
RUN dotnet restore

# Now you copy all source files and call build
COPY . ./
RUN dotnet publish -c Release -o out

# Now create a runtime image and copy all the build artifacts from build image
FROM mcr.microsoft.com/dotnet/aspnet:5.0
WORKDIR /app
COPY --from=build-env /app/out ./

# Our runtime image will call dotnet Project.Name.API.dll on startup
ENTRYPOINT ["dotnet", "Project.Name.API.dll"]

```

This docker file will do a 2 stage image build, first it will create a temporary image with .NET SDK called build-env, it will copy the source inside of that image and build it.

Second stage will build the runtime image, this image will only contain the .NET runtime and the built project files.

To create a docker image in your project, add a `Dockerfile` to the root of your solution, then build and run the image :

```bash
> docker build -t <image-name>
> docker run --rm -d --name <container-name> <image-name> \
    -p 8000:80 \
    -e ASPNETCORE_ENVIRONMENT=Development
```

You can add additional environment variables for the docker container with more `-e` switches.

## Running ASP.NET core with HTTPS inside of docker

More information on this can be found [here](https://docs.microsoft.com/en-us/aspnet/core/security/docker-compose-https?view=aspnetcore-5.0), but a quick overview

### Create a dev certificate for localhost

On Windows :

```ps1
dotnet dev-certs https -ep $env:USERPROFILE/.aspnet/https/<certificate-name>.pfx -p crypticpassword
dotnet dev-certs https --trust
```

On MacOS/Linux :

```bash
> dotnet dev-certs https -ep ${HOME}/.aspnet/https/<certificate-name>.pfx -p crypticpassword
> dotnet dev-certs https --trust
```

### Pass HTTPS configuration to docker image

On Windows :

```ps1
> $env:HOME=$env:USERPROFILE
> docker run --rm -d --name <container-name> -p 8000:80 -p 8001:443 \
        -e ASPNETCORE_ENVIRONMENT=Development \
        -e ASPNETCORE_URLS="https://+;http://+" \
        -e ASPNETCORE_HTTPS_PORT=8001 \
        -e ASPNETCORE_Kestrel__Certificates__Default__Password="crypticpassword" \
        -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/<certificate-name>.pfx \
        -v ${HOME}/.aspnet/https:/https/ \
        <image-name>
```

On MacOs/Linux

```bash
> docker run --rm -d --name <container-name> -p 8000:80 -p 8001:443 \
        -e ASPNETCORE_ENVIRONMENT=Development \
        -e ASPNETCORE_URLS="https://+;http://+" \
        -e ASPNETCORE_HTTPS_PORT=8001 \
        -e ASPNETCORE_Kestrel__Certificates__Default__Password="<crypticpassword>" \
        -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/<certificate-name>.pfx \
        -v ${HOME}/.aspnet/https:/https/ \
        <image-name>
```

You should now be able to access HTTPS version of your site at `https://localhost:8001`.

## Publishing docker images

Docker images can be published to a [registry](https://docs.docker.com/registry/) - this is how you deploy an image - you publish it to a registry and then you reference the registry image on the server you want to start the image.

[Docker Hub](https://hub.docker.com/) is the default registry you are pulling public images from by default.

To publish an image you need to tag it as an image on remote registry and then push the tag :

```bash
> docker image tag <image-name> <registry-host>/<image-name>
> docker image push <registry-host>/<image-name>
```

## Useful tips when working with docker

### Rebuilding images and updating containers

`docker build` builds a new image with latest changes but it does not restart/update currently running containers.

Stop the existing docker image, remove it and then start it again from the latest version.

If you're doing this often enough in your workflow consider using [docker-compose](#docker-compose) which can automate these steps and more.

### Container shell access

During development it can be useful to run shell commands inside of docker image :

```bash
> docker exec -it <image-name> bash
```

### Creating an image from running container (manual image patching)

Create a container, do some modifications (eg. via [shell access](#container-shell-acess)), and create a new image :

```bash
> docker commit <image-name>
```

This image can then be pushed, started, etc.

Avoid using this regularly (images should build correctly from a Dockerfile), but in emergency situations or during development it's a tool to be aware of.

### Image tags

In all the places `<image-name>` is used in the instructions above `image-tag` can be added to uniquely represent an image version.

Eg. working on multiple branches, for each branch :

```bash
> docker build -t <image-name>:<feature-name>
> docker run --name <container-name-with-feature> <image-name>:<feature-name>
```

This will build a tagged version of the image and start a container from that version.

## Docker compose

docker-compose is a lightweight container orchestration tool, it's used to start multiple service containers and connect them together.

Compose is usually used to create an entire development environment, a typical docker-compose will start up a service for database, backend, frontend, HTTP proxy, etc., connect them all together in a private network and expose public ports on the host.

docker-compose can be used for creating production environments, but other solutions are often better suited for that and docker-compose is usually used to simplify setting up the development environment.

Compose configuration is contained within a [docker-compose.yml](#example-docker-compose.yml) script. This script references [Dockerfiles](#dockerfile) to be built locally or prebuilt images that will be started as a service. You can specify environment variables to be passed to the container inside of `docker-compose.yml`, volumes, port mappings, etc. For details see the [reference for docker-compose.yml](https://docs.docker.com/compose/compose-file/)

Some basic commands when using `docker-compose` :

- start services :

  ```bash
  > docker-compose up --build
  ```

  `--build` switch tells compose to rebuild any services that reference a [Dockerfile](#dockerfile) instead of an image.

- stop services :

  ```bash
  > docker-compose stop
  ```

- stop and remove all containers created by services :

  ```bash
  > docker-compose down
  ```

- it's possible to run multiple instances of a single service (for testing eg. load balancer) :

  ```bash
  docker-compose up --scale example-api=4
  ```

  this will spin up 4 instances of example-api container inside of compose network, note that you will need to handle networking configuration slightly differently : you can either map a range of `ports` or use `expose` setup a load balancer if you want to access the API on host.

### Compose networking

Like mentioned before, docker-compose creates a private network for it's services. Some helpful facts about this network :

- containers inside of a network can access other containers by service name (eg. you can connect to your database container from api on `example-db`)
- specifying `ports` mapping on a service, similar to [docker run -p](#docker-container), exposes the service ports to host machine
- specifying `expose` list on a service exposes those ports to internal network only (communication between services)
- you can access host network on `host.docker.internal`

## Example docker-compose.yml

```yml
version: "3.9"
services:
  example-api:
    build:
      context: .
      dockerfile: ./Example.API/Dockerfile
    restart: always
    ports:
      - "8000:80"
      - "8001:443"
    volumes:
      - ${HOME}/.aspnet/https/:/https/
      - logvolume:/var/log
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+;http://+
      - ASPNETCORE_HTTPS_PORT=8001
      - ASPNETCORE_Kestrel__Certificates__Default__Password=crypticpassword
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/example-api.pfx
      - ConnectionStrings__ExampleDb=Server=example-db,1433;Database=exampledb;User Id=sa;Password=crypticpassword;
    links:
      - example-db
    depends_on:
      - example-db

  example-migration:
    build:
      context: .
      dockerfile: ./Example.Data.Db/Dockerfile
    environment:
      - ConnectionStrings__ExampleDb=server=Server=example-db,1433;Database=exampledb;User Id=sa;Password=crypticpassword;
    restart: on-failure
    links:
      - example-db
    depends_on:
      - example-db

  example-db:
    image: mcr.microsoft.com/mssql/server:2019-latest
    restart: always
    ports:
      - "1433:1433"
    environment:
      - SA_PASSWORD=crypticpassword
      - ACCEPT_EULA=Y
    healthcheck:
      test: CMD /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P crypticpassword -Q "SELECT 1" || exit 1
      timeout: 20s
      retries: 10
      start_period: 10s
volumes:
  logvolume: {}
```

This script specifies 3 containers need to get started, `example-api` and `example-migration` are built locally from two separate [Dockerfiles](#dockerfile) and `example-db` is started from Microsoft SQL server 2019 image.

`example-migration` could then be configured to build a .NET project similar to `example-api` and run migrations from .NET CLI :

```dockerfile
# syntax=docker/dockerfile:1
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env
WORKDIR /app

RUN dotnet tool install -g dotnet-ef --version 5.0.10
ENV PATH $PATH:/root/.dotnet/tools

COPY *.sln ./
COPY **/*.csproj ./
RUN for file in $(ls *.csproj); do mkdir -p ./${file%.csproj}/ && mv $file ./${file%.csproj}/; done
RUN dotnet restore

# Run the migrations on database
COPY . ./
CMD dotnet ef database update -p Test.Data.Db -s Test.API
```
