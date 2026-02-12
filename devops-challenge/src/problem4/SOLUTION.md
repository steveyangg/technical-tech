Wrong API Port in Nginx

- In nginx/conf.d/default.conf: proxy_pass http://api:3001; But in api/src/index.js: app.listen(3000)
-> causes intermittent connection failures

- In docker-compose.yml:

depends_on:
  - postgres
  - redis

depends_on only ensures startup order, not readiness. API may start before PostgreSQL or Redis is ready, causing connection errors.

- None of the services (API, Postgres, Redis) have health checks. 
-> Docker cannot detect unhealthy containers, Nginx may route traffic to a broken API

---- Diagnosed the issues by checked container logs using:

docker compose logs api
docker compose logs nginx
- Reviewed docker-compose.yml for dependency and health management
- Observed connection errors and 502 responses.
- Inspected Nginx configuration and noticed port mismatch.

Fixing: 
1. Fix Nginx Port
Updated default.conf:
proxy_pass http://api:3000;

2. Add Health Checks

Updated docker-compose.yml:

api:
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:3000/status"]
    interval: 10s
    timeout: 5s
    retries: 3

postgres:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
    interval: 10s
    retries: 5
redis:
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
3. Improve Service Dependencies
depends_on:
  postgres:
    condition: service_healthy
  redis:
    condition: service_healthy
-> Ensures API starts only when dependencies are ready.

4. Monitoring and Alerts

In production, I would add:

Monitoring
Prometheus: metrics
Grafana: dashboards
Node exporter
PostgreSQL exporter
Logging
Centralized logs (ELK / Loki)

Alerts: 
API error rate > 5%
Response time > threshold
Database connection failures
Container restarts
High memory/CPU usage

Example alerts:
API unavailable for > 1 minute
Postgres unhealthy
Redis not responding

5. Prevention in Production

To prevent similar issues in production, I would:

    CI/CD Validation
Lint configs
Run integration tests
Validate ports and endpoints

    Infrastructure Best Practices
Mandatory health checks
Readiness probes
Resource limits
Auto-restart policies
    Production Architecture
Use Kubernetes instead of raw Docker Compose
Readiness/Liveness probes
Rolling deployments
Blue-green deployment

    Documentation
Maintain service contracts (ports, APIs)
Versioned configs
Runbooks