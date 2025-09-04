# Nginx Reverse Proxy for API Services

This reverse proxy setup routes requests to your 4 API endpoints:

- `/auth/` - Authentication service
- `/assets/` - Assets management service  
- `/collections/` - Collections service
- `/query` - GraphQL query endpoint

## Quick Start

1. **Configure your services**:
   - Update the upstream server addresses in `conf.d/api.conf`
   - Modify the service configurations in `docker-compose.yml`

2. **Build and run**:
   ```bash
   docker-compose up --build
   ```

3. **Test the endpoints**:
   ```bash
   # Health check
   curl http://localhost/health
   
   # Test endpoints (adjust based on your API)
   curl http://localhost/auth/status
   curl http://localhost/assets/
   curl http://localhost/collections/
   curl -X POST http://localhost/query -H "Content-Type: application/json" -d '{"query":"{ __schema { types { name } } }"}'
   ```

## Configuration

### Service Endpoints

Update the upstream servers in `conf.d/api.conf`:

```nginx
upstream auth_service {
    server your-auth-host:port;
}
```

### Environment Variables

Copy `.env.example` to `.env` and customize:

```bash
cp .env.example .env
```

### Rate Limiting

The proxy includes rate limiting:
- Auth endpoints: 5 requests/second
- Other API endpoints: 10 requests/second

### CORS Support

CORS headers are configured for cross-origin requests. Adjust in `conf.d/api.conf` if needed.

### File Uploads

Assets endpoint supports file uploads up to 100MB. Adjust `client_max_body_size` in the configuration.

## SSL/HTTPS Support

To add SSL support:

1. Add your SSL certificates to a `ssl/` directory
2. Update the nginx configuration to include SSL settings
3. Expose port 443 in the Containerfile

Example SSL configuration:
```nginx
server {
    listen 443 ssl;
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    # ... rest of configuration
}
```

## Monitoring

Nginx logs are available in the `logs/` directory:
- Access logs: `logs/access.log`
- Error logs: `logs/error.log`

## Customization

### GitHub Actions CI/CD

The repository includes several GitHub Actions workflows:

#### 1. Build and Publish to GitHub Container Registry
**File**: `.github/workflows/build-and-publish.yml`

Automatically builds and pushes the nginx reverse proxy image to GitHub Container Registry (ghcr.io) when:
- Code is pushed to `main` or `develop` branches
- Tags starting with `v` are created
- Pull requests are opened

**Setup**: No additional configuration needed - uses `GITHUB_TOKEN` automatically.

#### 2. Build and Publish to Docker Hub
**File**: `.github/workflows/build-and-publish-dockerhub.yml`

Alternative workflow for Docker Hub publishing.

**Setup**:
1. Update `DOCKERHUB_USERNAME` in the workflow file
2. Add `DOCKERHUB_TOKEN` secret in repository settings
3. Disable the GitHub Container Registry workflow if not needed

#### 3. Multi-Registry Publishing
**File**: `.github/workflows/build-and-publish-multi.yml`

Publishes to multiple registries simultaneously and includes security scanning.

**Setup**:
1. Uncomment and configure Docker Hub section if needed
2. Add required secrets for additional registries

#### 4. Configuration Testing
**File**: `.github/workflows/test-nginx.yml`

Tests nginx configuration syntax and container startup on every push/PR.

### Container Registry Options

The image will be available at:
- **GitHub Container Registry**: `ghcr.io/your-username/your-repo/nginx-reverse-proxy:latest`
- **Docker Hub**: `your-username/nginx-reverse-proxy:latest`

### Usage in Production

Pull and run the published image:

```bash
# From GitHub Container Registry
docker pull ghcr.io/your-username/your-repo/nginx-reverse-proxy:latest
docker run -p 80:80 ghcr.io/your-username/your-repo/nginx-reverse-proxy:latest

# From Docker Hub  
docker pull your-username/nginx-reverse-proxy:latest
docker run -p 80:80 your-username/nginx-reverse-proxy:latest
```

### Adding New Endpoints

1. Add upstream definition in `conf.d/api.conf`
2. Add location block with proxy configuration
3. Update docker-compose.yml if needed

### Security Headers

Security headers are configured by default. Customize in `conf.d/api.conf`:
- X-Frame-Options
- X-XSS-Protection  
- Content-Security-Policy
- etc.

## Troubleshooting

### Service Connection Issues

If services can't be reached:
1. Check service names in docker-compose.yml match upstream definitions
2. Verify services are running: `docker-compose ps`
3. Check nginx logs: `docker-compose logs nginx-proxy`

### CORS Issues

If you encounter CORS errors:
1. Verify CORS headers in nginx configuration
2. Check that OPTIONS requests are handled correctly
3. Ensure your frontend domain is allowed

### Performance Tuning

For high traffic:
1. Adjust `worker_connections` in nginx.conf
2. Tune rate limiting zones
3. Configure upstream load balancing if using multiple instances
