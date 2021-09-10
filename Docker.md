# Docker for .NET

## Docker overview

The motivation behind using docker is having reproducible environments for your application development and deployment.

Instead of installing all the libraries, tools, setting configuration everywhere you want to run your application, you prepare a [Docker image](#docker-image) which contains everything your application needs to run (linux apps, add configuration files/resources, libraries, tools, etc.). This image then runs consistently between your development machine and production environment (cloud, random server). It's also self-contained and isolated so other applications running on your PC, or different versions of some tool won't affect your docker image.

All cloud providers support running docker images and it usually it simplifies the development/deployment/testing cycle (fewer "Works on my machine !" scenarios).

Docker images do add some overhead, they usually need to run inside of a linux virtual machine, they contain a copy of all dependencies in every image, building and updating the image takes time, debugging inside of the image is not as straightforward as debugging on your machine, etc.

Without going into details (see official docs for that), here's a basic overview of the important concepts behind Docker.

### Docker image

Image is a self-contained application environment. A good analogy for an image would be a virtual machine _snapshot_ - you install all the libraries you need for your application to run inside of a virtual machine and then create a snapshot.

You can then use the image ("VM snapshot") as a template to start one or more instances of a [docker container](#docker-container). You can also [upload this image](#uploading-docker-images) to a registry to start [docker containers](#docker-container) on different hosts.

Instead of manually installing things into a VM docker images are built from a script called [Dockerfile](#dockerfile).

You can build a docker image from CLI :

```bash
> docker build -t <image-name> .
```

### Dockerfile

This is a script file that specifies how a [Docker image](#docker-image) will be built (eg. copy source files from your project folder, run dotnet build inside of Docker image).

An important feature of dockerfile is that it can reference existing docker images, so for example you can say :

```Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:5.0
```

This means that the docker image will have all the stuff from the referenced image preinstalled (in this case you will have a Linux machine with a .NET 5.0 SDK installed).

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

To start a new container from an image do :

```bash
> docker run -d --name <container-name> -p 8000:80 -p 8001:443 <image-name>
```

This will run start a new docker container `<container-name>` and expose ports `80` and `433` from the container to ports `8000` and `8001` on host machine. You can then go to `localhost:8000` on your machine to access the application running inside of the container.

The `-d` switch for run means that docker container should run in background and not capture your current terminal.

To get a list of all containers on the machine run :

```bash
> docker ps -a

CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS        PORTS              NAMES
a56253f77395   random-api    "dotnet Random.API..."    30 hours ago   Up 30 hours   :::8000->80/tcp    random-api
0cb476024407   mariadb       "docker-entrypoint.s…"   10 days ago    Up 2 days     :::3306->3306/tcp  random-db
```

`-a` switch tells it to show stopped containers as well, by default it will only print the running ones.

Stop a container by running :

```bash
> docker stop <container-name|container-id>
```

It's important to know that if you remove a docker container all the data in the container will also be removed.

```docker
> docker rm <container-name>
```

If you had a database running in this container all the data would be lost. To persist the data and be able to share it between different containers (eg. if you update the database image to a new version and you want to start a new container running the new DB version with the old data), you need to use [volumes](https://docs.docker.com/storage/volumes/).

If you want to share files between host and docker image (eg. useful when developing and you have files that change often and you don't want to rebuild the image and restart the container) you can use [binding mounts](https://docs.docker.com/storage/bind-mounts/) to expose your host paths to the docker image.

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
COPY **/*.csproj .
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
COPY . .
RUN dotnet publish -c Release -o out

# Now create a runtime image and copy all the build artifacts from build image
FROM mcr.microsoft.com/dotnet/aspnet:5.0
WORKDIR /app
COPY --from=build-env /app/out .

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

### On MacOS/Linux

To get HTTPS running inside of docker you will need to create a dev certificate for localhost

```bash
> dotnet dev-certs https -ep ${HOME}/.aspnet/https/<certificate-name>.pfx -p <crypticpassword>
> dotnet dev-certs https --trust
```

Then when starting an image you need to pass extra environment variables to configure HTTPS

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

Just because you ran `docker build` to rebuild the image with your latest changes does not mean that your currently running container will automatically restart with that image.

You need to stop the existing docker image, remove it and then start it again which will use the latest image. If you're doing this often enough in your workflow consider using [docker-compose](#docker-compose) which can automate these steps and more.

### Container shell access

During development it can be useful to run shell commands inside of docker image, to do this run :

```bash
> docker exec -it <image-name> bash
```

### Creating an image from running container (manual image patching)

You can create a container, do some modifications (eg. via [shell access](#container-shell-acess)), and create an image from it :

```bash
> docker commit <image-name>
```

You can then push this image, etc. Avoid using this regularly (images should build correctly from a Dockerfile), but in emergency situations or during development it's a good trick to know.

### Image tags

In all the places I've used `<image-name>` in the instructions above you can add a `image-tag` to uniquely represent an image version, for example if you are working on multiple branches, for each branch you can :

```bash
> docker build -t <image-name>:<feature-name>
> docker run --name <container-name-with-feature> <image-name>:<feature-name>
```

This will build a tagged version of the image and start a container from that version.
