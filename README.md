# Optimizing Docker Images for Go Applications: A Multi-Stage Build Approach

Rasoul Zamani

## Abstract

Docker has revolutionized software deployment by enabling applications to run in isolated, lightweight containers, ensuring consistency across environments. However, inefficient Docker images can lead to bloated storage, slower deployments, and security vulnerabilities. Optimizing Docker images is crucial for enhancing performance, reducing attack surfaces, and minimizing resource usage that sequentially can reduce cost of our server in long term.

This article explores how to optimize a Docker file for a simple Go web application built with the Gin framework by leveraging multi-stage builds approach. Initially, common single-stage Dockerfile was used, which resulted in a docker image with size of 597MB. By implementing a multi-stage method, we reduced the image size to 18MB.

This article provides a step-by-step breakdown of the optimization process, comparing single-stage and multi-stage builds, analyzing the impact on performance and security, and discussing best practices for containerizing Go applications efficiently. By the end, readers will understand how to create smaller, faster, and more secure Docker images for their Go web apps.

keywords:Docker, multi-stage Dockerfile, Dockerize Go Web App, GO, Golang.

## 1. Introduction

In modern software development, Docker has become a foundational tool for building, shipping, and running applications across different environments. At its core, Docker is a containerization platform. It allows you to package your application along with all its dependencies, libraries, and configuration into a container. This guarantees that your app will run consistently, whether it's on a developer's laptop, a test server, or in the cloud. Containers isolate the application from the host system, solving the "it works on my machine" problem and enabling reproducible, reliable deployments.

Beyond just packaging, Docker offers fine-grained control over build environments through features like multi-stage builds. This allows developers to define multiple phases in a single Dockerfile; such as compiling the application in one stage, and producing a clean, minimal runtime image in another. The result is smaller, faster, and more secure containers, which are easier to deploy.

In this article, Go (Golang), a statically typed and compiled language developed at Google is used. Go is designed for speed, simplicity, and scalability, making it ideal for building web services, APIs, and backend systems. One of Go’s major strengths is its ability to produce self-contained binaries — executables that don’t require any runtime or external dependencies. This is a huge advantage when working with Docker, because it allows for ultra-minimal runtime images, often just a few megabytes in size.

The example code in this article uses Gin, a popular and lightweight web framework built on top of Go’s standard net/http package. Gin provides robust routing, middleware support, JSON handling, and more. When paired with Go's fast compile times and native concurrency features, Gin enables you to build web APIs and microservices that are not only fast and scalable but also clean and easy to maintain.

In this article, at first docker file proposed for a simple go web app code is presented. Then, docker file is optimized by the multi-stage technique and benefits of this approach are shown.

## 2 .Methodology: Optimizing Docker Image with Multi-Stage Builds

When deploying a Go application using Docker, the size of the final image plays a critical role in performance, security, and efficiency. Initially, we used a single-stage Docker build, which resulted in an unnecessarily large image. By adopting multi-stage builds, we significantly reduced the image size while maintaining all necessary functionalities.

1. Initial Approach: Single-Stage Build
   Our first Dockerfile followed a single-stage approach, meaning it contained everything required to build and run the Go application within the same image.It starts from golang image layer, and after setting working directory, install all dependencies need for this project based on go.mod file. In the next step all codes copied to image and then application is build. Finally a port with number of 8080 is exposed to stablish connection between docker and external world. The default command of image was set to run created executable file so whenever a container ran, the web app will be up and ready to serve.

Single-Stage Dockerfile:

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

EXPOSE 8080

# Run the application

CMD ["./main"]
```

Our go code for test this docker file is simple Gin get api written in main.go:

```
package main

import (
"net/http"

    "github.com/gin-gonic/gin"

)

func main() {
r := gin.Default()

    r.GET("/", func(c *gin.Context) {
    	c.JSON(http.StatusOK, gin.H{"message": "Hello, World!"})
    })

    r.Run(":8080")

}
```

after building the image by `docker build -t go-app .` command, we could create and run container from our base go app image:
`docker run -p 8080:8080 --name my-gin-app  go-app`
then we can test our app in browser by hitting `localhost:8080`.

Issues with the Single-Stage Build:
This approach works well for development but introduces several inefficiencies in production:

The first and most important issue of the previous Dockerfile is that the image size is large. The image contains everything required for development, including the Go compiler, build tools, and dependencies, which are not needed at runtime. By executing `docker image ls` command we could see the size of image is about 600 MB (it is exactly 597 MB in our example).
This large image in addition to use more space in the server (that mean more cost), will take more time to transfer and deploy. Therefore, has effect on deploy and CI/CD (continuous integration/continuous deployment) and increase time cost too.
In addition, because the image includes unnecessary libraries, attack surface and consequently security risks are increased.

2. Optimized Approach: Multi-Stage Build
   To address one-stage docker file issues, we introduced a multi-stage build, which separates the build process from the final runtime environment. This significantly reduces the image size while keeping it functional and secure. In stage one, app dependencies are installed, code base is copied and final executable app is created like previous docker file.Then in next stage, we start from alpine (lightweight linux image) and then by `COPY --from=builder /app/main .` we copy what we exactly need in our docker image from previous stage. Finally the port 8080 is exposed and command for running app is set as well.

Optimized Multi-Stage Dockerfile:

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

EXPOSE 8080

# Run the application

CMD ["./main"]
```

## 3. Key Improvements and Analysis

