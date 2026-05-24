# Deployment Guide - Skillnox Contest Platform

## 1. Document Information

| Field | Value |
|-------|-------|
| **Document Title** | Deployment Guide - Skillnox Contest Platform |
| **Version** | 1.0 |
| **Date** | December 2024 |
| **Author** | DevOps Team |
| **Status** | Approved |

## 2. Overview

This guide provides comprehensive instructions for deploying the Skillnox contest platform in various environments, from local development to production. The platform consists of a React frontend, Node.js backend, PostgreSQL database, and Redis cache.

## 3. System Requirements

### 3.1 Minimum Hardware Requirements

#### Development Environment
- **CPU**: 2 cores, 2.0 GHz
- **RAM**: 4GB
- **Storage**: 20GB available space
- **Network**: Broadband internet connection

#### Production Environment
- **CPU**: 4 cores, 2.5 GHz
- **RAM**: 8GB (16GB recommended)
- **Storage**: 100GB SSD
- **Network**: High-speed internet with low latency

### 3.2 Software Requirements

#### Required Software
- **Node.js**: Version 18.0 or higher
- **npm**: Version 8.0 or higher
- **PostgreSQL**: Version 12 or higher
- **Redis**: Version 6.0 or higher
- **Git**: Version 2.0 or higher

#### Optional Software
- **Docker**: Version 20.0 or higher
- **Docker Compose**: Version 2.0 or higher
- **Nginx**: Version 1.18 or higher (for production)
- **PM2**: Version 5.0 or higher (for process management)

## 4. Environment Setup

### 4.1 Local Development Environment

#### Step 1: Clone Repository
```bash
git clone <repository-url>
cd Skillnox
```

#### Step 2: Install Dependencies
```bash
npm install
```

#### Step 3: Environment Configuration
Create a `.env` file in the root directory:
```env
# Database Configuration
DATABASE_URL=postgresql://username:password@localhost:5432/skillnox_dev

# Session Configuration
SESSION_SECRET=your-development-session-secret

# Environment
NODE_ENV=development

# Redis Configuration (Optional for development)
REDIS_URL=redis://localhost:6379

# Server Configuration
PORT=5000
CLIENT_URL=http://localhost:3000
```

#### Step 4: Database Setup
```bash
# Start PostgreSQL service
sudo service postgresql start

# Create database
createdb skillnox_dev

# Run database migrations
npm run db:push
```

#### Step 5: Start Development Server
```bash
npm run dev
```

The application will be available at:
- Frontend: http://localhost:3000
- Backend API: http://localhost:5000

### 4.2 Staging Environment

#### Step 1: Server Preparation
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib -y

# Install Redis
sudo apt install redis-server -y

# Install Nginx
sudo apt install nginx -y
```

#### Step 2: Database Configuration
```bash
# Switch to postgres user
sudo -u postgres psql

# Create database and user
CREATE DATABASE skillnox_staging;
CREATE USER skillnox_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE skillnox_staging TO skillnox_user;
\q
```

#### Step 3: Application Deployment
```bash
# Clone repository
git clone <repository-url> /var/www/skillnox
cd /var/www/skillnox

# Install dependencies
npm install

# Build application
npm run build

# Set environment variables
sudo nano /var/www/skillnox/.env
```

Staging environment configuration:
```env
DATABASE_URL=postgresql://skillnox_user:secure_password@localhost:5432/skillnox_staging
SESSION_SECRET=your-staging-session-secret
NODE_ENV=staging
REDIS_URL=redis://localhost:6379
PORT=5000
CLIENT_URL=https://staging.skillnox.com
```

#### Step 4: Process Management
```bash
# Install PM2
npm install -g pm2

# Start application
pm2 start dist/index.js --name skillnox-staging

