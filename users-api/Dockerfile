# Stage 1: Build the Spring Boot application
FROM maven:3.8-openjdk-8 AS build-stage
# Set the working directory
WORKDIR /app
# Copy the pom.xml and source code
COPY pom.xml .
COPY src ./src
# Build the application
RUN mvn clean package -DskipTests
# Stage 2: Create a minimal runtime image
FROM openjdk:8-jdk-slim
# Set the working directory
WORKDIR /app
# Copy the JAR file from the build stage
COPY --from=build-stage /app/target/*.jar app.jar
# Expose the application port
EXPOSE 8083
# Run the application
CMD ["java", "-jar", "/app/app.jar"]
