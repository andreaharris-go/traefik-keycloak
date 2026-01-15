# Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                               User Browser                               │
└────────────────────────────┬────────────────────────────────────────────┘
                             │
                             │ 1. HTTP Request to
                             │    http://qdrant.localhost
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          Traefik (Port 80)                              │
│                     Reverse Proxy & Load Balancer                       │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────┐       │
│  │            ForwardAuth Middleware (Applied)                  │       │
│  │  Checks: Is user authenticated?                             │       │
│  │  - If NO  → 2. Redirect to Keycloak (307)                  │       │
│  │  - If YES → Forward request to Qdrant                       │       │
│  └─────────────────────────────────────────────────────────────┘       │
└───────────┬──────────────────────────────┬──────────────────────────────┘
            │                              │
            │ 2. Not authenticated         │ 5. Authenticated
            │    Redirect to Keycloak      │    Forward to Qdrant
            │                              │
            ▼                              ▼
┌───────────────────────────┐   ┌──────────────────────────────┐
│       Keycloak            │   │          Qdrant              │
│  (keycloak.localhost)     │   │   (qdrant.localhost)         │
├───────────────────────────┤   ├──────────────────────────────┤
│                           │   │                              │
│ 3. Display login page     │   │ 6. Return Qdrant UI          │
│                           │   │                              │
│ 4. User enters:           │   │    - Dashboard               │
│    - Username: testuser   │   │    - Collections             │
│    - Password: testpassword│  │    - API access              │
│                           │   │                              │
│ ✓ Validate credentials    │   └──────────────────────────────┘
│ ✓ Issue OAuth2 token      │
│ ✓ Redirect back to        │
│   traefik-forward-auth    │
└───────────┬───────────────┘
            │
            │ 4. OAuth callback
            │    with auth code
            │
            ▼
┌───────────────────────────────────────────┐
│     Traefik Forward Auth Service          │
│        (auth.localhost)                   │
├───────────────────────────────────────────┤
│                                           │
│ • Exchange code for token                │
│ • Validate user with Keycloak            │
│ • Set authentication cookie              │
│ • Redirect user back to Qdrant           │
│                                           │
└───────────────────────────────────────────┘
            │
            │ 5. Set cookie & redirect
            │    back to original URL
            │
            ▼
       (Back to Traefik)


┌─────────────────────────────────────────────────────────────────────┐
│                    Supporting Services                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────┐        ┌─────────────────────────────┐  │
│  │   PostgreSQL DB      │───────▶│   Keycloak Storage          │  │
│  │  (keycloak-postgres) │        │   - Users                   │  │
│  └──────────────────────┘        │   - Clients                 │  │
│                                   │   - Sessions                │  │
│                                   │   - Realms                  │  │
│                                   └─────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘


Authentication Flow (Detailed):
═══════════════════════════════════════════════════════════════════════

Step 1: Initial Access (Unauthenticated)
   User → Traefik → ForwardAuth (no cookie) → 307 Redirect to Keycloak

Step 2: Keycloak Login
   User sees Keycloak login page → Enters credentials → Submits form

Step 3: OAuth2 Authorization
   Keycloak validates → Redirects to ForwardAuth with auth code

Step 4: Token Exchange
   ForwardAuth → Exchanges code for access token with Keycloak
   ForwardAuth → Validates token
   ForwardAuth → Sets authentication cookie (_forward_auth)
   ForwardAuth → Redirects user back to original URL (qdrant.localhost)

Step 5: Authenticated Access
   User → Traefik → ForwardAuth (cookie present) → Validates cookie → 
   Allows request → Qdrant responds with UI

Step 6: Subsequent Requests
   All requests include cookie → ForwardAuth validates → Access granted


Network Layout:
═══════════════
All services run in a Docker bridge network: traefik-keycloak_traefik-network

External Access:
  - Port 80  → Traefik (HTTP)
  - Port 8080 → Traefik Dashboard

Internal Services (not directly accessible):
  - keycloak:8080
  - postgres:5432
  - qdrant:6333
  - traefik-forward-auth:4181

Domain Routing (via Host headers):
  - qdrant.localhost     → Qdrant (protected)
  - keycloak.localhost   → Keycloak (public)
  - auth.localhost       → ForwardAuth (public)
  - traefik.localhost    → Traefik Dashboard (public)
```
