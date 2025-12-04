# Building ShinyProxy Operator

This guide provides multiple options for building the ShinyProxy Operator without needing to install build tools locally.

## Prerequisites

Choose one of the following options:
- **Option 1**: Docker (recommended for most users)
- **Option 2**: Devbox (for development environments)
- **Option 3**: Local build (requires JDK 21 and Maven)

---

## Option 1: Build with Docker (Recommended)

This is the easiest option if you have Docker installed.

### Quick Build with Docker Compose

```bash
# Build the project
docker-compose -f docker-compose.build.yml up

# The JAR will be in ./target/shinyproxy-operator-jar-with-dependencies.jar
```

### Alternative: Build with Dockerfile Directly

```bash
# Build the Docker image
docker build -f Dockerfile.build -t shinyproxy-operator-builder .

# Run the build
docker run --rm -v "$(pwd)/target:/build/target" shinyproxy-operator-builder

# The JAR will be in ./target/shinyproxy-operator-jar-with-dependencies.jar
```

### Extract JAR from Container (if needed)

```bash
# Start the container
docker run -d --name builder shinyproxy-operator-builder

# Copy the JAR out
docker cp builder:/build/target/shinyproxy-operator-jar-with-dependencies.jar .

# Clean up
docker rm builder
```

---

## Option 2: Build with Devbox

[Devbox](https://www.jetify.com/devbox) provides an isolated development environment without Docker.

### Install Devbox

```bash
curl -fsSL https://get.jetify.com/devbox | bash
```

### Build the Project

```bash
# Enter the devbox shell (installs JDK 21 and Maven automatically)
devbox shell

# Build the project
devbox run build

# Or build without tests (faster)
devbox run build-fast

# The JAR will be at ./target/shinyproxy-operator-jar-with-dependencies.jar
```

---

## Option 3: Local Build

If you have JDK 21 and Maven installed locally:

```bash
mvn -U clean install
```

The JAR will be at `target/shinyproxy-operator-jar-with-dependencies.jar`.

---

## Verifying the Build

After building, verify the JAR exists:

```bash
ls -lh target/shinyproxy-operator-jar-with-dependencies.jar
```

You should see a file of approximately 50-100 MB.

---

## Running the Built JAR

```bash
java -jar target/shinyproxy-operator-jar-with-dependencies.jar
```

---

## Building a Docker Image

If you want to create a Docker image with the operator:

```bash
# First build the JAR using one of the methods above

# Then create a runtime Dockerfile (example):
cat > Dockerfile.runtime <<'EOF'
FROM eclipse-temurin:21-jre-alpine

WORKDIR /opt/shinyproxy-operator

COPY target/shinyproxy-operator-jar-with-dependencies.jar shinyproxy-operator.jar

ENTRYPOINT ["java", "-jar", "shinyproxy-operator.jar"]
EOF

# Build the runtime image
docker build -f Dockerfile.runtime -t shinyproxy-operator:local .
```

---

## Troubleshooting

### Docker build fails with network errors

If you experience network issues during the Maven build:

```bash
# Build with host network (Linux only)
docker build --network=host -f Dockerfile.build -t shinyproxy-operator-builder .
```

### Permission issues with target directory

```bash
# On Linux/Mac, you might need to fix permissions
sudo chown -R $USER:$USER target/
```

### Out of memory during build

```bash
# Increase Docker memory limit or add Maven options
docker run --rm -v "$(pwd)/target:/build/target" \
  -e MAVEN_OPTS="-Xmx2048m" \
  shinyproxy-operator-builder
```

---

## Development Workflow

For ongoing development:

1. **Use Devbox** for the best development experience with automatic dependency management
2. **Use Docker Compose** for one-off builds without installing tools
3. **Use Local Build** if you already have the tools installed

---

## Testing Your Changes

After building with your environment variable changes:

1. Create a test docker-compose.yml with the operator
2. Set `SHINYPROXY_ENV_*` environment variables
3. Verify they are passed to the ShinyProxy container

Example:
```yaml
services:
  shinyproxy-operator:
    image: shinyproxy-operator:local
    environment:
      - SPO_DOCKER_GID=999
      - SHINYPROXY_ENV_TEST_VAR=test_value
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
