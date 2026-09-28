# 003 — Container lifecycle and PID 1

## Objective

Investigate what happens when the main Nginx process is stopped inside a container and understand why stopping the service can also stop the container.

## Scenario

The `backend` container uses the official Nginx image.

Initial verification showed:

- The container was running.
- Nginx master and worker processes were running.
- Nginx listened on TCP/80.
- `curl http://localhost` returned `HTTP/1.1 200 OK`.
- `frontend` could resolve `backend` through Docker DNS and reach TCP/80.

## Investigation

The experiment was:

```bash
docker exec -it backend nginx -s stop
```

This sends a stop signal to the Nginx master process.

Afterwards, the container stopped instead of remaining `Up`.

## Why did the container stop?

The important concept is the container's main process, PID 1.

Conceptually:

```text
backend container
└── PID 1 → nginx master
              ├── nginx worker
              ├── nginx worker
              └── ...
```

The Nginx workers are child processes of the master. They do not control the lifecycle of the container.

When the Nginx master process terminates:

```text
nginx master stops
      ↓
PID 1 terminates
      ↓
Docker sees the main container process exit
      ↓
container → Exited
```

Therefore, the experiment produced:

```text
Container → Exited
Nginx     → stopped
HTTP      → unavailable
```

It did **not** produce the desired scenario:

```text
Container → Up
Nginx     → stopped
HTTP      → unavailable
```

## Verification

Before stopping Nginx:

```bash
docker container top backend
```

showed the Nginx master and worker processes.

HTTP verification:

```bash
docker exec -it backend curl -I http://localhost
```

returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.31.5
```

From the frontend:

```bash
docker exec -it frontend curl -I -s http://backend:80
```

also returned HTTP 200.

After stopping Nginx, the container exited because the main process had terminated.

## Important distinction

`docker ps` primarily tells us whether the container's main process is running.

It does not universally prove that an application is healthy.

In a more complex container where PID 1 is a supervisor or wrapper process, the container could remain `Up` while an internal service is unavailable.

For the official Nginx image used here, Nginx is the main process, so stopping Nginx also stops the container.

## Lessons learned

- PID 1 has a fundamental role in a container's lifecycle.
- Nginx master and worker processes have different roles.
- Stopping a child worker is not the same as stopping the master process.
- Stopping the main Nginx process causes this container to exit.
- `docker ps` is a container-state check, not a complete application-health check.
- Service health should be verified with application-level tests such as HTTP requests.
- Docker healthchecks provide a way to express application health separately from basic container state.

## Next step

Build a controlled healthcheck scenario where:

```text
Container → running
Application → unhealthy
```

This will provide a practical bridge toward Kubernetes `livenessProbe` and `readinessProbe`.
