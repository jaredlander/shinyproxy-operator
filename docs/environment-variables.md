# Environment Variables for ShinyProxy Container

Starting with version 2.3.1, the ShinyProxy Operator supports setting environment variables for the ShinyProxy container when using Docker backend.

## Usage

You can add environment variables to your ShinyProxy configuration by adding an `env` section at the root level of the spec. The environment variables are specified as a key-value map.

### Example Configuration

```yaml
apiVersion: openanalytics.eu/v1
kind: ShinyProxy
metadata:
  name: shinyproxy
  namespace: default
spec:
  # Environment variables for the ShinyProxy container
  env:
    MY_CUSTOM_VARIABLE: "custom_value"
    DATABASE_URL: "jdbc:postgresql://localhost:5432/mydb"
    DEBUG_MODE: "true"
    LOG_LEVEL: "INFO"
  
  fqdn: shinyproxy-demo.local
  image: openanalytics/shinyproxy:3.2.1
  
  proxy:
    title: ShinyProxy
    authentication: simple
    containerBackend: docker
    docker:
      internal-networking: true
    users:
      - name: demo
        password: demo
        groups: scientists
    specs:
      - id: 01_hello
        display-name: Hello Application
        container-cmd: ["R", "-e", "shinyproxy::run_01_hello()"]
        container-image: openanalytics/shinyproxy-demo
```

## Notes

- Environment variables are passed directly to the ShinyProxy container
- The operator will automatically add required environment variables such as `PROXY_VERSION`, `PROXY_REALM_ID`, and `SPRING_CONFIG_IMPORT`
- **Security**: User-provided environment variables cannot override reserved environment variables (`PROXY_VERSION`, `PROXY_REALM_ID`, `SPRING_CONFIG_IMPORT`)
- **Validation**: Environment variable keys must follow standard naming conventions (alphanumeric characters and underscores, starting with a letter or underscore)
- **Validation**: Environment variable values cannot contain newline characters (to prevent injection attacks)
- Invalid environment variables will be skipped with a warning logged
- This feature is currently only available for Docker backend deployments
- For Kubernetes deployments, use `kubernetesPodTemplateSpecPatches` to add environment variables

## Use Cases

Common use cases for environment variables include:

1. **Configuring external services**: Database URLs, API endpoints, etc.
2. **Feature flags**: Enable or disable features based on environment
3. **Logging configuration**: Set log levels or destinations
4. **Secrets management**: Pass sensitive configuration (though consider using secure secret management instead)
5. **Java/Spring Boot properties**: Override default Spring Boot configuration

## Example with Spring Boot Properties

You can use environment variables to override Spring Boot properties:

```yaml
env:
  SPRING_PROFILES_ACTIVE: "production"
  LOGGING_LEVEL_EU_OPENANALYTICS: "DEBUG"
  SERVER_PORT: "8080"
```
