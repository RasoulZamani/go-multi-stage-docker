# Initial Docker File

for this docker file:

```
# Use the official Golang image as the base image
FROM golang:1.22-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy the Go modules files and download dependencies
COPY go.mod go.sum ./
RUN go mod download

# Copy the application code into the container
COPY . .

# Build the Go application
RUN go build -o main .

# Expose the application port
EXPOSE 8083

# Run the application
CMD ["./main"]
```

I got this results:

```
docker build -t gin-hello-world .
[+] Building 488.1s (11/11) FINISHED                       docker:default
 => [internal] load build definition from Dockerfile                 0.2s
 => => transferring dockerfile: 462B                                 0.0s
 => [internal] load metadata for docker.io/library/golang:1.22-alp  78.7s
 => [internal] load .dockerignore                                    0.1s
 => => transferring context: 2B                                      0.0s
 => [1/6] FROM docker.io/library/golang:1.22-alpine@sha256:1699c1  218.4s
 => => resolve docker.io/library/golang:1.22-alpine@sha256:1699c100  0.7s
 => => sha256:6d405dfc5fdf3a45df1529cf060b920041f52 1.92kB / 1.92kB  0.0s
 => => sha256:1699c10032ca2582ec89a24a1312d986a3f 10.30kB / 10.30kB  0.0s
 => => sha256:4129f51f28c9ae5de799b958ba2aaa8f92f26 2.08kB / 2.08kB  0.0s
 => => sha256:4d75fd4b73869ed224045c010cdec787 294.90kB / 294.90kB  36.7s
 => => sha256:afa154b433c7f72db064d19e1bcfa84ee 69.36MB / 69.36MB  196.7s
 => => sha256:1f3e46996e2966e4faa5846e56e76e3748b73 3.64MB / 3.64MB  8.3s
 => => extracting sha256:1f3e46996e2966e4faa5846e56e76e3748b7315e2d  0.5s
 => => sha256:5f837c998576dcb54bc285997f33fcc2166dff6a 126B / 126B  20.3s
 => => sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb557 32B / 32B  21.0s
 => => extracting sha256:4d75fd4b73869ed224045c010cdec78756eefb6752  0.2s
 => => extracting sha256:afa154b433c7f72db064d19e1bcfa84ee196ad291  16.3s
 => => extracting sha256:5f837c998576dcb54bc285997f33fcc2166dff6aa4  0.0s
 => => extracting sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb557  0.0s
 => [internal] load build context                                    0.3s
 => => transferring context: 541B                                    0.0s
 => [2/6] WORKDIR /app                                               6.0s
 => [3/6] COPY go.mod go.sum ./                                      2.1s
 => [4/6] RUN go mod download                                       99.8s
 => [5/6] COPY . .                                                   1.4s
 => [6/6] RUN go build -o main .                                    42.4s
 => exporting to image                                              35.5s
 => => exporting layers                                             35.0s
 => => writing image sha256:93a6364e8bd9ff6fbc7f65c4246bac7f65ac2cf  0.1s
 => => naming to docker.io/library/gin-hello-world
```

it created 597MB image.
after running it by `docker run -p 8083:8083 --name my-gin-app  gin-hello-world`
it create and execute container with 4.43MB of memory usage. output of `docker stats my-gin-app`:

```
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O     PIDS
6451570ecd56   my-gin-app   0.00%     4.465MiB / 3.805GiB   0.11%     52kB / 8.56kB   36.9kB / 0B   5
```

# multistage docker

docker file:

```
# Stage 1: Build the Go application
FROM golang:1.22-alpine AS builder

# Set the working directory inside the container
WORKDIR /app

# Copy the Go modules files and download dependencies
COPY go.mod go.sum ./
RUN go mod download

# Copy the application code into the container
COPY . .

# Build the Go application
RUN go build -o main .

# Stage 2: Create the final image
FROM alpine:latest

# Set the working directory inside the container
WORKDIR /app

# Copy the binary from the builder image
COPY --from=builder /app/main .

# Expose the application port
EXPOSE 8083

# Run the application
CMD ["./main"]
```

build output:

```
[+] Building 73.4s (15/15) FINISHED               docker:default
 => [internal] load build definition from Dockerfile        0.4s
 => => transferring dockerfile: 650B                        0.1s
 => [internal] load metadata for docker.io/library/alpine:  5.6s
 => [internal] load metadata for docker.io/library/golang:  3.3s
 => [internal] load .dockerignore                           0.3s
 => => transferring context: 2B                             0.0s
 => [builder 1/6] FROM docker.io/library/golang:1.22-alpin  0.0s
 => [stage-1 1/3] FROM docker.io/library/alpine:latest@sh  15.0s
 => => resolve docker.io/library/alpine:latest@sha256:a856  2.4s
 => => sha256:a8560b36e8b8210634f77d9f7f9e 9.22kB / 9.22kB  0.0s
 => => sha256:1c4eef651f65e2f7daee7ee78588 1.02kB / 1.02kB  0.0s
 => => sha256:aded1e1a5b3705116fa0a92ba074a5e0 581B / 581B  0.0s
 => => sha256:f18232174bc91741fdf3da96d850 3.64MB / 3.64MB  7.8s
 => => extracting sha256:f18232174bc91741fdf3da96d85011092  0.9s
 => [internal] load build context                           0.2s
 => => transferring context: 9.39kB                         0.1s
 => CACHED [builder 2/6] WORKDIR /app                       0.0s
 => CACHED [builder 3/6] COPY go.mod go.sum ./              0.0s
 => CACHED [builder 4/6] RUN go mod download                0.0s
 => [builder 5/6] COPY . .                                  1.9s
 => [builder 6/6] RUN go build -o main .                   57.8s
 => [stage-1 2/3] WORKDIR /app                              5.5s
 => [stage-1 3/3] COPY --from=builder /app/main .           2.0s
 => exporting to image                                      1.6s
 => => exporting layers                                     1.1s
 => => writing image sha256:e668f5a889e423ba130007c90b32c5  0.1s
 => => naming to docker.io/library/optim-go-app             0.2s
```

it create **18MB** image! compare to previous non-multistage with about 600 MB, it is more compact more than 30 times!
however memory usage is not effected and is still about 4.3MB
