# Stage 1: Build the Spring Boot application
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

COPY pom.xml .
RUN mvn -B dependency:go-offline

COPY src ./src

RUN mvn -B clean package -DskipTests


# Stage 2: Run the application
FROM eclipse-temurin:17-jre

WORKDIR /app

RUN groupadd --system app && \
    useradd --system --gid app app

COPY --from=build --chown=app:app \
    /app/target/todo-summary-assistant-0.0.1-SNAPSHOT.jar \
    /app/app.jar

USER app

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
