# Dockerization

This document provides containerization strategy for the VB6 to .NET Core migrated application, including Dockerfiles, docker-compose configurations, and Kubernetes manifests for production deployment.

## Containerization Strategy

### Application Architecture
```
Containerized Application
├── Web API Container (ASP.NET Core)
├── Database Container (SQL Server/PostgreSQL)
├── Background Worker Container (Optional)
├── Reverse Proxy Container (nginx)
└── Monitoring Container (Prometheus/Grafana)
```

### Multi-Stage Dockerfile for Web API
```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src

# Copy csproj files and restore dependencies
COPY ["VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj", "VB6ToDotNetCore.Web/"]
COPY ["VB6ToDotNetCore.Core/VB6ToDotNetCore.Core.csproj", "VB6ToDotNetCore.Core/"]
COPY ["VB6ToDotNetCore.Data/VB6ToDotNetCore.Data.csproj", "VB6ToDotNetCore.Data/"]
RUN dotnet restore "VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj"

# Copy everything else and build
COPY . .
WORKDIR "/src/VB6ToDotNetCore.Web"
RUN dotnet build "VB6ToDotNetCore.Web.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "VB6ToDotNetCore.Web.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS final
WORKDIR /app

# Create non-root user for security
RUN adduser --disabled-password --gecos '' appuser && chown -R appuser:appuser /app
USER appuser

# Copy published application
COPY --from=publish /app/publish .

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

# Expose port
EXPOSE 80

# Set entry point
ENTRYPOINT ["dotnet", "VB6ToDotNetCore.Web.dll"]
```

### WPF/WinForms Application Containerization
```dockerfile
# For Windows-based UI applications (if needed)
FROM mcr.microsoft.com/dotnet/framework/sdk:4.8 AS build
WORKDIR /app

# Copy and restore
COPY VB6ToDotNetCore.UI/VB6ToDotNetCore.UI.csproj .
RUN nuget restore VB6ToDotNetCore.UI.csproj

# Copy source and build
COPY . .
RUN msbuild VB6ToDotNetCore.UI.csproj /p:Configuration=Release /p:OutputPath=/app/publish

# Runtime image
FROM mcr.microsoft.com/dotnet/framework/runtime:4.8 AS final
WORKDIR /app
COPY --from=build /app/publish .

# For Windows containers, use different health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD powershell -command "try { $response = Invoke-WebRequest -Uri http://localhost:8080/health -UseBasicParsing; if ($response.StatusCode -eq 200) { exit 0 } else { exit 1 } } catch { exit 1 }"

EXPOSE 8080
ENTRYPOINT ["VB6ToDotNetCore.UI.exe"]
```

### Background Worker Dockerfile
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src

COPY ["VB6ToDotNetCore.Worker/VB6ToDotNetCore.Worker.csproj", "VB6ToDotNetCore.Worker/"]
RUN dotnet restore "VB6ToDotNetCore.Worker/VB6ToDotNetCore.Worker.csproj"

COPY . .
WORKDIR "/src/VB6ToDotNetCore.Worker"
RUN dotnet build "VB6ToDotNetCore.Worker.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "VB6ToDotNetCore.Worker.csproj" -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/runtime:6.0 AS final
WORKDIR /app
COPY --from=publish /app/publish .

# Create non-root user
RUN adduser --disabled-password --gecos '' workeruser && chown -R workeruser:workeruser /app
USER workeruser

ENTRYPOINT ["dotnet", "VB6ToDotNetCore.Worker.dll"]
```

## Docker Compose Configuration

### Development Environment
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Database
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      ACCEPT_EULA: Y
      SA_PASSWORD: YourStrong!Passw0rd
      MSSQL_PID: Express
    ports:
      - "1433:1433"
    volumes:
      - sqlserver_data:/var/opt/mssql
      - ./init-db:/docker-entrypoint-initdb.d
    networks:
      - vb6network
    restart: unless-stopped

  # Redis for caching (if needed)
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - vb6network
    restart: unless-stopped

  # Web API
  api:
    build:
      context: .
      dockerfile: src/VB6ToDotNetCore.Web/Dockerfile
    ports:
      - "8080:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=db;Database=VB6ToDotNetCore;User Id=sa;Password=YourStrong!Passw0rd;
      - ConnectionStrings__Redis=redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    networks:
      - vb6network
    restart: unless-stopped

  # Background worker
  worker:
    build:
      context: .
      dockerfile: src/VB6ToDotNetCore.Worker/Dockerfile
    environment:
      - DOTNET_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=db;Database=VB6ToDotNetCore;User Id=sa;Password=YourStrong!Passw0rd;
    depends_on:
      - db
    networks:
      - vb6network
    restart: unless-stopped

  # Reverse proxy
  proxy:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/ssl/certs
    depends_on:
      - api
    networks:
      - vb6network
    restart: unless-stopped

volumes:
  sqlserver_data:
  redis_data:

networks:
  vb6network:
    driver: bridge
```

