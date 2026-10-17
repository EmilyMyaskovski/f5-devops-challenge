# Root Cause Analysis

## Defect 1: Proxy Image Fails to Build

### Symptom / Error Output
The proxy Docker image failed to build during Nginx installation.

### How I Diagnosed It
`docker compose build --progress=plain --no-cache proxy`

### Root Cause
The custom `nginx.conf` was copied before Nginx was installed, causing `dpkg` to prompt for a configuration file conflict during a non-interactive Docker build.

### Fix
Moved the Nginx package installation before the `COPY` instruction, allowing Nginx to install its default configuration before replacing it with the custom configuration.

---

## Defect 2: Proxy Cannot Reach the Backend

### Symptom / Error Output
Requests to `/api/info` through the Nginx proxy returned HTTP 502 Bad Gateway.

### How I Diagnosed It
`./scripts/run-tests.sh`

The Nginx logs showed that requests were being forwarded to `127.0.0.1:8080`.

### Root Cause
The Nginx upstream was configured to use `127.0.0.1:8080`, which refers to the proxy container itself instead of the backend container.

### Fix
Changed the upstream address in `proxy/nginx.conf` from `127.0.0.1:8080` to `backend:8080`, using Docker Compose's internal DNS to resolve the backend service name.

---

## Defect 3: Tests Still Fail After Fixing Defect 2

### Symptom / Error Output
Integration tests failed with HTTP 502 Bad Gateway because the backend was not ready to accept requests.

### How I Diagnosed It
- `./scripts/run-tests.sh`
- `docker compose logs backend`

The backend logs showed that the application was still loading its configuration when the tests started.

### Root Cause
The backend requires approximately 8 seconds to load its configuration before starting the HTTP server. Docker Compose used the default `service_started` condition, which does not guarantee that the backend is ready to accept HTTP requests.

### Fix, and Why I Chose It Over the Alternatives
Added an HTTP healthcheck to the backend using Python's `urllib.request` and changed the proxy dependency to `condition: service_healthy`.

This ensures that the proxy starts only after the backend is ready to accept HTTP requests. I chose this approach over a fixed `sleep` delay because it checks actual service readiness instead of relying on a predetermined startup time.

---

## Defect 4: CI Pipeline Fails

### Symptom / Error Output
The CI workflow executes `./scripts/run-tests.sh`, but the script was not marked as executable in Git, which could cause a `Permission denied` error on the Linux runner.

### How I Diagnosed It
`git ls-files -s scripts/run-tests.sh`

The file mode was `100644`, indicating that the executable permission was missing.

### Root Cause
The script was tracked with file mode `100644` instead of `100755`, so the executable permission required by the CI workflow was not preserved in Git.

### Fix
Ran `git update-index --chmod=+x scripts/run-tests.sh` to preserve the executable permission in Git.

Verified that the file mode changed to `100755`. Final verification in GitHub Actions is pending.

---

## Bonus Questions

### Bonus 1: Proxy Image Size Optimization

#### Initial State
The proxy image was based on Ubuntu 24.04 and included unnecessary packages such as `curl`, `htop`, `vim`, and `git`.

#### How I Diagnosed It
Measured the image size using:

`docker image inspect f5-devops-challenge-proxy:latest --format '{{.Size}}'`

#### Reason for Improvement
The additional packages increased the image size without being required for Nginx to operate.

#### Fix
Replaced Ubuntu with the official `nginx:alpine` image and removed the unnecessary packages and `apt-get` installation step.

#### Result
- **Before:** 121,179,872 bytes (121.18 MB).
- **After:** 28,817,875 bytes (28.82 MB).
- **Size reduction:** Approximately 76.2%.
- All integration tests passed.

---

### Bonus 2: Running Nginx as a Non-root User

#### Initial State
The original Nginx container ran its master process as root, increasing the potential impact of a security compromise.

#### How I Diagnosed It
Checked the configured container user using:

`docker image inspect f5-devops-challenge-proxy --format '{{.Config.User}}'`

The result was empty, indicating that no explicit user was configured.

#### Reason for Improvement
Running applications with root privileges unnecessarily increases security risks.

#### Fix
- Added `USER nginx` to the Dockerfile.
- Changed the internal listening port from `80` to `8080`.
- Updated Docker Compose port mappings and the test-runner's proxy URL.
- Configured writable temporary directories and moved the PID file to `/tmp/nginx.pid`.

#### Result
The proxy runs as the unprivileged `nginx` user, reducing the potential impact of a compromised container. All integration tests passed.

---

### Bonus 3: Additional Security Hardening

#### Initial State
Nginx could expose its version number in HTTP response headers and generated error pages.

#### How I Diagnosed It
Inspected the Nginx configuration and checked HTTP response headers using:

`curl -i http://localhost:8081/nginx-health`

#### Reason for Improvement
Exposing the exact Nginx version provides unnecessary information that could help attackers identify known vulnerabilities.

#### Fix
- Added `server_tokens off;` inside the `http` block in `proxy/nginx.conf`.
- Corrected the Docker Compose port mapping from `8081:80` to `8081:8080` to match the non-root Nginx configuration.

#### Result
- Confirmed HTTP 200 responses from the proxy.
- Verified that the `Server` header no longer exposes the Nginx version.
- All integration tests passed.

---

## Anything You Would Do Differently With More Time

- **Expand integration tests** to cover HTTP errors, backend startup readiness, and recovery after temporary service outages.
- **Improve system resilience** by implementing automated recovery mechanisms, including restart policies, request timeouts, retries with exponential backoff, and health monitoring with alerts.