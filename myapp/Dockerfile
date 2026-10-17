# Runtime image for the myapp JAR built by Maven (mvn package -> target/myapp-<version>.jar)
FROM eclipse-temurin:17-jre

WORKDIR /app

# Dedicated non-root user with a fixed numeric UID/GID
# (numeric so Kubernetes can verify runAsNonRoot later in the Helm step)
RUN groupadd --system --gid 10001 appgroup \
 && useradd --system --uid 10001 --gid appgroup --no-create-home --shell /usr/sbin/nologin appuser

# Version-independent name inside the image; the version lives in the image tag
COPY target/myapp-*.jar app.jar

USER 10001:10001

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
