# prometheus
---

# Deploying Prometheus with Docker and Subdomain (NGINX Reverse Proxy)

---

## Step 1: Prepare Directory

```bash
mkdir -p /opt/prometheus
cd /opt/prometheus
```

---

## Step 2: Create Prometheus Configuration

Create a file named `prometheus.yml`:

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

---

##  Step 3: Create Docker Compose File

Create a file named `docker-compose.yml`:

<pre><code>version: '3'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    restart: unless-stopped
</code></pre>

Then run:

```bash
docker-compose up -d
```

---

##  Step 4: Set Up DNS

Go to your DNS provider and create an **A record** for your subdomain:

| Type | Name       | Value               |
| ---- | ---------- | ------------------- |
| A    | prometheus | your.server.ip.addr |

---

##  Step 5: Configure NGINX Reverse Proxy

Install NGINX:

```bash
sudo apt install nginx -y
```

Create a config file:

```bash
sudo nano /etc/nginx/sites-available/prometheus
```

Paste the following:

<pre><code>server {
    listen 80;
    server_name prometheus.example.com;

    location / {
        proxy_pass http://localhost:9090/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
</code></pre>

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/prometheus /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

##  Step 6 (Optional): Add HTTPS with Let's Encrypt

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run Certbot:

```bash
sudo certbot --nginx -d prometheus.example.com
```

---

##  Step 7: Verify Access

Visit in your browser:

```
http://prometheus.example.com
```

or

```
https://prometheus.example.com
```

You should see the Prometheus dashboard!

---

##  Optional: Connect Prometheus to Grafana

In Grafana:

1. Go to **Settings → Data Sources → Add data source**
2. Choose **Prometheus**
3. In URL field, use:

   * `http://prometheus:9090` (if both run in the same Docker network), or
   * `http://prometheus.example.com` (if using external subdomain access)
4. Click **Save & Test**

