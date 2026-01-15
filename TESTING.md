# Testing Guide

This document provides step-by-step instructions for testing the Keycloak authentication flow for Qdrant UI.

## Prerequisites

- Docker and Docker Compose installed
- All services running (`docker compose up -d`)
- Wait 30-60 seconds for all services to be fully initialized

## Test 1: Verify Redirect to Keycloak Login

### Expected Behavior
When accessing Qdrant UI without authentication, users should be automatically redirected to Keycloak login page.

### Steps

1. **Open your web browser**

2. **Navigate to Qdrant UI**
   ```
   http://qdrant.localhost
   ```

3. **Verify automatic redirect**
   - You should be automatically redirected to the Keycloak login page
   - The URL should change to something like:
     ```
     http://keycloak.localhost/realms/qdrant/protocol/openid-connect/auth?...
     ```

4. **Confirm login page displays**
   - You should see a Keycloak login form
   - The page should show "Sign in to your account" or similar

### Test with curl (Alternative)
```bash
curl -I http://localhost -H "Host: qdrant.localhost"
```

**Expected Result:** HTTP 307 redirect with `Location` header pointing to Keycloak

## Test 2: Successful Login Flow

### Steps

1. **Access Qdrant UI** (you'll be redirected to Keycloak)
   ```
   http://qdrant.localhost
   ```

2. **Enter test credentials**
   - Username: `testuser`
   - Password: `testpassword`

3. **Click "Sign In"**

4. **Verify successful authentication**
   - You should be redirected back to Qdrant UI
   - The URL should be `http://qdrant.localhost`
   - You should see the Qdrant dashboard/UI

5. **Verify session persistence**
   - Close and reopen the browser tab
   - Navigate to `http://qdrant.localhost` again
   - You should NOT be redirected to login (session cookie is active)
   - You should see the Qdrant UI immediately

## Test 3: Verify Protected Access

### Test Direct API Access
```bash
curl -I http://localhost/collections -H "Host: qdrant.localhost"
```

**Expected Result:** HTTP 307 redirect to Keycloak (not authenticated)

### Test with Authentication Cookie
After logging in through the browser, export the cookie and test:
```bash
# Get cookie from browser developer tools (usually named _forward_auth)
curl -I http://localhost/collections \
  -H "Host: qdrant.localhost" \
  -H "Cookie: _forward_auth=YOUR_COOKIE_VALUE"
```

**Expected Result:** HTTP 200 or appropriate Qdrant response (authenticated)

## Test 4: Verify Logout Behavior

### Steps

1. **Clear browser cookies** for localhost domains
   - Open browser Developer Tools (F12)
   - Go to Application/Storage > Cookies
   - Delete all cookies for `localhost` domain

2. **Try to access Qdrant UI again**
   ```
   http://qdrant.localhost
   ```

3. **Verify redirect to login**
   - You should be redirected to Keycloak login page again
   - This confirms that clearing the session requires re-authentication

## Test 5: Verify Other Services

### Access Keycloak Admin Console
```
http://keycloak.localhost
```

- Username: `admin`
- Password: `admin`
- Should allow direct access (not protected by ForwardAuth)

### Access Traefik Dashboard
```
http://traefik.localhost:8080
```

- Should show the Traefik dashboard
- Should display all configured routers and services

## Test 6: Integration Test

Complete end-to-end test:

1. **Start fresh** (clear all cookies)
2. **Access Qdrant UI** → Should redirect to Keycloak
3. **Login with test user** → Should redirect back to Qdrant
4. **Use Qdrant UI** → Should work normally
5. **Close browser** → Session saved
6. **Reopen and access Qdrant** → Should work without login
7. **Clear cookies** → Next access requires login again

## Troubleshooting

### Issue: Services not accessible

**Check all containers are running:**
```bash
docker compose ps
```

All services should show "Up" status.

### Issue: Redirect loop

**Check logs:**
```bash
docker compose logs traefik-forward-auth
docker compose logs keycloak
```

Look for errors related to:
- OIDC configuration
- Network connectivity between services
- Invalid redirect URIs

### Issue: "Invalid redirect URI" error

**Verify Keycloak configuration:**
1. Access Keycloak admin console
2. Navigate to Clients > qdrant-client
3. Check "Valid Redirect URIs" includes:
   - `http://auth.localhost/*`
   - `http://qdrant.localhost/*`

### Issue: Authentication works but Qdrant UI doesn't load

**Check Qdrant logs:**
```bash
docker compose logs qdrant
```

Verify Qdrant service is running properly.

## Success Criteria

All tests should pass with the following outcomes:

✅ Unauthenticated access to Qdrant UI redirects to Keycloak  
✅ Valid credentials allow access to Qdrant UI  
✅ Invalid credentials show error message  
✅ Authenticated session persists across browser tabs  
✅ Cleared cookies require re-authentication  
✅ ForwardAuth middleware properly intercepts all Qdrant requests  

## Additional Notes

- The authentication cookie is set for the `.localhost` domain
- Session lifetime is 12 hours by default
- All traffic is HTTP (not HTTPS) for local development
- For production, enable HTTPS and secure cookies
