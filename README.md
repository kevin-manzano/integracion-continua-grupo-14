# Integración Continua - Grupo 14

Proyecto que usa dos contenedores Docker los cuales se comunican entre si:

- **api**: servicio backend en Node.js + Express
- **db**: base de datos PostgreSQL

Ambos contenedores se comunican a través de una red interna de Docker definida en `docker-compose.yml`. La API se conecta a la base de datos usando el nombre del servicio (`db`) como hostname.

## Requisitos

- [Docker](https://www.docker.com/) y Docker Compose (incluido en Docker Desktop)

## Estructura del proyecto

```
.
├── api/                # Código de la API (Node.js + Express)
│   ├── src/
│   ├── Dockerfile
│   └── package.json
├── db/
│   └── init/           # Scripts SQL que inicializan el esquema y datos de prueba
├── docker-compose.yml
├── .env.example        # Plantilla de variables de entorno
└── .env                # Variables de entorno reales (no se sube al repo)
```

## Cómo levantar el proyecto

1. Clona el repositorio.
2. Copia el archivo de variables de entorno de ejemplo:

   ```bash
   cp .env.example .env
   ```

   Ajusta los valores si lo necesitas (usuario, contraseña, puertos, etc.).

3. Levanta los contenedores:

   ```bash
   docker compose up -d --build
   ```

4. Verifica que ambos contenedores estén corriendo:

   ```bash
   docker compose ps
   ```

## Endpoints disponibles

La API queda expuesta en `http://localhost:3000` (puerto configurable vía `API_PORT` en `.env`).

| Método | Ruta       | Descripción                                   |
|--------|------------|------------------------------------------------|
| GET    | `/`        | Mensaje de bienvenida                           |
| GET    | `/health`  | Verifica que la API y la conexión a la DB estén activas |
| GET    | `/tasks`   | Lista todas las tareas guardadas en la DB       |
| POST   | `/tasks`   | Crea una nueva tarea (body: `{ "title": "..." }`) |

### Ejemplos con curl

```bash
curl http://localhost:3000/health

curl http://localhost:3000/tasks

curl -X POST http://localhost:3000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Nueva tarea"}'
```

## Detener el proyecto

```bash
docker compose down
```

Para eliminar también los datos persistidos de la base de datos:

```bash
docker compose down -v
```
