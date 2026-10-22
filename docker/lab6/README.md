# Lab 6

## 1. Dockerfile and build

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

```bash
docker build -t ivolve1 .
```

## 2. container1: variables with -e, limited to 1 CPU and 256 MB

```bash
docker run -d --name container1 -p 5000:5000 -e APP_NAME=iVolveApp1 -e APP_ENV=stg --cpus="1" --memory="256m" ivolve1
curl http://localhost:5000
docker exec container1 env | grep APP_
docker stats container1 --no-stream
```
## 3. container2: variables from env.txt

`env.txt`:
```
APP_NAME=iVolveApp2
APP_ENV=prod
```

```bash
docker stop container1 && docker rm container1
docker run -d --name container2 -p 5000:5000 --env-file env.txt ivolve1
curl http://localhost:5000
docker exec container2 env | grep APP_
```

## 4. Variables hardcoded in the Dockerfile

`Dockerfile.env` a

```dockerfile
ENV APP_NAME=iVolveApp3
ENV APP_ENV=dev
```

```bash
docker stop container2 && docker rm container2
docker build -t ivolve3 -f Dockerfile.env .
docker run -d --name container2 -p 5000:5000 ivolve3
curl http://localhost:5000
docker exec container2 env | grep APP_
```
