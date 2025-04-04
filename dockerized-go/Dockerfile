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
