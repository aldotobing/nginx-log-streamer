# 🌐 Nginx Log Streamer

A real-time log streaming solution for Nginx access logs using WebSockets. Stream your Nginx logs directly to your browser or application with minimal setup.

![Nginx Log Streamer Demo](https://img.shields.io/badge/status-active-success.svg)
![Docker Build](https://img.shields.io/docker/cloud/build/yourusername/nginx-log-streamer)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

## 🌟 Web UI Integration

This project pairs perfectly with the [Nginx Log Analyzer Web UI](https://github.com/aldotobing/nginx-log-analyzer-web-ui) - a powerful, interactive dashboard for visualizing and analyzing your Nginx logs in real-time.

### Key Features of the Web UI:
- Real-time log visualization with interactive charts
- Filter logs by date, IP address, request type, and more
- Traffic analytics and performance metrics
- User-friendly interface with dark/light themes
- Secure WebSocket (WSS) support

### How They Work Together:
1. **Log Streamer** (this project) runs as a WebSocket server that tails your Nginx logs
2. **Web UI** connects to the WebSocket server and displays the logs in a beautiful, interactive dashboard
3. Together, they provide a complete solution for real-time log monitoring and analysis

### Quick Start with Web UI:

1. **Deploy the Log Streamer** (this project) following the instructions above

2. **Set up the Web UI**
   ```bash
   # Clone the Web UI repository
   git clone https://github.com/aldotobing/nginx-log-analyzer-web-ui.git
   cd nginx-log-analyzer-web-ui
   
   # Install dependencies
   npm install
   
   # Start the development server
   npm run dev
   ```

3. **Configure the Web UI**
   - Open the Web UI in your browser (default: http://localhost:3000)
   - Go to Settings
   - Enter your WebSocket URL (e.g., `ws://your-server-ip:1234` or `wss://your-domain.com:1234` for SSL)
   - Save the settings

4. **View Real-time Logs**
   - The dashboard will automatically connect to your log streamer
   - Monitor logs in real-time with interactive visualizations

## ✨ Features

- **Real-time Log Streaming**: Get instant updates of Nginx access logs
- **Secure WebSocket Support**: Optional WSS (WebSocket Secure) with SSL/TLS
- **Docker & Docker Compose Ready**: Easy deployment with containerization
- **Lightweight**: Built with Node.js and ws for optimal performance
- **Configurable**: Customize log file paths, ports, and number of initial lines

## 🚀 Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Nginx installed and running
- (Optional) SSL certificates for secure WebSocket connections

### 1. Clone the Repository

```bash
git clone https://github.com/aldotobing/nginx-log-streamer.git
cd nginx-log-streamer
```

### 2. Configure Environment Variables

Create a `.env` file in the project root:

```env
# WebSocket server port (default: 1234)
WS_PORT=1234

# Path to Nginx access log file
LOG_FILE_PATH=/var/log/nginx/access.log

# Number of initial log lines to send on connection
TAIL_LINES=50

# Optional: SSL certificate paths (for WSS)
# SSL_CERT_PATH=/etc/letsencrypt/live/yourdomain.com/fullchain.pem
# SSL_KEY_PATH=/etc/letsencrypt/live/yourdomain.com/privkey.pem
```

### 3. Build and Run with Docker Compose

```bash
docker-compose up -d --build
```

### 4. Access the WebSocket Server

Your WebSocket server will be available at:
- `ws://your-server-ip:1234` (HTTP)
- `wss://your-domain.com:1234` (HTTPS, if SSL configured)

## 🔌 Client Implementation Example

### JavaScript Browser Client

```html
<!DOCTYPE html>
<html>
<head>
    <title>Nginx Log Viewer</title>
    <style>
        #logs {
            font-family: monospace;
            white-space: pre;
            height: 80vh;
            overflow-y: auto;
            background: #1e1e1e;
            color: #f0f0f0;
            padding: 10px;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <h1>Nginx Access Logs</h1>
    <div id="logs"></div>

    <script>
        const logs = document.getElementById('logs');
        
        // Use wss:// for secure connections
        const wsProtocol = window.location.protocol === 'https:' ? 'wss://' : 'ws://';
        const ws = new WebSocket(`${wsProtocol}${window.location.hostname}:1234`);
        
        ws.onmessage = function(event) {
            const logEntry = document.createElement('div');
            logEntry.textContent = event.data;
            logs.prepend(logEntry);
            
            // Keep only the last 1000 log entries
            while (logs.children.length > 1000) {
                logs.removeChild(logs.lastChild);
            }
        };
        
        // Send periodic pings to keep connection alive
        setInterval(() => {
            if (ws.readyState === WebSocket.OPEN) {
                ws.send(JSON.stringify({ type: 'ping' }));
            }
        }, 30000);
        
        ws.onclose = function() {
            logs.prepend('Connection closed. Attempting to reconnect...');
            setTimeout(() => window.location.reload(), 5000);
        };
    </script>
</body>
</html>
```

## 🔧 Configuration Options

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `WS_PORT` | No | `1234` | Port for WebSocket server |
| `LOG_FILE_PATH` | Yes | - | Path to Nginx access log file |
| `TAIL_LINES` | No | `50` | Number of initial log lines to send |
| `SSL_CERT_PATH` | No | - | Path to SSL certificate file |
| `SSL_KEY_PATH` | No | - | Path to SSL private key file |

### Nginx Configuration (Reverse Proxy)

For production use, it's recommended to set up Nginx as a reverse proxy:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    # WebSocket proxy
    location /ws/ {
        proxy_pass http://localhost:1234;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 1d;
    }
    
    # Serve the HTML client
    location / {
        root /path/to/your/html/client;
        try_files $uri /index.html;
    }
}
```

## 🛠 Development

### Local Development

1. Install dependencies:
   ```bash
   npm install
   ```

2. Start the server:
   ```bash
   WS_PORT=1234 LOG_FILE_PATH=/var/log/nginx/access.log TAIL_LINES=50 node server.js
   ```

### Building the Docker Image

```bash
docker build -t nginx-log-streamer .
```

### Running Tests

```bash
# Coming soon
# npm test
```

## 📦 Deployment

### Direct Docker Deployment with SSL

You can run the container directly with Docker using your own SSL certificates without needing an Nginx reverse proxy:

```bash
# Build the image
docker build -t yourusername/nginx-log-streamer:latest .

# Run the container with SSL support
docker run -d \
  --name nginx-log-streamer \
  --restart always \
  -e WS_PORT=1234 \
  -e LOG_FILE_PATH=/var/log/nginx/access.log \
  -e TAIL_LINES=50 \
  -e SSL_CERT_PATH=/etc/letsencrypt/live/your-domain.com/fullchain.pem \
  -e SSL_KEY_PATH=/etc/letsencrypt/live/your-domain.com/privkey.pem \
  -v /var/log/nginx:/var/log/nginx:ro \
  -v /etc/letsencrypt:/etc/letsencrypt:ro \
  -p 1234:1234 \
  yourusername/nginx-log-streamer:latest
```

**Note:**
- Replace `your-domain.com` with your actual domain name
- Ensure the SSL certificate files exist at the specified paths
- The container needs read access to both the Nginx logs and SSL certificates
- The `--restart always` flag ensures the container restarts automatically if it stops

### Kubernetes

Example deployment configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-log-streamer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-log-streamer
  template:
    metadata:
      labels:
        app: nginx-log-streamer
    spec:
      containers:
      - name: nginx-log-streamer
        image: yourusername/nginx-log-streamer:latest
        ports:
        - containerPort: 1234
        env:
        - name: WS_PORT
          value: "1234"
        - name: LOG_FILE_PATH
          value: "/var/log/nginx/access.log"
        - name: TAIL_LINES
          value: "50"
        volumeMounts:
        - name: nginx-logs
          mountPath: /var/log/nginx
          readOnly: true
      volumes:
      - name: nginx-logs
        hostPath:
          path: /var/log/nginx
          type: Directory
```

## 🔒 Security Considerations

1. **Use HTTPS/WSS**: Always use secure WebSocket connections in production.
2. **Authentication**: Implement authentication if exposing the WebSocket endpoint to the internet.
3. **Rate Limiting**: Consider implementing rate limiting to prevent abuse.
4. **Log Rotation**: Ensure proper log rotation is configured for Nginx.

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built with [Node.js](https://nodejs.org/) and [ws](https://github.com/websockets/ws)
- Inspired by the need for real-time log monitoring
- Special thanks to all contributors

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📧 Contact

Aldo Tobing - [@aldo_tobing](https://twitter.com/aldo_tobing) 

Project Link: [https://github.com/aldotobing/nginx-log-streamer](https://github.com/aldotobing/nginx-log-streamer)