### Production Environment
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      ACCEPT_EULA: Y
      SA_PASSWORD: ${DB_PASSWORD}
      MSSQL_PID: Express
    volumes:
      - sqlserver_data:/var/opt/mssql
      - ./backups:/var/opt/mssql/backups
    networks:
      - vb6network
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '1.0'
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - vb6network
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
    restart: unless-stopped

  api:
    image: vb6-dotnet-core-api:${TAG:-latest}
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://+:80
      - ConnectionStrings__DefaultConnection=Server=db;Database=VB6ToDotNetCore;User Id=sa;Password=${DB_PASSWORD};
      - ConnectionStrings__Redis=redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    networks:
      - vb6network
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  worker:
    image: vb6-dotnet-core-worker:${TAG:-latest}
    environment:
      - DOTNET_ENVIRONMENT=Production
      - ConnectionStrings__DefaultConnection=Server=db;Database=VB6ToDotNetCore;User Id=sa;Password=${DB_PASSWORD};
    depends_on:
      - db
    networks:
      - vb6network
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 256M
          cpus: '0.25'
    restart: unless-stopped

  proxy:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.prod.conf:/etc/nginx/nginx.conf:ro
      - ssl_certs:/etc/ssl/certs
    depends_on:
      - api
    networks:
      - vb6network
    deploy:
      resources:
        limits:
          memory: 128M
          cpus: '0.25'
    restart: unless-stopped

volumes:
  sqlserver_data:
    driver: local
  redis_data:
    driver: local
  ssl_certs:
    driver: local

networks:
  vb6network:
    driver: overlay
```

## Kubernetes Manifests

### Namespace
```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: vb6-dotnet-core
  labels:
    name: vb6-dotnet-core
```

### ConfigMap for Configuration
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vb6-config
  namespace: vb6-dotnet-core
data:
  ASPNETCORE_ENVIRONMENT: "Production"
  ASPNETCORE_URLS: "http://+:80"
```

### Secret for Sensitive Data
```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: vb6-secrets
  namespace: vb6-dotnet-core
type: Opaque
data:
  db-password: <base64-encoded-password>
  jwt-key: <base64-encoded-jwt-key>
  api-key: <base64-encoded-api-key>
```

### Database Deployment
```yaml
# db-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vb6-db
  namespace: vb6-dotnet-core
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vb6-db
  template:
    metadata:
      labels:
        app: vb6-db
    spec:
      containers:
      - name: sqlserver
        image: mcr.microsoft.com/mssql/server:2022-latest
        ports:
        - containerPort: 1433
        env:
        - name: ACCEPT_EULA
          value: "Y"
        - name: SA_PASSWORD
          valueFrom:
            secretKeyRef:
              name: vb6-secrets
              key: db-password
        - name: MSSQL_PID
          value: "Express"
        volumeMounts:
        - name: mssql-data
          mountPath: /var/opt/mssql
        resources:
          limits:
            memory: "2Gi"
            cpu: "1000m"
          requests:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          exec:
            command:
            - /opt/mssql-tools/bin/sqlcmd
            - -S
            - localhost
            - -U
            - sa
            - -P
            - $(SA_PASSWORD)
            - -Q
            - "SELECT 1"
          initialDelaySeconds: 30
          periodSeconds: 10
      volumes:
      - name: mssql-data
        persistentVolumeClaim:
          claimName: mssql-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: vb6-db-service
  namespace: vb6-dotnet-core
spec:
  selector:
    app: vb6-db
  ports:
  - port: 1433
    targetPort: 1433
  type: ClusterIP

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mssql-pvc
  namespace: vb6-dotnet-core
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### API Deployment
```yaml
# api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vb6-api
  namespace: vb6-dotnet-core
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vb6-api
  template:
    metadata:
      labels:
        app: vb6-api
    spec:
      containers:
      - name: api
        image: vb6-dotnet-core-api:latest
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          valueFrom:
            configMapKeyRef:
              name: vb6-config
              key: ASPNETCORE_ENVIRONMENT
        - name: ASPNETCORE_URLS
          valueFrom:
            configMapKeyRef:
              name: vb6-config
              key: ASPNETCORE_URLS
        - name: ConnectionStrings__DefaultConnection
          value: "Server=vb6-db-service;Database=VB6ToDotNetCore;User Id=sa;Password=$(DB_PASSWORD)"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: vb6-secrets
              key: db-password
        envFrom:
        - configMapRef:
            name: vb6-config
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
          requests:
            memory: "256Mi"
            cpu: "250m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: vb6-api-service
  namespace: vb6-dotnet-core
spec:
  selector:
    app: vb6-api
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vb6-api-ingress
  namespace: vb6-dotnet-core
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.yourdomain.com
    secretName: vb6-api-tls
  rules:
  - host: api.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vb6-api-service
            port:
              number: 80
