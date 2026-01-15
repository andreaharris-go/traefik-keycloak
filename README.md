# Traefik-Keycloak Authentication for Qdrant UI

This repository demonstrates how to force users to log in with Keycloak before accessing Qdrant UI using Traefik as a reverse proxy with ForwardAuth middleware.

## Features

- 🔐 **Keycloak Authentication**: Users must authenticate via Keycloak before accessing Qdrant UI
- 🚦 **Traefik Reverse Proxy**: All traffic is routed through Traefik with ForwardAuth middleware
- 🎯 **Qdrant UI Access**: Protected Qdrant UI accessible only after successful authentication
- 🏠 **Local Development**: Uses *.localhost domains for easy local testing without /etc/hosts modifications

## Architecture

```
User Request → Traefik → ForwardAuth Middleware → Keycloak (if not authenticated)
                    ↓
            Qdrant UI (if authenticated)
```

## Services

The stack includes the following services:

| Service | URL | Description |
|---------|-----|-------------|
| Traefik Dashboard | http://traefik.localhost:8080 | Traefik web UI and API |
| Keycloak | http://keycloak.localhost | Identity and access management |
| Qdrant UI | http://qdrant.localhost | Protected Qdrant vector database UI |
| Auth Service | http://auth.localhost | Forward authentication service |

## Prerequisites

- Docker and Docker Compose installed
- Ports 80 and 8080 available on your machine

## Quick Start

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd traefik-keycloak
   ```

2. **Start the stack**
   ```bash
   docker-compose up -d
   ```

3. **Wait for services to be ready** (about 30-60 seconds)
   ```bash
   docker-compose logs -f
   ```

4. **Access Qdrant UI**
   - Open your browser and navigate to: http://qdrant.localhost
   - You will be automatically redirected to Keycloak login page
   - Use the test credentials:
     - **Username**: `testuser`
     - **Password**: `testpassword`
   - After successful login, you'll be redirected back to Qdrant UI

## How It Works

1. **User attempts to access Qdrant UI** at http://qdrant.localhost
2. **Traefik intercepts the request** and applies the ForwardAuth middleware
3. **ForwardAuth checks for authentication** by validating the session cookie
4. **If not authenticated**, user is redirected to Keycloak login page
5. **User logs in** with Keycloak credentials
6. **Keycloak redirects back** to ForwardAuth with authentication token
7. **ForwardAuth validates token** and sets a session cookie
8. **User is redirected** to the original Qdrant UI URL
9. **Subsequent requests** are automatically authenticated via the session cookie

## Configuration

### Keycloak Configuration

The Keycloak realm is automatically imported from `keycloak/realm-export.json` with:
- **Realm**: qdrant
- **Client ID**: qdrant-client
- **Client Secret**: qdrant-client-secret
- **Test User**: testuser / testpassword

### Keycloak Admin Access

- URL: http://keycloak.localhost
- Username: `admin`
- Password: `admin`

### Traefik Configuration

Traefik is configured with:
- Docker provider for automatic service discovery
- HTTP entrypoint on port 80
- Dashboard accessible at http://traefik.localhost:8080

### ForwardAuth Middleware

The ForwardAuth middleware is configured to:
- Use OpenID Connect (OIDC) with Keycloak
- Protect routes with the `traefik-forward-auth` middleware label
- Store authentication in cookies with domain `.localhost`

## Customization

### Change Default Credentials

1. Update the Keycloak admin credentials in `docker-compose.yml`:
   ```yaml
   KEYCLOAK_ADMIN: your_admin_username
   KEYCLOAK_ADMIN_PASSWORD: your_admin_password
   ```

2. Modify test user credentials in `keycloak/realm-export.json`

3. Update the client secret in both `docker-compose.yml` and `keycloak/realm-export.json`

### Add More Users

You can add users via the Keycloak Admin Console:
1. Go to http://keycloak.localhost
2. Login with admin credentials
3. Select the "qdrant" realm
4. Navigate to Users → Add User

### Protect Additional Services

To protect other services, add the middleware to their labels:
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myservice.rule=Host(`myservice.localhost`)"
  - "traefik.http.routers.myservice.middlewares=traefik-forward-auth"
```

## Troubleshooting

### Services not accessible

Check that all containers are running:
```bash
docker-compose ps
```

View logs for specific services:
```bash
docker-compose logs traefik
docker-compose logs keycloak
docker-compose logs traefik-forward-auth
```

### Redirect loop

This usually means the ForwardAuth service cannot reach Keycloak. Check:
- All services are on the same Docker network
- Keycloak is fully started (check logs)
- The issuer URL in docker-compose.yml is correct

### "Invalid redirect URI" error

Ensure the redirect URIs in `keycloak/realm-export.json` match your setup. The realm import should handle this automatically.

### Reset everything

To start fresh:
```bash
docker-compose down -v
docker-compose up -d
```

## Security Notes

⚠️ **This setup is for local development only!**

For production use:
- Use HTTPS/TLS for all services
- Use strong, unique secrets and passwords
- Store secrets in a secure secret management system
- Enable proper SSL certificate validation
- Use a proper domain with valid SSL certificates
- Review and harden Keycloak security settings
- Enable HTTPS redirects in Traefik
- Set secure cookie flags

## Stopping the Stack

```bash
docker-compose down
```

To also remove volumes:
```bash
docker-compose down -v
```

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.