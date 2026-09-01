# Hands-On L3: Installing Docker & Building a Multi-Container Microservice

## Overview
This repository contains the configuration and source code for setting up a local containerized environment. The purpose of this repo is to deploy a Python Flask web application with a Redis cache, and run a PostgreSQL database.

---

## Prerequisites
* Docker Desktop installed and running ([Download Docker](https://docs.docker.com/get-started/get-docker/))

Verify Docker is running:
```bash
docker --version
```

## Project Structure
```bash
.
├── .gitignore
├── README.md
├── Dockerfile
├── compose.yaml
├── requirements.txt
├── app.py
└── screenshots.pdf   # Document containing required output & Docker Desktop screenshots
```

## Step-by-Step Execution Guide

### Standalone PostgreSQL Container
Pull and run the official PostgreSQL image to verify container isolation and networking.
1.	Pull the PostgreSQL image:

```bash
docker pull postgres:16
```

2.	Start the PostgreSQL container:
  
```bash
docker run -d -p 5432:5432 --name postgres1 -e POSTGRES_PASSWORD=pass12345 postgres:16
```

(Note: If host port 5432 is already in use, bind to another port like -p 5433:5432).

3.	Access the container shell:
  
  ```bash
docker exec -it postgres1 bash
```

4.	Connect via psql:
```bash
psql -d postgres -U postgres
```
Exit psql using \q and then type exit to leave the container.

5.	Clean up PostgreSQL resources:
```bash
docker stop postgres1 && docker rm postgres1
```

### Multi-Container App (Flask + Redis)
1. Define dependencies
```bash
flask==3.0.3
redis==5.0.7
```
2. Define app.py
```bash
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```


3. Create a dockerfile to containerize application
```bash
FROM python:3.12-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```


4. Definse services
```bash
services:
  web:
    build: .
    ports:
      - "8000:5000"
    depends_on:
      - redis
  redis:
    image: "redis:alpine"
```
   
6. Build and run the appplication
```bash
docker compose up
```


8. Open the application in a browser and refresh a few times. To stop the stack press Ctrl + C

    http://localhots:8000


## Learning Outcomes
* Container Lifecycle Management
* Understand fundamentals of working with containerized environments 
* Hands-on experience pulling images, mounting network ports, passing environment variables, and accessing running container namespaces via docker exec.
* Learn to connect and orchestrate multiple components in a system