By adopting a multi-stage Docker build strategy, we unlocked a number of practical and measurable benefits. Beyond just reducing image size, this approach brings improvements in performance, maintainability, and security. In this section, we’ll dive into the key areas where multi-stage builds make a significant impact, from deployment speed and cost savings to cleaner architecture and a reduced attack surface.
Let’s explore these advantages in more detail:
3.1 Lower Storage Costs and Faster Deployment

One of the most immediate and measurable benefits of using multi-stage builds is the dramatic reduction in image size. In our case, we managed to shrink the Docker image from 597MB to just 18MB—a reduction of over 30x. This leaner image brings several operational advantages:

Faster Pulls and Pushes: Smaller images are quicker to transfer across networks. Whether pulling from a CI/CD pipeline or deploying to a Kubernetes cluster, the reduced size minimizes latency and speeds up deployment times. This is especially valuable in edge computing or bandwidth-constrained environments.

Reduced Storage Costs: Container registries like AWS ECR, Docker Hub, or Google Container Registry often charge based on storage usage. By trimming unnecessary build-time dependencies and files, organizations can lower storage bills, especially when working with many microservices or frequently updating images.

Improved CI/CD Performance: In continuous integration and delivery (CI/CD) pipelines, every second counts. Smaller images mean faster build, test, and deploy cycles, allowing teams to ship code faster and with more confidence.

Scalability Benefits: In scenarios where your application scales across multiple nodes or environments (like serverless platforms or container orchestrators), smaller images reduce the resource footprint, leading to better performance and quicker horizontal scaling.

3.2 Separation of Concerns (Cleaner Architecture and Maintainability)

Multi-stage Docker builds naturally enforce a cleaner, more modular architecture by separating the build environment from the runtime environment.

Clear Role Separation: The first stage (often called the “builder”) contains all the tools, compilers, and dependencies required to build the application — in this case, Go tooling, libraries, and source files. The second stage contains only what’s necessary to run the compiled application — typically just the binary and a minimal base image like Alpine Linux. This separation ensures that only essential artifacts make it into the production image.

Better Maintainability: The Dockerfile becomes easier to read, debug, and extend. Developers can focus on optimizing each stage independently, improving workflow clarity and reducing complexity over time.

Alignment with the SOC Principle: In software design, Separation of Concerns (SOC) is the idea of organizing code and systems so each part has a clear and distinct responsibility. Multi-stage Docker builds are a practical reflection of this principle, resulting in images that are logically organized and easier to reason about.

Simplified Debugging and Upgrades: When the runtime image is clean and minimal, it’s easier to spot issues, update dependencies, or swap out base images without risking unintended side effects from leftover build-time tools.

3.3 Enhanced Security Through Minimalism

Security is one of the most compelling reasons to use multi-stage builds — especially when combined with minimal base images like Alpine Linux.

Reduced Attack Surface: By stripping away unnecessary build tools, source code, and package managers, the final image becomes far less vulnerable to exploits. There are simply fewer binaries and libraries that can be compromised.

Minimized OS Footprint: Alpine Linux, commonly used in production stages, is designed for security and minimalism. It has a much smaller set of default packages compared to a full OS image, which means fewer known vulnerabilities and a lower likelihood of unpatched exploits.

No Compiler, No Risk: Build tools like the Go compiler are powerful but unnecessary — and potentially risky — in production. Removing them prevents misuse and blocks certain types of supply chain attacks.

Compliance-Friendly: For teams following security best practices or compliance standards (e.g., CIS Benchmarks, SOC2, ISO 27001), minimal images are easier to audit and lock down, making it simpler to meet organizational or regulatory requirements.

## Conclusion

By refactoring single-stage to a multi-stage Dockerfile, the Go application's image size was successfully optimized. The transformation resulted in a much smaller (about 18MB instead of 600MB), more secure, and highly efficient image, improving both performance and deployment speed in addition to reducing server costs.
For production environments, multi-stage builds should be considered best practice, as they help maintain a clean, lightweight, and secure Docker image.

## References

Docker, Get Started with Docker, https://docs.docker.com/get-started/

Merkel, Dirk. “Docker: Lightweight Linux Containers for Consistent Development and Deployment.” Linux Journal, 2014.

Turnbull, James. The Docker Book: Containerization is the new virtualization. James Turnbull, 2014.

Docker Docs, Multi-stage Builds, https://docs.docker.com/develop/develop-images/multistage-build/

Ellis, Alex. “Mutli-stage Docker Builds.” alexellis.io, 2017. https://blog.alexellis.io/mutli-stage-docker-builds/

Donovan, Alan A. A., and Brian W. Kernighan. The Go Programming Language. Addison-Wesley, 2015.

Pike, Rob. “Go at Google: Language Design in the Service of Software Engineering.” Google, 2012.

Go Documentation, https://golang.org/doc/

Google Cloud, Building Minimal Docker Containers with Go, https://cloud.google.com/blog/products/containers-kubernetes/containerize-this-not-that

Gin GitHub Repository, https://github.com/gin-gonic/gin

Sam Williams, “Building REST APIs with Gin in Go.” TutorialEdge.net, 2020. https://tutorialedge.net/golang/go-gin-gonic-crash-course/

McGrath, John Arundel, and Dave Coombs. Cloud Native Go: Building Web Applications and Microservices for the Cloud with Go and React. O'Reilly Media, 2017.

Golang Blog, Docker from Scratch in Go, https://blog.golang.org/docker

Deb, Chandrika. “Dockerize a Golang Gin Application.” dev.to, 2021. https://dev.to/chandrikadeb7/dockerize-a-golang-gin-application-4nae
