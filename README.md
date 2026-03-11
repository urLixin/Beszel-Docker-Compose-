
---

# 📊 Beszel Monitoring Stack

A lightweight, high-performance monitoring solution. This repository contains the Docker Compose configuration to deploy the **Beszel Hub** and a **Local Agent** using a secure Unix Socket.

## 🚀 Quick Start

### 1. Create the Environment

First, create a directory and a `.env` file to store your access key.

```bash
mkdir beszel && cd beszel
touch .env

```

### 2. Docker Compose Configuration

Create a `docker-compose.yml` file and paste the following. This setup uses a shared volume for the socket, allowing the Hub and Agent to communicate without exposing additional network ports.

```yaml
services:
  beszel:
    image: henrygd/beszel:latest
    container_name: beszel
    restart: unless-stopped
    ports:
      - 8090:8090
    volumes:
      - ./beszel_data:/beszel_data
      - ./beszel_socket:/beszel_socket
  beszel-agent:
    image: henrygd/beszel-agent:latest
    container_name: beszel-agent
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./beszel_socket:/beszel_socket
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      LISTEN: /beszel_socket/beszel.sock
      # Do not remove quotes around the key
      KEY: ${KEY}
networks: {}

```

---

## 🛠️ Setup Instructions

### Step 1: Launch the Hub

Run the following command to start only the dashboard:

```bash
docker compose up -d beszel

```

### Step 2: Retrieve your Key

1. Open your browser to `http://your-ip:8090`.
2. Create your admin account.
3. Click **"Add System"** in the top right.
4. Copy the **Public Key** (e.g., `ssh-ed25519 AAAAC3...`).

### Step 3: Link the Local Agent

1. Open your `.env` file and paste the key:
```env
KEY="ssh-ed25519 YOUR_COPIED_KEY_HERE"

```


2. Start the agent:
```bash
docker compose up -d beszel-agent

```



---

## 🌐 Adding External Servers

To monitor a remote server (VPS, another homelab, etc.), you only need to run the Agent on that machine.

### **On the Remote Machine:**

Create a simple `docker-compose.yml`:

```yaml
services:
  beszel-agent:
    image: henrygd/beszel-agent:latest
    container_name: beszel-agent
    restart: unless-stopped
    network_mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      PORT: 45876
      KEY: "ssh-ed25519 YOUR_HUB_KEY"

```

### **In the Hub UI:**

1. Click **Add System**.
2. **Name:** Your server name.
3. **Host / IP:** The Public IP or Domain of the remote server.
4. **Port:** `45876` (Default).
5. *Ensure Port `45876` is open on the remote server's firewall.*

---

## 📝 Configuration Overview

| Service | Connection Type | Description |
| --- | --- | --- |
| **Local Agent** | Unix Socket | Communicates via `/beszel_socket/beszel.sock`. |
| **Remote Agent** | TCP/IP | Communicates via Port `45876`. |
| **Network Mode** | `host` | Required for the agent to access host hardware stats. |
| **Docker Socket** | Read-Only | Allows Beszel to show stats for individual containers. |

---

> **Tip:** If you are using a reverse proxy, point it to port `8090` of the Beszel Hub container. The agent does not need to be proxied. ⚡
