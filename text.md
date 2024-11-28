class: center, middle

# I wrote dockerfiles for a week. This is what I learned
## #6 will shock you!

<br />

### Loris Bognanni
#### Nov 2024

.bottom-notes[
<a href="https://github.com/LBognanni/docker-tips/blob/main/text.md"><img src="markdown.png" style="width:2em;" title="Read this presentation as markdown" /></a>
<a href="https://github.com/LBognanni/docker-tips"><img src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" style="width:2em;" title="Github repo" /></a> 
]

---

layout: true

.title[
    I wrote dockerfiles for a week. This is what I learned]

---

## What is Docker?
![Docker meme](tldr.png)
**TLDR:**
  - Docker is a tool that allows you to run applications in containers.
  - Containers are isolated environments that run on the host machine.
  - _It's not virtualization_ *
  - It solves the "it works on my machine" problem.


---

## Core Components
- 🖼️ **Images:** Read-only templates for containers. Will contain your app binaries.
- 📦️ **Containers:** Running instances of images.
- 📜 **Dockerfile:** Script to define an image.
- 🐋 **Docker Engine:** Runtime for building and running containers.

<br />

#### In practice:
 - We upload images to ECR (Elastic Container Registry).
 - We run containers in ECS or Fargate. 
 - Smaller images are faster to upload and deploy!


---

## Dockerfile Basics

A dockerfile (usually named `Dockerfile`, with no extension) is a text file containing a set of commands that define what the final image should look like

#### Key Commands
  - `FROM`: Specifies the base image.
  - `WORKDIR`: Sets the working directory.
  - `COPY`: Copy files into the image.
  - `RUN`: Execute commands at build time.
  - `CMD`: Specify the container's runtime command.

---

## Example Dockerfile

```dockerfile
FROM python:3.12
WORKDIR /usr/local/app

# Install the application dependencies
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copy in the source code
COPY src ./src
EXPOSE 5000

# Setup an app user so the container doesn't run as the root user
RUN useradd app
USER app

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80"]
```

This will create an image for an example Python web application.

---

## It's ✨layers✨ all the way down

- Each command in the dockerfile creates a new "layer"
- Think of layers as changes such as added files, etc in a VCS
- Docker is smart, when building it will cache layers if their contents have not changed
- Each layer adds a little overhead in terms of final size, depending on what it does

---

## The Builder pattern in Docker

Using layers to take advantage of caching and generating smaller images

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build_env
WORKDIR /App

# Copy everything
COPY . ./
# Restore as distinct layers
RUN dotnet restore
# Build and publish a release
RUN dotnet publish -c Release -o out

# Build runtime image
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /App
COPY --from=build_env /App/out .
ENTRYPOINT ["dotnet", "DotNet.Docker.dll"]

```

---

## Speed up your dev time with layers
#### Take advantage of caching:
- Split restoring dependencies and building
  - you can also `COPY` only your `.csproj`s or `package.json` 
  - and then `COPY` the rest of the files for building
- Split `RUN` commands while you're working on them
- Be careful about where your dockerfile lives
  - if it lives next to the code, when you update it it will implicitly invalidate the cache for _everything else_


---

## Top tips
- be smart about `COPY` and `RUN` pairs to improve your build times
- Use cache busting variables to rerun only part of the dockerfile when updating things like environment variables that don't invalidate the cache
- Where should I run my docker build from? It _does_ makes a big difference!
- Unsure of what files ended up in your docker image? Use [Dive](https://github.com/wagoodman/dive) to inspect your image's layers

---

## Top tips (cont'd)



- If you created/edited your shell script file in Windows, make sure it was saved with line ending in Unix format

- use `set -e` in shell scripts to crash the container if the script fails, otherwise the default way shell scripts work is to ignore errors and continue

![](pain.png)

---

## Docker image Optimisation
- Use small base images (e.g. `alpine` based images)
- Don't build in the distributable docker stage
	- use a build stage instead, or build outside the container
- Try to run all your shell commands in one go (or use a shell script)
	- especially commands that change many files like `apt-get install` (once you're done hacking)
- Use the `.dockerignore` file to avoid copying stuff you don't need into the container

---
layout: true

---
class: center, middle

# 🎉 Thanks for listening!
### Comments? Questions?