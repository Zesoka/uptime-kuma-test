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

Dentro de la red de Docker, los monitores deben apuntar al nombre del servicio (no a `localhost`). Ojo con el puerto de `demo-api`: el contenedor escucha internamente en `8080` (se mapea a `8081` en el host), así que la URL interna necesita ese puerto explícito.

1. **HTTP simple** → `http://demo-web` para ver un monitor sano.
2. **Códigos de error** → `http://demo-api:8080/status/500` para ver cómo Uptime Kuma detecta caídas.
3. **Latencia / timeout** → `http://demo-api:8080/delay/5`, con un "Request Timeout" del monitor menor a 5s (por ejemplo 3s), para provocar un timeout controlado.
4. **Caída real** → `docker compose stop demo-web` y observar cómo Uptime Kuma marca el monitor como caído; `docker compose start demo-web` para recuperarlo.

## Notificaciones

Se configuró una notificación **Email (SMTP)** vía Gmail (`smtp.gmail.com:587`, STARTTLS) en Settings → Notifications, marcada como "Default" y aplicada a todos los monitores. La contraseña de aplicación de Gmail no se versiona en ningún archivo del repo; se carga a mano en el formulario de Uptime Kuma la primera vez.

Todos los monitores usan intervalo de chequeo de **20 segundos** y `Retries: 0`, para que una caída se detecte y notifique lo antes posible.

## Monitoreo de un stack de Wazuh (SIEM) externo

Si además tenés un stack de [Wazuh](https://github.com/wazuh/wazuh-docker) single-node corriendo en otra carpeta (manager + indexer + dashboard, con sus puertos publicados al host), podés monitorearlo desde este mismo Uptime Kuma sin unir las redes de Docker: como Uptime Kuma corre en un contenedor, usá `host.docker.internal` en vez de `localhost` para llegar a los puertos publicados por el otro compose.

Monitores de tipo **TCP Port** (evita lidiar con los certificados autofirmados de Wazuh):

| Monitor            | Host                    | Puerto |
|---------------------|-------------------------|--------|
| Wazuh Dashboard      | `host.docker.internal`  | 443    |
| Wazuh Indexer        | `host.docker.internal`  | 9200   |
| Wazuh Manager API    | `host.docker.internal`  | 55000  |

## Notas

- Los datos de Uptime Kuma persisten en el volumen nombrado `uptime-kuma-data`.
- Este entorno es solo para pruebas locales, no está pensado para producción.
