# Uptime Kuma - Entorno de pruebas

Entorno local con Docker Compose para probar [Uptime Kuma](https://github.com/louislam/uptime-kuma) contra un par de servicios de ejemplo, sin depender de infraestructura real.

## Servicios

| Servicio     | Descripción                                   | URL local                |
|--------------|------------------------------------------------|---------------------------|
| `uptime-kuma`| Panel de monitoreo Uptime Kuma                  | http://localhost:3001     |
| `demo-web`   | Nginx estático, para probar monitores HTTP(S)   | http://localhost:8080     |
| `demo-api`   | [go-httpbin](https://github.com/mccutchen/go-httpbin), permite forzar códigos de estado y demoras (`/status/500`, `/delay/5`, etc.) | http://localhost:8081 |

## Uso

Levantar el entorno:

```bash
docker compose up -d
```

Abrir Uptime Kuma en http://localhost:3001 y crear el usuario administrador la primera vez.

Detener el entorno:

```bash
docker compose down
```

Detener y borrar también los datos de Uptime Kuma:

```bash
docker compose down -v
```

## Monitores sugeridos para probar

1. **HTTP simple** → `http://demo-web` (usar el nombre del servicio dentro de la red de Docker, o `http://localhost:8080` si el monitor se define desde fuera del contenedor de Kuma).
2. **Códigos de error** → `http://demo-api/status/500` para ver cómo Uptime Kuma detecta caídas.
3. **Latencia / timeout** → `http://demo-api/delay/5` para probar el umbral de timeout de un monitor.
4. **Caída real** → `docker compose stop demo-web` y observar cómo Uptime Kuma marca el monitor como caído; `docker compose start demo-web` para recuperarlo.

## Notas

- Los datos de Uptime Kuma persisten en el volumen nombrado `uptime-kuma-data`.
- Este entorno es solo para pruebas locales, no está pensado para producción.
