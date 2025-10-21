# Nginx Load Balancer Configuration

This repository contains an nginx configuration for load balancing across backend servers.

## 📋 Overview

This nginx setup provides:
- **Load balancing** across 2 backend servers using round-robin algorithm
- **Performance optimization** with 8 worker processes and 4096 connections per worker
- **Security** by blocking direct IP access on port 80

## 🏗️ Architecture

```
Client Requests
      ↓
   Nginx Load Balancer (Port 3000)
      ↓
Round-Robin Distribution
      ↓
┌─────────────────────┬─────────────────────┐
│ Backend Server 1    │ Backend Server 2    │
│ 54.255.182.118:5000│ 54.255.182.118:5001│
└─────────────────────┴─────────────────────┘
```

## ⚙️ Configuration Details

### Load Balancer Server
- **Listen Port**: 3000
- **Load Balancing Method**: Round-robin (default)
- **Backend Servers**:
  - `54.255.182.118:5000`
  - `54.255.182.118:5001`

### Security Features
- **IP Access Block**: Direct access on port 80 returns 404
- **Proxy Headers**: Forwards Host, X-Real-IP, X-Forwarded-For, X-Forwarded-Proto

### HTTP Settings
- **MIME Types**: Included from mime.types file
- **Default Type**: application/octet-stream

## 🚀 Quick Start

### Prerequisites
- Nginx 1.24.0 or later
- 2 backend applications running on 54.255.182.118:5000 and 54.255.182.118:5001

### 1. Ensure Backend Services Are Running
Verify your applications are accessible at:
```bash
http://54.255.182.118:5000
http://54.255.182.118:5001
```

### 2. Test Nginx Configuration
```bash
nginx -t -c conf/nginx.conf
```

### 3. Start Nginx
```bash
nginx -c conf/nginx.conf
```

### 4. Test Load Balancer
```bash
curl http://localhost:3000
```

The requests will be distributed between your two backend servers in round-robin fashion.

## 🔄 How Round-Robin Load Balancing Works

1. **Request 1** → Backend Server 1 (54.255.182.118:5000)
2. **Request 2** → Backend Server 2 (54.255.182.118:5001)
3. **Request 3** → Backend Server 1 (54.255.182.118:5000) ← Cycle repeats

This ensures even distribution of traffic across both available servers.

## 📊 Monitoring

The current configuration uses nginx's default behavior:
- Requests are distributed using round-robin algorithm
- Failed requests are automatically retried on the other server
- No explicit logging is configured (commented out in config)

## 🛠️ Configuration Files

- `conf/nginx.conf` - Main nginx configuration
- `conf/mime.types` - MIME type definitions
- `html/` - Default error pages
- `temp/` - Temporary file storage

##  Testing the Setup

### 1. Basic Functionality Test
```bash
# Test if load balancer is responding
curl -I http://localhost:3000
```

### 2. Round-Robin Verification
Make multiple requests and observe distribution:
```bash
for i in {1..4}; do
  curl http://localhost:3000
  echo "Request $i completed"
done
```

### 3. Backend Server Testing
Test backend servers directly:
```bash
curl http://54.255.182.118:5000
curl http://54.255.182.118:5001
```

## 📝 Troubleshooting

### Common Issues

1. **"Connection refused"**
   - Ensure backend servers at 54.255.182.118:5000 and 54.255.182.118:5001 are running
   - Check network connectivity to 54.255.182.118

2. **"502 Bad Gateway"**
   - Backend servers are not responding
   - Verify backend application health

3. **Port 80 returns 404**
   - This is expected behavior - direct IP access is blocked

## 🔒 Security Notes

- The configuration intentionally blocks direct IP access on port 80
- All requests should go through the load balancer on port 3000
- Proxy headers are set to forward client information to backend servers