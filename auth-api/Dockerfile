FROM golang:1.21-alpine AS build-stage
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o auth-api ./
FROM alpine:3.19
WORKDIR /app
COPY --from=build-stage /app/auth-api .
EXPOSE 8081
CMD ["./auth-api"]