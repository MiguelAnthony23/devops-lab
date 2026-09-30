# 009 — Container Up ≠ Application Healthy

## Problem

El contenedor `mission-009-web` permanecía en estado `Up`, pero Docker lo marcaba como `unhealthy`.

La aplicación principal seguía respondiendo correctamente en HTTP:

```text
HTTP/1.1 200 OK
Mission 009 web service is responding.
```

Por tanto, había una discrepancia entre:

- estado del contenedor;
- disponibilidad de Nginx;
- estado del healthcheck.

## Investigation

### 1. Estado del contenedor

Inicialmente:

```text
mission-009-web    Up ... (healthy)
```

Después de provocar el fallo:

```text
mission-009-web    Up ... (unhealthy)
```

Esto demuestra que `Up` no significa automáticamente que la aplicación esté sana.

### 2. Respuesta de la aplicación

La raíz seguía respondiendo:

```sh
curl -i http://localhost:8089/
```

Resultado:

```text
HTTP/1.1 200 OK
Mission 009 web service is responding.
```

Nginx estaba funcionando y atendiendo tráfico.

### 3. Definición del healthcheck

El healthcheck estaba definido en Compose como:

```yaml
healthcheck:
  test: ["CMD-SHELL", "curl --fail --silent http://127.0.0.1/healthz || exit 1"]
  interval: 5s
  timeout: 2s
  retries: 2
  start_period: 3s
```

Puntos importantes:

- `127.0.0.1` representa el propio contenedor.
- `/healthz` es un endpoint servido por Nginx.
- `curl --fail` considera los errores HTTP como fallo del comando.
- `|| exit 1` garantiza un código de salida distinto de cero cuando la comprobación falla.
- Docker usa el resultado de ese comando para calcular el estado de salud.

### 4. Condición real de `/healthz`

La configuración de Nginx contenía:

```nginx
if (!-f /tmp/mission-009-healthy) {
    return 503 "Application health check failed.\n";
}
return 200 "Application is healthy.\n";
```

La condición `-f` no significa solamente "existe una ruta". Comprueba que la ruta exista y sea un **fichero regular**.

### 5. Evidencia del fallo

El fichero utilizado por la comprobación fue eliminado intencionadamente.

Después se obtuvo:

```text
HTTP/1.1 503 Service Temporarily Unavailable
Application health check failed.
```

El historial del healthcheck mostró:

```text
ExitCode: 1
```

Aquí `ExitCode` corresponde al comando del healthcheck:

- `0` → comprobación correcta;
- distinto de `0` → comprobación fallida.

### 6. Investigación adicional: fichero vs directorio

Durante el fix se creó accidentalmente un directorio con el mismo nombre:

```sh
docker exec mission-009-web mkdir /tmp/mission-009-healthy
```

La evidencia fue:

```text
drwxr-xr-x    1 root root ... /tmp/mission-009-healthy
^
```

La primera letra `d` indica que era un directorio.

Esto seguía provocando HTTP 503 porque Nginx utilizaba `-f`, que requiere un fichero regular.

Se eliminó el directorio y se creó el fichero vacío:

```sh
docker exec mission-009-web rm -r /tmp/mission-009-healthy
docker exec mission-009-web touch /tmp/mission-009-healthy
```

## Root cause

La causa raíz del estado `unhealthy` fue que el endpoint `/healthz` dependía de la existencia del fichero regular `/tmp/mission-009-healthy`, y ese fichero había sido eliminado.

La cadena causal fue:

```text
/tmp/mission-009-healthy no existe
        ↓
-f devuelve false
        ↓
/healthz devuelve HTTP 503
        ↓
curl --fail devuelve código distinto de 0
        ↓
healthcheck termina con ExitCode 1
        ↓
Docker marca el contenedor como unhealthy
```

El contenedor no estaba caído y Nginx no estaba detenido. El problema estaba específicamente en la condición utilizada para determinar la salud de la aplicación.

## Fix

Se eliminó el directorio creado accidentalmente y se restauró el recurso esperado como fichero regular:

```sh
docker exec mission-009-web rm -r /tmp/mission-009-healthy
docker exec mission-009-web touch /tmp/mission-009-healthy
```

No fue necesario modificar Nginx, Docker Compose ni el healthcheck.

## Verification

### 1. El recurso volvió a ser un fichero

```text
-rw-r--r--    1 root root 0 ... /tmp/mission-009-healthy
^```

La primera letra `-` indica un fichero regular.

### 2. Endpoint de salud

```text
HTTP/1.1 200 OK
Application is healthy.
```

### 3. Historial del healthcheck

Después del fix:

```text
ExitCode: 0
Output: "Application is healthy.\n"
```

Esto demuestra que el comando del healthcheck terminó correctamente.

### 4. Resultado esperado

La cadena quedó restaurada:

```text
fichero regular presente
        ↓
/healthz → HTTP 200
        ↓
healthcheck → ExitCode 0
        ↓
Docker → healthy
```

## Lessons learned

1. **`Up` no significa `healthy`.** El proceso principal puede seguir funcionando mientras una comprobación específica de salud falla.
2. **Un healthcheck es un comando definido por nosotros.** Docker no decide por sí solo si la aplicación está sana; ejecuta la prueba configurada.
3. **Hay que distinguir proceso, aplicación y healthcheck.** Nginx podía responder `200` en `/` mientras `/healthz` devolvía `503`.
4. **Los códigos de salida son evidencia.** En un healthcheck, `ExitCode 0` indica éxito y un valor distinto de cero indica fallo.
5. **El tipo de recurso importa.** Un directorio llamado `mission-009-healthy` no satisface `-f`; se necesitaba un fichero regular.
6. **La verificación debe cerrar la cadena causal.** No basta con crear el recurso: comprobamos el tipo de fichero, el endpoint y el historial del healthcheck.
7. **Los healthchecks pueden ser una base para mecanismos posteriores de Kubernetes**, como readiness y liveness probes, donde también es importante diferenciar disponibilidad del proceso y salud/estado de la aplicación.