```

### Worker Deployment
```yaml
# worker-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vb6-worker
  namespace: vb6-dotnet-core
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vb6-worker
  template:
    metadata:
      labels:
        app: vb6-worker
    spec:
      containers:
      - name: worker
        image: vb6-dotnet-core-worker:latest
        env:
        - name: DOTNET_ENVIRONMENT
          value: "Production"
        - name: ConnectionStrings__DefaultConnection
          value: "Server=vb6-db-service;Database=VB6ToDotNetCore;User Id=sa;Password=$(DB_PASSWORD)"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: vb6-secrets
              key: db-password
        resources:
          limits:
            memory: "256Mi"
            cpu: "250m"
          requests:
            memory: "128Mi"
            cpu: "125m"
```

### Horizontal Pod Autoscaler
```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: vb6-api-hpa
  namespace: vb6-dotnet-core
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vb6-api
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### NGINX Configuration
```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;

    # Performance
    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 100M;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/xml+rss
        application/json;

    upstream api_backend {
        server vb6-api:80;
    }

    server {
        listen 80;
        server_name localhost;

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Referrer-Policy "no-referrer-when-downgrade" always;
        add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

        # API routing
        location /api/ {
            proxy_pass http://api_backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection keep-alive;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
            proxy_read_timeout 86400;
        }

        # Health check
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }

        # Static files (if serving SPA)
        location / {
            try_files $uri $uri/ /index.html;
            root /usr/share/nginx/html;
            index index.html index.htm;
        }
    }
}
```

## Monitoring and Logging

### Prometheus Configuration
```yaml
# prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: vb6-dotnet-core
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s

    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"

    scrape_configs:
      - job_name: 'vb6-api'
        static_configs:
          - targets: ['vb6-api-service:80']
        metrics_path: '/metrics'
        scrape_interval: 5s

      - job_name: 'vb6-db'
        static_configs:
          - targets: ['vb6-db-service:1433']
        scrape_interval: 30s
```

### Grafana Dashboard
```yaml
# grafana-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: vb6-dotnet-core
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: vb6-secrets
              key: grafana-admin-password
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
        - name: grafana-config
          mountPath: /etc/grafana/provisioning
      volumes:
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: grafana-pvc
      - name: grafana-config
        configMap:
          name: grafana-config

---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
  namespace: vb6-dotnet-core
spec:
  selector:
    app: grafana
  ports:
  - port: 3000
    targetPort: 3000
  type: ClusterIP
```

## Deployment Scripts

### Build and Push Script
```bash
#!/bin/bash
# build-and-push.sh

set -e

# Configuration
REGISTRY="your-registry.com"
IMAGE_NAME="vb6-dotnet-core"
TAG=${1:-"latest"}

echo "Building and pushing Docker images..."

# Build API image
echo "Building API image..."
docker build -f src/VB6ToDotNetCore.Web/Dockerfile -t $REGISTRY/$IMAGE_NAME-api:$TAG .

# Build Worker image
echo "Building Worker image..."
docker build -f src/VB6ToDotNetCore.Worker/Dockerfile -t $REGISTRY/$IMAGE_NAME-worker:$TAG .

# Push images
echo "Pushing images to registry..."
docker push $REGISTRY/$IMAGE_NAME-api:$TAG
docker push $REGISTRY/$IMAGE_NAME-worker:$TAG

echo "Build and push completed successfully!"
```

### Kubernetes Deployment Script
```bash
#!/bin/bash
# deploy-k8s.sh

set -e

NAMESPACE="vb6-dotnet-core"
TAG=${1:-"latest"}

echo "Deploying to Kubernetes..."

# Apply configurations
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/db-deployment.yaml
kubectl apply -f k8s/api-deployment.yaml
kubectl apply -f k8s/worker-deployment.yaml
kubectl apply -f k8s/hpa.yaml

# Update images
kubectl set image deployment/vb6-api api=$REGISTRY/vb6-dotnet-core-api:$TAG -n $NAMESPACE
kubectl set image deployment/vb6-worker worker=$REGISTRY/vb6-dotnet-core-worker:$TAG -n $NAMESPACE

# Wait for rollout
kubectl rollout status deployment/vb6-api -n $NAMESPACE
kubectl rollout status deployment/vb6-worker -n $NAMESPACE

echo "Deployment completed successfully!"
```

## Container Orchestration Best Practices

### Health Checks
- Implement proper health check endpoints
- Use startup, liveness, and readiness probes
- Monitor application-specific health metrics

### Resource Management
- Set appropriate CPU and memory limits
- Use resource requests for scheduling
- Monitor resource usage with metrics

### Security
- Run containers as non-root users
- Use read-only root filesystems where possible
- Implement network policies
- Scan images for vulnerabilities

### Scaling
- Implement horizontal pod autoscaling
- Use cluster autoscaling for cloud providers
- Monitor application performance metrics

### Backup and Recovery
- Regular database backups
- Persistent volume snapshots
- Disaster recovery procedures

This containerization strategy provides a complete path from development to production deployment, ensuring scalability, security, and maintainability of the migrated VB6 to .NET Core application.