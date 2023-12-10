---
title: "Fix Docker Container Golang Execute Format Error"
date: 2023-12-10T20:12:21+11:00
draft: false
---

Bumped into error `Exec format error` when tried to run a Go binary within a Docker container. The Go binary is compiled from my M2 Macbook. M2 Macbook uses ARM CPU arch. As a result the Go binary was by default complied with ARM CPU arch, which is different from the Docker image's amd64 CPU arch. Hence the execution format error. To fix it, you can take two approaches.

First, we can build the Go binary within the Docker image. To do this, we add the build steps into Dockerfile as below.
```Dockerfile
COPY go.mod go.sum ./
COPY *.go ./
RUN go build -o /solar
```
This fixes the error, as now the binary is compiled within the container. The downside is a large image with unncessary dependencies installed.

Another way to solve this issue is to compile the Go binary to AMD CPU arch using Environment Variable. To do this, simply build the binary in MacOS with this command.
```Shell
CGO_ENABLED=- GOOS=linux go build -o bin/solar
```
`CGO_ENABLED=-` disables CGO tool during the build process. CGO allows calling C functions from Go code and vice versa. Disabling cgo creates a statically linked binary without any dependencies on external C libaries. This is perfect for our container scencary. `GOOS=linux` is pretty easy to understand, just means we build the binary for linux OS. With these two Environment Variable set, `go build` command compiles the binary for `amd64` CPU arch.
