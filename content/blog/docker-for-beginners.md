---
title: "Docker for Hungry Beginners"
date: 2026-01-21
draft: false
tags: [docker, containers, devops, beginners]
---

Picture this. You have a Python script. It needs specific versions: Python 3.11, pandas 2.1.3, and a Postgres database. It runs perfectly on your laptop. You send it to your colleague Blue. She spends two hours installing the right Python version, debugging why pandas won't install, and figuring out why Postgres won't start. Eventually she gets it working. Then she sends it to Red, and the process starts again. BTW we're going full Reservoir Dogs with this one (Excuse my calculated clumsiness).

This is the problem Docker solves. It packages your application with all its dependencies into a single unit called a container. That container runs exactly the same way on any computer with Docker installed.

## What Exactly is a Container?

A container includes:
- Your application code
- The programming language runtime (Python, Node.js, etc.)
- System libraries and tools
- Environment variables and configuration

Everything the application needs to run is inside the container. The host computer only needs Docker installed.

## The Dockerfile: Your Blueprint

The Dockerfile is a text file that describes how to build your container. It's a list of instructions. Each instruction creates a layer in the container.

Here's a simple Dockerfile for a Python application:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

Let's break this down line by line:

`FROM python:3.11-slim` - Start with a base image that has Python 3.11 installed. The `slim` version is smaller than the full image.

`WORKDIR /app` - Set the working directory inside the container to `/app`. All following commands will run from here.

`COPY requirements.txt .` - Copy the `requirements.txt` file from your computer into the container.

`RUN pip install --no-cache-dir -r requirements.txt` - Install the Python packages listed in requirements.txt.

`COPY . .` - Copy all files from your current directory into the container.

`CMD ["python", "app.py"]` - Specify the command to run when the container starts.

## Building and Running

Save this as `Dockerfile` in your project directory. Make sure you have a `requirements.txt` file with your dependencies and an `app.py` file.

Build the image:
```bash
docker build -t my-python-app .
```

The `-t` flag tags the image with a name (`my-python-app`). The `.` tells Docker to look for the Dockerfile in the current directory.

Run the container:
```bash
docker run -p 5000:5000 my-python-app
```

The `-p 5000:5000` maps port 5000 inside the container to port 5000 on your computer. If your app listens on port 5000 inside the container, you can access it at `http://localhost:5000`.

## The Big Benefit: Consistency

Run this same container on your laptop, a coworker's computer, a testing server, or a production server. It behaves identically everywhere. No more "works on my machine" discussions.

## Docker Compose for Multiple Services

Most applications need more than one service. A web app might need a web server, a database, and a cache. Docker Compose lets you define and run multiple containers together.

Create a `docker-compose.yml` file:

```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secretpassword
      POSTGRES_DB: mydatabase
```

This file defines two services: `web` (your application) and `db` (a PostgreSQL database).

The `web` service is built from the Dockerfile in the current directory (`.`). It exposes port 8000. The `depends_on` line ensures the database starts before the web application.

The `db` service uses the official PostgreSQL 15 image. It sets environment variables for the database password and name.

Run everything with one command:
```bash
docker-compose up
```

Docker Compose starts both containers and sets up networking between them. Your web application can connect to the database using the hostname `db` (the service name in the YAML file).

## How Containers Talk to Each Other

When you run containers with Docker Compose, they join a virtual network. Each service can reach the others using the service name as the hostname.

In your application code, connect to the database like this:
```python
import psycopg2

# 'db' is the service name from docker-compose.yml
conn = psycopg2.connect(
    host="db",
    database="mydatabase",
    user="postgres",
    password="secretpassword"
)
```

This works because Docker sets up DNS resolution between containers. The hostname `db` resolves to the database container's IP address.

## Persistent Data

Containers are ephemeral by default. When a container stops, all data inside it is lost. For databases, you need to persist data outside the container.

Docker volumes solve this. They store data on the host machine and mount it into the container.

Update your `docker-compose.yml`:
```yaml
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secretpassword
      POSTGRES_DB: mydatabase
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

The `volumes` section creates a named volume called `postgres_data`. It's mounted at `/var/lib/postgresql/data` inside the container, where PostgreSQL stores its data.

Now when you stop and restart your containers, the database data persists.

## Common Commands You'll Use

`docker ps` - List running containers
`docker stop container_name` - Stop a running container
`docker rm container_name` - Remove a stopped container
`docker images` - List downloaded images
`docker rmi image_name` - Remove an image
`docker-compose down` - Stop and remove all containers defined in docker-compose.yml
`docker logs container_name` - View container output
`docker exec -it container_name bash` - Open a shell inside a running container

## A Complete Example

Let's containerize a simple Flask application with a PostgreSQL database.

Project structure:
```
flask-app/
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── app.py
└── init.sql
```

`requirements.txt`:
```
Flask==3.0.0
psycopg2-binary==2.9.9
```

`app.py`:
```python
from flask import Flask, jsonify
import psycopg2
import os

app = Flask(__name__)

def get_db_connection():
    conn = psycopg2.connect(
        host="db",
        database="mydatabase",
        user="postgres",
        password=os.getenv("DB_PASSWORD")
    )
    return conn

@app.route('/')
def index():
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute('SELECT version()')
    version = cur.fetchone()[0]
    cur.close()
    conn.close()
    return jsonify({'postgres_version': version})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```

`Dockerfile`:
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

`docker-compose.yml`:
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      DB_PASSWORD: mysecretpassword
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: mydatabase
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Run it:
```bash
docker-compose up
```

Open `http://localhost:8000` in your browser. You'll see the PostgreSQL version returned as JSON. Every computer that runs these commands gets exactly the same result.

## What's Next

This covers the basics of running applications with Docker. The next step is learning to optimize your Docker images for size and speed, and understanding how to deploy containers to production environments.

The key takeaway: Docker eliminates environment differences as a source of bugs. You define the environment once in the Dockerfile, and it works consistently everywhere.


*This is the first article in a Docker series. Next: making Docker images smaller and faster.*