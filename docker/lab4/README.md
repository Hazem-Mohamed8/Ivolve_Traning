# Lab 4:

## Steps

### 1. Build the original image

```bash
docker build -t image1 .
docker images image1
```

### 2. Run and verify container1

```bash
docker run -d --name container1 -p 8080:8080 image1
curl localhost:8080
docker stop container1 && docker rm container1
```

### 3. Multi-stage Dockerfile

```dockerfile
# build
FROM maven:3.9.15-eclipse-temurin-17-alpine AS build
WORKDIR /app
COPY . .
RUN mvn package -DskipTests

# run
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
RUN adduser -D appuser
COPY --from=build /app/target/demo-0.0.1-SNAPSHOT.jar app.jar
USER appuser
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

### 4. Build the optimized image

```bash
docker build -t image2 .
docker images | grep -E "image1|image2"
```

| Image  | Size   |
| ------ | ------ |
| image1 | 420 MB |
| image2 | 204MB  |

### 5. Run container2 and verify non-root

```bash
docker run -d --name container2 -p 8080:8080 image2
curl localhost:8080
docker exec container2 whoami      # appuser
docker exec container2 id
docker stop container2 && docker rm container2
```

### 6. Export and load the image

```bash
docker save -o lab-image.tar image2
docker rmi image2
docker load -i lab-image.tar
docker images image2
```
