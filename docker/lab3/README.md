# Lab 3

## Part 1: Build Inside Docker (`Dockerfile.1`)

```dockerfile
FROM maven:3.9-eclipse-temurin-17
RUN useradd -m app
USER app
WORKDIR /home/app
COPY --chown=app . .
RUN mvn package
EXPOSE 8080
CMD ["java", "-jar", "target/calculator.jar"]
```

### Steps

Build the image:

```bash
docker build -f Dockerfile.1 -t app1 .
```

![build app1](screenshots/01-build-app1.png)

Check the image size:

```bash
docker images app1
```

![app1 size](screenshots/02-app1-size.png)

Run the container and verify the application:

```bash
docker run -it --name container1 app1
```

![run container1](screenshots/03-container1-run.png)

---

## Part 2: Build Locally (`Dockerfile.2`)

```dockerfile
FROM eclipse-temurin:17-jre
RUN useradd -m app
USER app
WORKDIR /home/app
COPY target/calculator.jar .
EXPOSE 8080
CMD ["java", "-jar", "calculator.jar"]
```

### Steps

Build the application locally:

```bash
mvn package
ls target/calculator.jar
```

![local build](screenshots/05-local-build.png)

Build the image:

```bash
docker build -f Dockerfile.2 -t app2 .
```

![build app2](screenshots/06-build-app2.png)

Check the image size:

```bash
docker images app2
```

![app2 size](screenshots/07-app2-size.png)

Run the container and verify the application:

```bash
docker run -it --name container2 app2
```

![run container2](screenshots/08-container2-run.png)

---

## Image Size Comparison

| Image | Approach            | Base image                     | Size   |
| ----- | ------------------- | ------------------------------ | ------ |
| app1  | Build inside Docker | `maven:3.9-eclipse-temurin-17` | 534 MB |
| app2  | Build locally       | `eclipse-temurin:17-jre`       | 296 MB |
