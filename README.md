# Contenedores: Frontend + Backend + Base de Datos

Este proyecto es un ejemplo de una aplicación dividida en **tres contenedores**:
- **frontend**: Nginx sirviendo archivos estáticos y actuando como *reverse-proxy* para `/api` hacia el backend.
- **backend**: API REST sencilla en Node.js (Express) que se conecta a PostgreSQL.
- **db**: Base de datos PostgreSQL.

> Puertos expuestos por defecto:
> - Frontend: `http://localhost:8080`
> - Backend: `http://localhost:3000`
> - PostgreSQL: `localhost:5432` (solo dentro de la red de Docker, salvo que cambies puertos)

---

## Estructura del repositorio

```
three-tier-docker-app/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   ├── index.html
│   └── nginx.conf
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js
│   └── .env.example
└── db/
    └── Dockerfile
```

---

## ¿Cómo interactúan los contenedores?

Todos los servicios comparten la red de Docker `appnet` declarada en `docker-compose.yml`. Gracias a esto, pueden resolverse por **nombre de servicio**:

- El **backend** se conecta a la **db** usando `host=db`, `port=5432` (variables de entorno `POSTGRES_*`).
- El **frontend** (Nginx) expone estáticos y además **redirige las peticiones** a `/api/*` hacia `http://backend:3000/` con `proxy_pass`. Desde el navegador, tú consultas `http://localhost:8080/api/...` y Nginx reenvía al backend dentro de la red Docker.

### Flujo de una petición (ejemplo POST /api/todos)
1. El usuario abre `http://localhost:8080` (frontend).
2. El JS del frontend hace `fetch('/api/todos')`.
3. Nginx recibe `/api/todos` y lo **proxy-pasa** a `backend:3000`.
4. El backend ejecuta SQL sobre PostgreSQL (servicio `db`) para guardar/leer datos.
5. La respuesta vuelve por Nginx al navegador.

---

## Construcción y ejecución

Requisitos: Docker y Docker Compose.

```bash
# 1) Posicionarse en la carpeta raíz
cd three-tier-docker-app

# 2) Construir y levantar todo
docker compose up --build
# (o: docker-compose up --build)

# 3) Abrir el frontend
xdg-open http://localhost:8080 || open http://localhost:8080 || start http://localhost:8080
```

Los logs de cada servicio aparecerán en la terminal. Para tumbar los contenedores:
```bash
docker compose down
```

> **Nota:** La primera vez que se levanta, el backend crea la tabla `todos` automáticamente.

---

## Variables de entorno

El **backend** lee variables desde el entorno del contenedor (definidas en `docker-compose.yml`). Puedes copiar `backend/.env.example` a `.env` si quieres levantar el backend fuera de Docker.

- `POSTGRES_HOST=db`
- `POSTGRES_PORT=5432`
- `POSTGRES_DB=appdb`
- `POSTGRES_USER=appuser`
- `POSTGRES_PASSWORD=apppassword`
- `PORT=3000`

La **db** (PostgreSQL) se configura en su Dockerfile y en `docker-compose.yml`. Usa un volumen `dbdata` para persistencia.

---

## Endpoints del backend

- `GET /api/health` → chequeo de salud.
- `GET /api/todos` → lista todos los TODOs.
- `POST /api/todos` → crea un TODO. Body JSON: `{ "title": "texto" }`.

Ejemplo con `curl`:
```bash
curl -X POST http://localhost:8080/api/todos -H "Content-Type: application/json" -d '{"title":"Aprender Docker Compose"}'
curl http://localhost:8080/api/todos
```

---

## Personalización

- **Frontend**: reemplaza `frontend/index.html` por tu app (React/Vite/Next estáticos). Mantén el `location /api/` del `nginx.conf`.
- **Backend**: añade rutas en `backend/server.js`, crea capas de negocio, etc.
- **DB**: agrega scripts de inicialización (por ejemplo `init.sql`) copiándolos a `/docker-entrypoint-initdb.d/` en la imagen.

---

## Seguridad y consideraciones

- Este es un demo. Para producción, agrega:
  - Variables de entorno seguras (usa secretos).
  - Migraciones controladas (e.g., Prisma, Knex, Sequelize).
  - CORS más restrictivo o integra todo tras el proxy del frontend.
  - Healthchecks más robustos y `restart: unless-stopped` si aplica.
  - Backups del volumen `dbdata`.


