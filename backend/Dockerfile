# syntax=docker/dockerfile:1

# ---------- Stage 1: build jar (Gradle 8.14.3 + JDK 21 co san trong image) ----------
FROM gradle:8.14.3-jdk21 AS build
WORKDIR /app
COPY mirror.gradle /mirror.gradle
COPY settings.gradle build.gradle ./
COPY src ./src
RUN --mount=type=cache,target=/home/gradle/.gradle \
    gradle bootJar -x test --no-daemon -I /mirror.gradle \
    && find build/libs -name '*.jar' -not -name '*-plain.jar' -exec cp {} app.jar \;

# ---------- Stage 2: runtime chi co JRE + jar ----------
FROM eclipse-temurin:21-jre
WORKDIR /app
RUN groupadd --system spring && useradd --system --gid spring spring
COPY --from=build /app/app.jar app.jar
USER spring
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
