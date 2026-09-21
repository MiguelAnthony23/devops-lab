# Troubleshooting

This directory documents practical troubleshooting scenarios performed in the DevOps lab.

## Methodology

Each incident follows:

1. **Problem**
2. **Investigation**
3. **Hypothesis**
4. **Evidence**
5. **Root cause**
6. **Fix**
7. **Verification**
8. **Lessons learned**

## Troubleshooting layers

```text
Name resolution
      ↓
IP connectivity
      ↓
TCP / Port
      ↓
HTTP
      ↓
Application / Service
```

A container being `Up` is not sufficient evidence that the application inside it is healthy.

## Incidents

- [001 — Docker network connectivity](./001-docker-network-connectivity.md)
- [002 — Docker port and HTTP troubleshooting](./002-docker-port-troubleshooting.md)
