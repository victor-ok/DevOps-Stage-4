# Stage 1: Build the Go application
FROM golang:1.21-alpine AS build-stage
# Set the working directory
WORKDIR /app
# Copy go.mod and go.sum to download dependencies
COPY go.mod go.sum ./
# Download dependencies
RUN go mod download
# Copy the rest of the application code
COPY . .
# Build the Go application
RUN CGO_ENABLED=0 GOOS=linux go build -o auth-api ./
# Stage 2: Create a minimal runtime image
FROM alpine:3.19
# Set the working directory
WORKDIR /app
# Copy the built binary from the build stage
COPY --from=build-stage /app/auth-api .
# Expose port
EXPOSE 8081
# Run the application
CMD ["./auth-api"]