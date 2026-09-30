# Mission 009 — Container Up ≠ Application Healthy

## Objetivo

Practicar la diferencia entre:

- estado del contenedor;
- respuesta HTTP de la aplicación;
- estado del Docker healthcheck.

El escenario permite tener el contenedor en estado `Up`, Nginx respondiendo correctamente en `/`, pero el healthcheck en estado `unhealthy`.

## Iniciar el laboratorio

Desde este directorio:

```sh
docker compose up --build -d
```

La aplicación se publica en `http://localhost:8089`.

## Qué comprueba el healthcheck

Docker ejecuta cada 5 segundos:

```sh
curl --fail --silent http://127.0.0.1/healthz || exit 1
```

El endpoint `/healthz` comprueba que exista el fichero regular `/tmp/mission-009-healthy`.

- Fichero regular presente → HTTP 200 → healthcheck pasa.
- Fichero ausente o sustituido por un directorio → HTTP 503 → healthcheck falla.

El `ExitCode` mostrado en `.State.Health.Log` pertenece al comando del healthcheck, no al proceso principal del contenedor.

## Reglas del laboratorio

Sigue el proceso:

Problem → Investigation → Hypothesis → Evidence → Root Cause → Fix → Verification → Lessons Learned.

Herramientas permitidas:

```sh
docker ps
docker inspect
docker logs
docker exec
curl
```

## Limpiar el laboratorio

```sh
docker compose down
```
