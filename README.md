# Docker Multi-Service Container Setup

A hands-on learning project demonstrating **Docker networking** and **multi-service containers** using FastAPI, Express.js, and Nginx.

## 🎯 What I Built

Two separate Docker containers, each running **both an application server AND Nginx** as a reverse proxy:

1. **Container 1**: Ubuntu + FastAPI + Nginx (Python)
2. **Container 2**: Alpine + Express.js + Nginx (Node.js)

Both containers use the **default bridge network** (no custom network configuration needed).

## 🏗️ Architecture

Each container runs two services simultaneously using **supervisord**:

```
┌─────────────────────────────────────────┐
│  Container 1 (fastapi-nginx)            │
│                                         │
│  ┌──────────┐         ┌─────────────┐  │
│  │  Nginx   │ ──────> │  FastAPI    │  │
│  │  Port 80 │         │  Port 8000  │  │
│  └──────────┘         └─────────────┘  │
│       ↑                                 │
│       │ Exposed via -p 8001:80          │
└───────┼─────────────────────────────────┘
        │
   localhost:8001


┌─────────────────────────────────────────┐
│  Container 2 (express-nginx)            │
│                                         │
│  ┌──────────┐         ┌─────────────┐  │
│  │  Nginx   │ ──────> │  Express.js │  │
│  │  Port 80 │         │  Port 3000  │  │
│  └──────────┘         └─────────────┘  │
│       ↑                                 │
│       │ Exposed via -p 8002:80          │
└───────┼─────────────────────────────────┘
        │
   localhost:8002
```

### Key Concepts Demonstrated

✅ **Reverse Proxy Pattern**: Nginx sits in front of application servers, handling incoming requests  
✅ **Process Management**: Supervisord manages multiple processes in a single container  
✅ **Port Mapping**: Host ports mapped to container ports (`-p 8001:80`)  
✅ **Security**: Application servers not directly exposed to the outside world  
✅ **Lightweight Images**: Ubuntu 24.04 for FastAPI, Alpine 3.18 for Express

## 📁 Project Structure

```
docker-networking/
├── fastapi-app/
│   ├── Dockerfile           # Ubuntu-based multi-service container
│   ├── app.py              # FastAPI application (port 8000)
│   ├── requirements.txt    # Python dependencies
│   ├── nginx.conf          # Nginx reverse proxy config
│   └── supervisord.conf    # Process manager config
├── express-app/
│   ├── Dockerfile          # Alpine-based multi-service container
│   ├── app.js              # Express.js application (port 3000)
│   ├── package.json        # Node.js dependencies
│   ├── nginx.conf          # Nginx reverse proxy config
│   └── supervisord.conf    # Process manager config
└── README.md               # This file
```

## 🚀 How to Build and Run

### Step 1: Build Docker Images

```bash
# Build FastAPI container
cd fastapi-app
docker build -t fastapi-nginx .
cd ..

# Build Express.js container
cd express-app
docker build -t express-nginx .
cd ..
```

### Step 2: Run Containers

Open **two separate terminals**:

**Terminal 1 - FastAPI:**
```bash
docker run --rm --name fastapi-container -p 8001:80 fastapi-nginx
```

**Terminal 2 - Express.js:**
```bash
docker run --rm --name express-container -p 8002:80 express-nginx
```

### Step 3: Test the Endpoints

```bash
# Test FastAPI (through Nginx)
curl http://localhost:8001/
# Response: {"message": "Hello from container 1 - fastapi"}

# Test Express.js (through Nginx)
curl http://localhost:8002/
# Response: {"message": "Hello from container 2 - express.js"}
```

### Step 4: Stop Containers

Press `Ctrl+C` in each terminal window to gracefully stop the containers.

## 🔍 Traffic Flow Explained

When you visit `http://localhost:8001/`:

```
Browser Request
    ↓
Host Machine (port 8001)
    ↓
Docker Port Mapping (-p 8001:80)
    ↓
Container: Nginx (port 80)
    ↓
proxy_pass to http://127.0.0.1:8000
    ↓
Container: FastAPI (port 8000)
    ↓
Response flows back through the same path
```

## 📚 Key Technologies Used

| Technology | Purpose | Container |
|------------|---------|-----------|
| **FastAPI** | Python web framework | Container 1 |
| **Express.js** | Node.js web framework | Container 2 |
| **Nginx** | Reverse proxy server | Both |
| **Supervisord** | Process control system | Both |
| **Ubuntu 24.04** | Base OS | Container 1 |
| **Alpine 3.18** | Lightweight base OS | Container 2 |

## 🛠️ Important Commands

### List Running Containers
```bash
docker ps
```

### View Container Logs
```bash
docker logs fastapi-container
docker logs express-container
```

### Stop and Remove Containers (if running with -d)
```bash
docker stop fastapi-container express-container
docker rm fastapi-container express-container
```

### View Images
```bash
docker images | grep nginx
```

### Remove Images
```bash
docker rmi fastapi-nginx express-nginx
```

## 💡 What I Learned

### 1. **Multi-Service Containers**
   - How to run multiple processes in a single container using supervisord
   - When it's appropriate vs using Docker Compose

### 2. **Reverse Proxy Pattern**
   - Nginx forwards requests to application servers
   - Application servers (FastAPI/Express) are NOT directly exposed to the internet
   - This mirrors real-world production architecture

### 3. **Docker Networking Basics**
   - Port mapping with `-p HOST:CONTAINER`
   - Default bridge network behavior
   - Container isolation

### 4. **Process Management**
   - Supervisord as PID 1 in containers
   - Managing multiple processes (Nginx + App Server)
   - Graceful shutdown with `SIGINT` and `SIGTERM`

### 5. **Security Best Practices**
   - Only Nginx is exposed to the host
   - Application servers listen only on localhost within container
   - Defense in depth: requests must go through Nginx first

## 🐛 Troubleshooting

**Port already in use:**
```bash
# Kill the process using the port (macOS/Linux)
lsof -ti:8001 | xargs kill -9
lsof -ti:8002 | xargs kill -9
```

**Container name already exists:**
```bash
docker rm fastapi-container express-container
```

**Nginx fails to start:**
- Check nginx.conf syntax
- Ensure the application port (8000/3000) matches in both nginx.conf and the app

**Supervisor errors:**
- Check supervisord.conf paths
- Ensure all referenced files exist in the container

## 🎓 Next Steps

To further enhance this project, you could:

- [ ] Add Docker Compose for easier orchestration
- [ ] Create a custom Docker network for container-to-container communication
- [ ] Add SSL/TLS certificates for HTTPS
- [ ] Implement health checks in Dockerfiles
- [ ] Add environment variables for configuration
- [ ] Set up logging and monitoring
- [ ] Create a third Nginx container as a load balancer

## 📖 References

- [Docker Documentation](https://docs.docker.com/)
- [Nginx Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [Supervisord Documentation](http://supervisord.org/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Express.js Documentation](https://expressjs.com/)

---

**Author**: Bibhuti 
**Date**: December 25, 2024  
**Purpose**: Educational project to understand Docker multi-service containers and reverse proxy architecture
