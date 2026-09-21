# 001 — Docker Network Connectivity

## Objective

Understand how containers communicate through a user-defined Docker bridge network and how Docker DNS provides service discovery by container name.

## Environment

- Fedora
- Docker
- `frontend` and `backend`
- User-defined bridge network: `conexion`

## Investigation

Both containers were attached to `conexion`. Docker automatically assigned private IP addresses such as:

```text
backend  → 172.20.0.2
frontend → 172.20.0.3
```

The exact addresses are not treated as stable identifiers.

### Test connectivity by name

```bash
docker exec -it frontend ping -c 3 backend
```

The name resolved to the backend IP and connectivity succeeded.

### Intentionally break the network

```bash
docker network disconnect conexion frontend
```

After the disconnect, the frontend could no longer use the Docker network for backend communication.

Testing the name:

```bash
docker exec -it frontend ping -c 3 backend
```

failed because Docker DNS on `conexion` was no longer available to the frontend.

Testing the backend IP directly:

```bash
docker exec -it frontend ping -c 3 172.20.0.2
```

also failed, showing that this was not only a DNS problem: network connectivity itself was gone.

### Restore

```bash
docker network connect conexion frontend
```

Connectivity was restored.

## Root cause

The frontend and backend were no longer attached to the same Docker network.

## Key concepts

### Docker DNS

On a user-defined bridge network:

```text
frontend → backend → 172.20.0.2
```

Applications should normally use service/container names rather than hardcoded container IPs.

### DNS vs connectivity

```text
ping backend
    ↓
DNS + connectivity

ping 172.20.0.2
    ↓
connectivity only
```

Separating these tests helps isolate failures.

## Lessons learned

- Docker assigns container IPs automatically.
- Container IPs should not be treated as stable application identifiers.
- User-defined networks provide DNS-based service discovery.
- Containers need appropriate shared network connectivity to communicate.
- Testing a hostname and an IP separately helps distinguish DNS from network failures.
