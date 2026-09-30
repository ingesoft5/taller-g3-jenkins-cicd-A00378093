# =============================================================================
# Taller Evaluativo 2 - Fase 2: Empaquetamiento Inmutable Multi-Stage
# Backend: studytrack-api
# =============================================================================

# ---------- Stage 1: Builder ----------
# Imagen con Maven + JDK 17 (solo existe en la fase de compilación).
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder
WORKDIR /build

# 1) Copiamos SOLO el pom y descargamos dependencias: esta capa queda en
#    caché mientras el pom no cambie, aunque cambie el código fuente.
COPY pom.xml .
RUN mvn -B -q dependency:go-offline

# 2) Copiamos el código y empaquetamos (las pruebas ya corrieron en el
#    pipeline, por eso -DskipTests).
COPY src ./src
ARG APP_VERSION=0.0.1-SNAPSHOT
RUN mvn -B package -DskipTests -Drevision=${APP_VERSION}

# ---------- Stage 2: Runtime ----------
# Solo JRE (sin Maven, sin compilador) -> imagen pequeña y con menor superficie de ataque.
FROM eclipse-temurin:17-jre-alpine
ARG APP_VERSION=0.0.1-SNAPSHOT
LABEL org.opencontainers.image.title="studytrack-api" \
      org.opencontainers.image.version="${APP_VERSION}"

# Usuario sin privilegios
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=builder --chown=appuser:appgroup /build/target/studytrack-api.jar app.jar

USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
