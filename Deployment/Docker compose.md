`docker-compose` is a tool built on top of docker and is used to start multiple service containers and connect them together.

Compose is usually used to create an entire development environment: A typical docker-compose will start up a service for the database, backend, frontend, HTTP proxy and so on. Then you need to connect them all in a private network and expose public ports on the host.

docker-compose can be used to create production environments, but other solutions are often better suited for that. Because of that, docker-compose is usually used to simplify setting up the development environment.

Compose configuration is contained within a docker-compose.yml script. This script references 'Dockerfiles' which can be built locally or prebuilt images which can be started as a service. You can specify environment variables to be passed to the container inside of the `docker-compose.yml` file, volumes, port mappings, etc. For details see the [reference for docker-compose.yml](https://docs.docker.com/compose/compose-file/)

Some basic commands when using `docker-compose` :

- start services :

  ```bash
  > docker-compose up --build
  ```

  `--build` switch tells compose to rebuild any services that reference a `Dockerfile` instead of an image.

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

  This will spin up 4 instances of the example-api container inside of compose network. Note that you will need to handle networking configuration slightly differently: you can either map a range of `ports` or use `expose` setup a load balancer if you want to access the API on a host.

### Compose networking

As mentioned before, docker-compose creates a private network for it's services. Some helpful facts about this network :

- containers inside of a network can access other containers by service name (eg. you can connect to your database container from API on `example-db`)
- specifying `ports` mapping on a service, similar to `docker run -p` exposes the service ports to the host machine
- specifying `expose` list on a service exposes those ports to the internal network only (communication between services)
- you can access the host network on `host.docker.internal`

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

This script specifies 3 containers that need to get started, `example-api` and `example-migration` are built locally from two separate `Dockerfiles` and `example-db` is started from a Microsoft SQL server 2019 image.

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
