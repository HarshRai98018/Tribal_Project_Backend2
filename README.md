# TribalCraft Connect API

Spring Boot 3.x REST API backend for the TribalCraft Connect platform, deployed to **Azure App Service (Linux, Java 21 SE)**.

## Local Development

```bash
# Run with H2 in-memory database
mvn spring-boot:run -Dspring-boot.run.profiles=local
```

## Build

```bash
mvn clean package -DskipTests
# Executable JAR produced at: target/sampleBootLombok-0.0.1-SNAPSHOT.jar
```

## Azure App Service Configuration

### Java Runtime Settings

| Setting | Value |
|---------|-------|
| Java version | Java 21 |
| Java web server stack | Java SE (embedded server) |
| Startup command | *(leave blank — Azure auto-detects the JAR)* |

### Required Application Settings (Environment Variables)

Set these in **Azure Portal → App Service → Configuration → Application settings** or via the Azure CLI:

| Name | Description |
|------|-------------|
| `SPRING_DATASOURCE_URL` | JDBC connection string, e.g. `jdbc:mysql://host:port/dbname` |
| `SPRING_DATASOURCE_USERNAME` | Database username |
| `SPRING_DATASOURCE_PASSWORD` | Database password |
| `APP_JWT_SECRET` | Long random secret used to sign JWT tokens (≥ 32 chars) |
| `APP_CORS_ALLOWED_ORIGINS` | Comma-separated list of allowed origins, e.g. `https://yourfrontend.vercel.app` |
| `APP_RECAPTCHA_ENABLED` | `true` or `false` |
| `APP_RECAPTCHA_SITE_KEY` | Google reCAPTCHA v2 site key |
| `APP_RECAPTCHA_SECRET_KEY` | Google reCAPTCHA v2 secret key |
| `APP_SUPABASE_URL` | Supabase project URL |
| `APP_SUPABASE_SERVICE_ROLE_KEY` | Supabase service-role key |
| `APP_SUPABASE_BUCKET_NAME` | Supabase storage bucket name |
| `WEBSITES_PORT` | `80` — tells Azure the port the app listens on |
| `SERVER_PORT` | `80` — sets the embedded Tomcat port (fallback) |

> **Port resolution order**: The application resolves its listening port via
> `${PORT:${WEBSITES_PORT:${SERVER_PORT:8080}}}`.  
> Azure App Service (Linux) injects `PORT` at runtime; `WEBSITES_PORT` and `SERVER_PORT` serve as fallbacks.

### Health Check

Configure in **Azure Portal → App Service → Configuration → General settings → Health check path**:

```
/api/health
```

This endpoint returns `{"status":"UP"}` immediately without touching the database, so Azure's health probe passes quickly during container startup.

### CI/CD

The repository includes a GitHub Actions workflow at `.github/workflows/main_tribal-craft-api.yml` that:

1. Sets up Java 21 (Microsoft distribution)
2. Builds the fat JAR with `mvn clean package -DskipTests`
3. Uploads the executable JAR as a build artifact (plain JAR is excluded)
4. Logs in to Azure using OIDC (no long-lived secrets)
5. Deploys the JAR to Azure App Service

#### Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `AZUREAPPSERVICE_CLIENTID_*` | Azure service principal client ID |
| `AZUREAPPSERVICE_TENANTID_*` | Azure tenant ID |
| `AZUREAPPSERVICE_SUBSCRIPTIONID_*` | Azure subscription ID |
