# Keycloak Configuration

## ⚠️ Security Warning

This Keycloak realm configuration is for **DEVELOPMENT ONLY**.

### Insecure Settings in this Configuration:

1. **Client Secret**: The client secret `qdrant-client-secret` is hardcoded and publicly visible
2. **User Password**: Test user password `testpassword` is stored in plaintext
3. **SSL Disabled**: `sslRequired` is set to `none`
4. **Public Registration**: User registration is enabled

### For Production Use:

1. **Generate Secure Secrets**:
   - Use a cryptographically secure random generator
   - Store secrets in a secrets management system (e.g., HashiCorp Vault, AWS Secrets Manager)
   - Never commit secrets to version control

2. **Enable SSL/TLS**:
   - Set `sslRequired` to `external` or `all`
   - Use valid SSL certificates
   - Configure Keycloak behind HTTPS

3. **Remove Test Users**:
   - Delete or disable the test user
   - Create real users through Keycloak admin console
   - Use strong password policies

4. **Review Security Settings**:
   - Disable public registration if not needed
   - Enable bruteforce protection
   - Configure appropriate token lifetimes
   - Review and restrict redirect URIs
   - Enable audit logging

5. **Use Environment Variables**:
   - Don't hardcode sensitive values in the realm export
   - Use Keycloak's variable substitution features
   - Configure via environment variables or config files

## Realm Configuration Details

- **Realm Name**: qdrant
- **Client ID**: qdrant-client
- **Test User**: testuser / testpassword
- **Supported Flows**: Authorization Code Flow (OAuth2/OIDC)

## Modifying the Configuration

To customize this realm for your needs:

1. Import the realm into Keycloak
2. Access Keycloak Admin Console at http://keycloak.localhost
3. Navigate to the "qdrant" realm
4. Make changes through the UI
5. Export the updated realm if needed

## Client Configuration

The `qdrant-client` is configured with:
- **Client Authentication**: ON (confidential client)
- **Standard Flow**: Enabled (Authorization Code Flow)
- **Valid Redirect URIs**:
  - `http://auth.localhost/*`
  - `http://qdrant.localhost/*`
- **Web Origins**:
  - `http://auth.localhost`
  - `http://qdrant.localhost`
