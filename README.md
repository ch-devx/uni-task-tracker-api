# tareas-uni-api

Cloudflare Worker que sirve como API REST para la app [tareas-uni](https://github.com/ch-devx/tareas-uni). Se conecta a una base de datos Neon PostgreSQL y expone endpoints para gestionar tareas y materias.

**Worker URL:** https://uni-tasks-worker.tareas-uni.workers.dev

## Stack

- Cloudflare Workers (JavaScript)
- [@neondatabase/serverless](https://github.com/neondatabase/serverless) — driver de Neon para entornos edge
- Neon PostgreSQL — base de datos

## Endpoints

### Materias
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/subjects` | Listar todas las materias |
| POST | `/subjects` | Crear materia |
| PUT | `/subjects/:id` | Editar materia |
| DELETE | `/subjects/:id` | Eliminar materia |

### Tareas
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/tasks` | Listar tareas pendientes (orden por deadline) |
| GET | `/tasks/done` | Listar tareas completadas |
| POST | `/tasks` | Crear tarea |
| PUT | `/tasks/:id` | Editar tarea |
| PATCH | `/tasks/:id/toggle` | Alternar estado pending/done |
| DELETE | `/tasks/:id` | Eliminar tarea |

## Estructura

```
tareas-uni-api/
├── src/
│   └── index.js       # Lógica completa del Worker
├── wrangler.jsonc     # Configuración de Cloudflare
└── package.json
```

## Setup

### 1. Requisitos

- [Node.js](https://nodejs.org/) LTS
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/): `npm install -g wrangler`
- Cuenta en [Cloudflare](https://cloudflare.com)
- Base de datos en [Neon](https://neon.tech)

### 2. Clonar e instalar dependencias

```bash
git clone https://github.com/ch-devx/tareas-uni-api.git
cd tareas-uni-api
npm install
```

### 3. Autenticarse en Cloudflare

```bash
wrangler login
```

### 4. Configurar el secret de la base de datos

```bash
wrangler secret put DATABASE_URL
# Pegar el connection string de Neon cuando lo pida
# Formato: postgresql://usuario:password@host/dbname?sslmode=require
```

### 5. Deploy

```bash
npm run deploy
```

## Desarrollo local

```bash
npm run dev
```

El Worker corre en `http://localhost:8787`. Para que funcione localmente necesitas un archivo `.dev.vars` en la raíz con:

```
DATABASE_URL=postgresql://usuario:password@host/dbname?sslmode=require
```

## Base de datos

El schema está en Neon. Las tablas se crearon desde el proyecto Python original con SQLAlchemy. Si necesitas recrearlas:

```sql
CREATE TABLE subjects (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    color VARCHAR(7) NOT NULL DEFAULT '#6c757d',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    subject_id INTEGER REFERENCES subjects(id) ON DELETE SET NULL,
    deadline DATE NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```
