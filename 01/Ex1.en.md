# Building Applications in a Container

## Creating a Sample .NET Core Application

```bash
dotnet new mvc -n mydockerapp

cd mydockerapp

# Run the project
dotnet run

# Open the website in the browser
```

## We Want to Transfer the Following Process to Dockerfile

```bash
dotnet restore "mydockerapp.csproj"
dotnet build "mydockerapp.csproj" -c Release -o /out
dotnet publish "mydockerapp.csproj" -c Release -o /app

dotnet mydockerapp.dll
```

# Steps

## Step 1. Building the Application in a Container

First, create a `Dockerfile` in the current directory.

For this task, you'll need two sets of commands:

### Dockerfile Commands:
- `FROM XXX` - specifies the layer we are using. Where XXX is the docker image. For building, we'll use the image `mcr.microsoft.com/dotnet/sdk:6.0`
- `COPY [ "local_source", "container_dest" ]` - copies local files to the container. Brackets are important
- `WORKDIR` - specifies the working directory (commands will be executed relative to this directory)
- `RUN` - executes a command in the console inside the container

.NET Core Commands:
- `dotnet restore "XXX.csproj"` - restores .NET Core packages
- `dotnet build "XXX.csproj" -c Release -o /YYY` - builds the project
    Legend:
    - `XXX.csproj` - is the path to the csproj file
    - `YYY` - is the directory where we want to place the built artifacts

Console Commands:
- `docker build . -t MYTAG` - builds the Dockerfile existing in the current directory and gives it the tag MYTAG
- `docker images` - displays all images in the repository
- `docker images XXX` - displays all images named XXX

# Step 2. Publishing the Application in a Container

Dockerfile Commands:
- `FROM source_image AS name` - adding AS after the image name allows you to use a custom name for the docker image.

.NET Core Commands:
- `dotnet publish "XXX.csproj" -c Release -o /app` - prepares a .NET Core package ready for publication

# Step 3. Running the Application in a Container

Create a container that will use the published package and run our .NET Core application

Dockerfile Commands/Information:
- `EXPOSE 80` - exposes port 80 to the outside
- Image for running: `mcr.microsoft.com/dotnet/aspnet:5.0-buster-slim` - the previous one was the SDK.
- `COPY --from=XXX /app .` - `--from=XXX` where `XXX` is the name of the docker image from which we will be copying
- `ENTRYPOINT ["exec", "parameters"]` - specifies which command should be run when executing docker run

.NET Core Commands:
- `dotnet XXX.dll` - runs the .NET application

Console Commands:
- `docker run --rm -p LOCAL_PORT:DOCKER_PORT --name CONTAINER_NAME DOCKER_IMAGE` - creates and runs an instance of the docker container.

Where:
- `--rm` - after stopping, the container will be automatically removed
- `-p` - port mapping
- `--name` - assigns a custom name to the running container
