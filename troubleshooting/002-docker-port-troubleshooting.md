# 002 — Docker Port and HTTP Troubleshooting

## Objective

Understand container ports, published host ports, listening services, Docker DNS, TCP connectivity, and HTTP connectivity.

## Explicit host-to-container mapping

```bash
docker run -d -p 8080:80 nginx
```

This means:

```text
Host port 8080 → Container port 80
```

## Publishing without specifying the host port

```bash
docker run -d -p 8080 nginx
```

This publishes container port 8080 using a dynamically selected host port. It does **not** mean host 8080 → container 80.

An example mapping was:

```text
0.0.0.0:32769->80/tcp
[::]:32769->80/tcp
```

## Verify the service

Nginx configuration was inspected:

```bash
docker exec -it backend grep -r "listen" /etc/nginx/
```

Nginx was listening on port 80.

Local verification:

```bash
docker exec -it backend curl -I http://localhost
```

returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.31.5
```

## Important localhost distinction

`localhost` inside a container refers to that same container.

Therefore, from `frontend`, this:

```text
http://localhost
```

does not test the backend.

The correct frontend-to-backend test is:

```bash
docker exec -it frontend curl -I -s http://backend:80
```

which returned HTTP 200.

## Closed-port investigation

Testing port 90:

```bash
docker exec -it frontend curl -v http://backend:90
```

showed:

```text
Host backend was resolved
IPv4: 172.20.0.2
Trying 172.20.0.2:90...
connect ... failed: Connection refused
```

This proves:

- DNS worked.
- Network connectivity worked.
- TCP connection to port 90 failed.
- HTTP was never reached.
- No application-level request could be processed.

## Troubleshooting model

```text
DNS
 ↓
IP connectivity
 ↓
TCP / port
 ↓
HTTP
 ↓
Application
```

For port 80:

```text
DNS       ✓
Network   ✓
TCP 80    ✓
HTTP      ✓
Nginx     ✓
```

For port 90:

```text
DNS       ✓
Network   ✓
TCP 90    ✗
HTTP      —
Nginx     —
```

## Key lesson

A container being `Up` only tells us that the container process is running. It does not prove that the expected service is running, listening on the expected port, reachable over the network, or responding correctly.

Each layer must be verified independently.
