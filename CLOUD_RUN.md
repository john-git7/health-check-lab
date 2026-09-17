# Cloud Run Mapping

This document maps the health and readiness checks implemented in this repository to the equivalent concepts in Google Cloud Run (and Kubernetes).

## Local Implementation
We implemented two endpoints:
- `GET /health` (liveness check): A shallow check to verify the application process is running.
- `GET /ready` (readiness check): A deep check to verify the application can connect to its dependencies (e.g., the database).

In our `docker-compose.yml`, we used a `HEALTHCHECK` mapped to the `/health` endpoint to automatically restart the container if the application hangs.

## Cloud Run Concepts

### 1. Startup Probes
Cloud Run uses startup probes to determine when an application has successfully started and is ready to accept traffic.
- **Equivalent:** A customized probe checking the `/health` or `/ready` endpoint, or a TCP socket check on the port.
- **Why it matters:** It prevents traffic from being routed to the container while it's still initializing.

### 2. Liveness Probes
Cloud Run liveness probes continuously monitor the health of the container during its lifecycle.
- **Equivalent:** Our `GET /health` endpoint.
- **Behavior:** If the liveness probe fails, Cloud Run will restart the container instance, much like the `restart: unless-stopped` policy triggered by the Docker Compose `HEALTHCHECK`.

### Kubernetes Context
In Kubernetes:
- **Liveness Probe:** Maps to our `GET /health` endpoint. Restarts the pod if it fails.
- **Readiness Probe:** Maps to our `GET /ready` endpoint. Removes the pod from the service load balancer if it fails (doesn't restart it, just stops sending traffic).
- **Startup Probe:** Prevents other probes from running until the app has started.