# Save PM2 configuration
pm2 save
pm2 startup
```

#### Step 5: Nginx Configuration
Create `/etc/nginx/sites-available/skillnox-staging`:
```nginx
server {
    listen 80;
    server_name staging.skillnox.com;

    # Frontend
    location / {
        root /var/www/skillnox/dist/public;
        try_files $uri $uri/ /index.html;
    }

    # API
    location /api {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # WebSocket
    location /socket.io/ {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/skillnox-staging /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 4.3 Production Environment

#### Step 1: Server Setup
```bash
# Create application user
sudo adduser skillnox
sudo usermod -aG sudo skillnox

# Switch to application user
su - skillnox
```

#### Step 2: SSL Certificate Setup
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtain SSL certificate
sudo certbot --nginx -d skillnox.com -d www.skillnox.com
```

#### Step 3: Database Configuration
```bash
# Configure PostgreSQL for production
sudo nano /etc/postgresql/12/main/postgresql.conf

# Set memory and connection limits
shared_buffers = 256MB
max_connections = 200
effective_cache_size = 1GB

# Restart PostgreSQL
sudo systemctl restart postgresql
```

#### Step 4: Redis Configuration
```bash
# Configure Redis for production
sudo nano /etc/redis/redis.conf

# Set memory limit and persistence
maxmemory 512mb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000

# Restart Redis
sudo systemctl restart redis-server
```

#### Step 5: Application Deployment
```bash
# Clone repository
git clone <repository-url> /home/skillnox/app
cd /home/skillnox/app

# Install dependencies
npm install --production

# Build application
npm run build

# Set production environment
sudo nano /home/skillnox/app/.env
```

Production environment configuration:
```env
DATABASE_URL=postgresql://skillnox_user:secure_production_password@localhost:5432/skillnox_production
SESSION_SECRET=your-production-session-secret
NODE_ENV=production
REDIS_URL=redis://localhost:6379
PORT=5000
CLIENT_URL=https://skillnox.com
```

#### Step 6: Process Management with PM2
```bash
# Install PM2
npm install -g pm2

# Create PM2 ecosystem file
nano ecosystem.config.js
```

PM2 ecosystem configuration:
```javascript
module.exports = {
  apps: [{
    name: 'skillnox-production',
    script: 'dist/index.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 5000
    },
    error_file: '/var/log/skillnox/error.log',
    out_file: '/var/log/skillnox/out.log',
    log_file: '/var/log/skillnox/combined.log',
    time: true
  }]
};
```

Start the application:
```bash
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

#### Step 7: Nginx Production Configuration
Create `/etc/nginx/sites-available/skillnox-production`:
```nginx
server {
    listen 80;
    server_name skillnox.com www.skillnox.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name skillnox.com www.skillnox.com;

    ssl_certificate /etc/letsencrypt/live/skillnox.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/skillnox.com/privkey.pem;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

    # Frontend
    location / {
        root /home/skillnox/app/dist/public;
        try_files $uri $uri/ /index.html;
        
        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # API
    location /api {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Rate limiting
        limit_req zone=api burst=20 nodelay;
    }

    # WebSocket
    location /socket.io/ {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Rate limiting configuration
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
}
```

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/skillnox-production /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## 5. Docker Deployment

### 5.1 Docker Compose Setup

Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
      - "5000:5000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://skillnox:password@db:5432/skillnox
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./uploads:/app/uploads

  db:
    image: postgres:12
    environment:
      - POSTGRES_DB=skillnox
      - POSTGRES_USER=skillnox
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl
    depends_on:
      - app

volumes:
  postgres_data:
  redis_data:
```

### 5.2 Dockerfile

Create `Dockerfile`:
```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Install production dependencies
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy built application
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/client/dist ./client/dist

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S skillnox -u 1001

# Change ownership
RUN chown -R skillnox:nodejs /app
USER skillnox

EXPOSE 3000 5000

CMD ["node", "dist/index.js"]
```

### 5.3 Docker Deployment Commands

```bash
# Build and start services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Update application
docker-compose pull
docker-compose up -d
```

## 6. Database Management

### 6.1 Database Migrations

```bash
# Run migrations
npm run db:push

# Generate migration
npx drizzle-kit generate

# Apply specific migration
npx drizzle-kit push
```

### 6.2 Database Backups

#### Automated Backup Script
Create `backup.sh`:
```bash
#!/bin/bash

# Configuration
DB_NAME="skillnox_production"
DB_USER="skillnox_user"
BACKUP_DIR="/var/backups/skillnox"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Create backup
pg_dump -h localhost -U $DB_USER -d $DB_NAME > $BACKUP_DIR/skillnox_$DATE.sql

# Compress backup
gzip $BACKUP_DIR/skillnox_$DATE.sql

# Remove backups older than 30 days
find $BACKUP_DIR -name "skillnox_*.sql.gz" -mtime +30 -delete

echo "Backup completed: skillnox_$DATE.sql.gz"
```

#### Cron Job for Automated Backups
```bash
# Add to crontab
0 2 * * * /path/to/backup.sh
```

### 6.3 Database Restoration

```bash
# Restore from backup
gunzip -c skillnox_20241201_020000.sql.gz | psql -h localhost -U skillnox_user -d skillnox_production
```

## 7. Monitoring and Logging

### 7.1 Application Monitoring

#### PM2 Monitoring
```bash
# Monitor application
pm2 monit

# View logs
pm2 logs skillnox-production

# Restart application
pm2 restart skillnox-production
```

#### System Monitoring Script
Create `monitor.sh`:
```bash
#!/bin/bash

# Check application status
if ! pm2 list | grep -q "skillnox-production.*online"; then
    echo "Application is down, restarting..."
    pm2 restart skillnox-production
fi

# Check database connection
if ! pg_isready -h localhost -p 5432; then
    echo "Database is down!"
    # Send alert notification
fi

# Check Redis connection
if ! redis-cli ping | grep -q "PONG"; then
    echo "Redis is down!"
    # Send alert notification
fi
```

### 7.2 Log Management

#### Log Rotation Configuration
Create `/etc/logrotate.d/skillnox`:
```
/var/log/skillnox/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 644 skillnox skillnox
    postrotate
        pm2 reloadLogs
    endscript
}
```

## 8. Security Configuration

### 8.1 Firewall Setup

```bash
# Configure UFW firewall
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
sudo ufw deny 5000  # Block direct access to app port
```

### 8.2 SSL/TLS Configuration

#### Let's Encrypt Setup
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d skillnox.com -d www.skillnox.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

### 8.3 Security Headers

Add to Nginx configuration:
```nginx
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';";
```

## 9. Performance Optimization

### 9.1 Database Optimization

#### PostgreSQL Configuration
```bash
# Edit postgresql.conf
sudo nano /etc/postgresql/12/main/postgresql.conf

# Optimize for production
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
```

### 9.2 Redis Optimization

```bash
# Edit redis.conf
sudo nano /etc/redis/redis.conf

# Memory optimization
maxmemory 512mb
maxmemory-policy allkeys-lru

# Persistence optimization
save 900 1
save 300 10
save 60 10000
```

### 9.3 Nginx Optimization

```nginx
# Worker processes
worker_processes auto;

# Connection limits
events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

# HTTP optimization
http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
}
```

## 10. Troubleshooting

### 10.1 Common Issues

#### Application Won't Start
```bash
# Check logs
pm2 logs skillnox-production

