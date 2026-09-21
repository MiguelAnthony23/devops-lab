# DevOps Lab

Practical DevOps lab focused on building, breaking, troubleshooting, and automating infrastructure and applications.

The goal is to document hands-on engineering work rather than simply list technologies.

## Engineering approach

Every lab follows:

**Problem → Investigation → Hypothesis → Evidence → Root Cause → Fix → Verification → Lessons Learned**

## Current focus

- Docker and containerization
- Container networking
- Service and port troubleshooting
- Linux-based investigation
- Kubernetes
- Infrastructure as Code
- CI/CD
- Security
- Observability

## Repository structure

```text
devops-lab/
├── README.md
├── docker/
├── networking/
├── troubleshooting/
│   ├── README.md
│   ├── 001-docker-network-connectivity.md
│   └── 002-docker-port-troubleshooting.md
├── kubernetes/
├── terraform/
├── ci-cd/
├── security/
└── monitoring/
```

Directories are added when they contain real work; the repository is intentionally built incrementally.

## Labs

### Docker

Containerization, images, port publishing, processes, logs, and service behaviour.

### Networking

Docker bridge networks, container-to-container connectivity, DNS-based service discovery, IP addressing, and network troubleshooting.

### Troubleshooting

Failure scenarios investigated layer by layer rather than relying only on container status.

```text
DNS
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

## Roadmap

- [x] Docker fundamentals
- [x] Docker networking
- [x] Port and HTTP troubleshooting
- [ ] Dockerfiles and image optimization
- [ ] Kubernetes with kind
- [ ] Kubernetes troubleshooting
- [ ] Terraform
- [ ] CI/CD with GitHub Actions
- [ ] Security
- [ ] Monitoring and observability

## Environment

The lab is built locally on Fedora using open-source tooling. Cloud access is not required for the core exercises.

Current tooling includes:

- Docker
- kind
- Kubernetes
- Terraform
- PowerShell
- Git / GitHub

## Objective

The project is designed to demonstrate the ability to:

1. Deploy a service.
2. Understand how its components communicate.
3. Intentionally introduce failures.
4. Investigate failures using observable evidence.
5. Identify root causes.
6. Apply appropriate fixes.
7. Verify recovery.
8. Document the reasoning so the incident can be reproduced.

## Status

🚧 Active learning project.
