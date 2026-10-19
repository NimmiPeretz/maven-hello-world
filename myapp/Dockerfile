# Multistage build: compile, test and package inside Docker, ship only the JAR on a JRE.
# Build context = myapp/

# Stage 1: build 
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /build

# Resolve dependencies first so this layer is cached until pom.xml changes
COPY pom.xml .
RUN mvn -B -q dependency:go-offline

COPY src ./src
RUN mvn -B clean package

# Stage 2: artifact export 
FROM scratch AS artifact
COPY --from=build /build/target/myapp-*.jar /

# Stage 3: runtime
FROM eclipse-temurin:17-jre AS runtime

WORKDIR /app

# Dedicated non-root user with a fixed numeric UID/GID
# (numeric so Kubernetes can verify runAsNonRoot in the Helm chart)
RUN groupadd --system --gid 10001 appgroup \
 && useradd --system --uid 10001 --gid appgroup --no-create-home --shell /usr/sbin/nologin appuser

# Version-independent name inside the image; the version lives in the image tag
COPY --from=build /build/target/myapp-*.jar app.jar

USER 10001:10001

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