# Check port availability
netstat -tulpn | grep :5000

# Check environment variables
pm2 show skillnox-production
```

#### Database Connection Issues
```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Check database connectivity
psql -h localhost -U skillnox_user -d skillnox_production

# Check connection limits
psql -c "SELECT count(*) FROM pg_stat_activity;"
```

#### Redis Connection Issues
```bash
# Check Redis status
sudo systemctl status redis-server

# Test Redis connection
redis-cli ping

# Check Redis logs
sudo journalctl -u redis-server
```

### 10.2 Performance Issues

#### High CPU Usage
```bash
# Check process usage
top -p $(pgrep -f "node.*skillnox")

# Check PM2 metrics
pm2 monit
```

#### High Memory Usage
```bash
# Check memory usage
free -h

# Check PM2 memory usage
pm2 list
```

#### Database Performance
```bash
# Check slow queries
sudo -u postgres psql -c "SELECT query, mean_time, calls FROM pg_stat_statements ORDER BY mean_time DESC LIMIT 10;"
```

## 11. Maintenance Procedures

### 11.1 Regular Maintenance Tasks

#### Daily Tasks
- Monitor application logs
- Check system resources
- Verify backup completion

#### Weekly Tasks
- Review performance metrics
- Check security updates
- Clean up old logs

#### Monthly Tasks
- Update system packages
- Review and rotate logs
- Performance analysis
- Security audit

### 11.2 Update Procedures

#### Application Updates
```bash
# Pull latest changes
git pull origin main

# Install dependencies
npm install

# Run database migrations
npm run db:push

# Build application
npm run build

# Restart application
pm2 restart skillnox-production
```

#### System Updates
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Restart services
sudo systemctl restart postgresql
sudo systemctl restart redis-server
sudo systemctl restart nginx
pm2 restart skillnox-production
```

## 12. Disaster Recovery

### 12.1 Backup Strategy

#### Database Backups
- Daily automated backups
- Weekly full backups
- Monthly archive backups
- Cross-region backup replication

#### Application Backups
- Source code version control
- Configuration file backups
- SSL certificate backups
- Environment variable backups

### 12.2 Recovery Procedures

#### Database Recovery
```bash
# Stop application
pm2 stop skillnox-production

# Restore database
gunzip -c backup_file.sql.gz | psql -h localhost -U skillnox_user -d skillnox_production

# Start application
pm2 start skillnox-production
```

#### Full System Recovery
1. Provision new server
2. Install required software
3. Restore database from backup
4. Deploy application code
5. Configure services
6. Update DNS records
7. Verify functionality

---

**Document Version**: 1.0  
**Last Updated**: December 2024  
**Author**: DevOps Team  
**Review Status**: Approved
