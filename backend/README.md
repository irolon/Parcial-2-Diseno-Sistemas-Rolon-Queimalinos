# FitTrainer — Backend

API REST (monolito modular) en **Node.js + Express**, datos en **Supabase/PostgreSQL**.

## Patrón por módulo: Controller → Service → Repository → Model

| Capa | Archivo | Responsabilidad |
|------|---------|-----------------|
| Controller | `*.controller.js` | Recibe el request, llama al service, responde. Nada más. |
| Service | `*.service.js` | Lógica de negocio, permisos, orquestación. |
| Repository | `*.repository.js` | Queries a Supabase. Única capa que toca la DB. |
| Model | `*.model.js` | Entidad de dominio: estructura + mapeo de filas. Sin lógica. |
| (Routes) | `*.routes.js` | Define rutas y enchufa middlewares (validación, auth, rol). |
| (Validation) | `*.validation.js` | Validadores de input del módulo. |

## Estructura

Organización **por capa**, y dentro de cada capa una **carpeta por módulo**.

```
src/
├── config/env.js            # carga + validación de variables de entorno
├── shared/                  # transversal a todos los módulos
│   ├── supabase.js          # cliente Supabase (service role)
│   ├── middlewares/         # authenticate, requireRole, validate, errores
│   └── utils/               # errors, password (bcrypt), jwt, asyncHandler
├── controllers/
│   ├── auth/auth.controller.js
│   └── health/health.controller.js
├── services/
│   └── auth/auth.service.js          (+ auth.service.test.js)
├── repositories/
│   └── auth/auth.repository.js
├── models/
│   └── auth/trainer.model.js         (+ trainer.model.test.js)
│       auth/refresh-token.model.js
├── validations/
│   └── auth/auth.validation.js
├── routes/
│   ├── auth/auth.routes.js
│   └── health/health.routes.js
├── database/schema.sql      # DDL (Fase 1: trainers + refresh_tokens)
├── app.js                   # arma la app de Express
└── server.js                # entrypoint (levanta el server)

tests/                       # TODOS los tests viven acá (fuera de src/)
├── unit/                    # unitarios — espejan la estructura de src/
│   ├── services/auth/auth.service.test.js
│   └── models/auth/trainer.model.test.js
└── integration/             # end-to-end de endpoints HTTP (Supertest)
```

> Cada módulo nuevo (invitations, exercises, routines…) agrega su carpeta
> `<modulo>/` dentro de cada capa que necesite: `controllers/<modulo>/`,
> `services/<modulo>/`, `repositories/<modulo>/`, `models/<modulo>/`,
> `routes/<modulo>/`, `validations/<modulo>/`.

## Setup

```bash
cp .env.example .env     # completar con credenciales de Supabase + JWT_SECRET
npm install
```

Luego ejecutar `src/database/schema.sql` en el **SQL Editor** de Supabase.

## Scripts

| Comando | Descripción |
|---------|-------------|
| `npm run dev` | Dev con nodemon (hot reload) |
| `npm start` | Producción |
| `npm test` | Tests (Jest) |
| `npm run test:coverage` | Tests con cobertura |
| `npm run lint` | ESLint |

## Endpoints (Fase 1)

| Método | Ruta | Auth | Descripción |
|--------|------|------|-------------|
| GET | `/health` | — | Liveness |
| GET | `/health/db` | — | Conectividad con Supabase |
| POST | `/api/auth/register` | — | Registra un trainer |
| POST | `/api/auth/login` | — | Login (email + password) |
| POST | `/api/auth/refresh` | — | Nuevo access token |
| POST | `/api/auth/logout` | Bearer | Revoca el refresh token |

### Autenticación

- **Access token:** JWT firmado con `JWT_SECRET` (`{ sub, role, email }`), expira según `JWT_EXPIRES_IN`.
- **Refresh token:** token opaco; en la DB se guarda solo su hash SHA-256. El logout lo revoca.

### Ejemplo

```bash
curl -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Ignacio","email":"ignacio@fittrainer.app","password":"Password1"}'
```

## Notas de implementación

Respecto del DER documentado en Notion, esta implementación agrega dos cosas que
la autenticación necesita y el modelo original no contemplaba:

1. **`trainers.password_hash`** — el DER no incluía credenciales.
2. **Tabla `refresh_tokens`** — para poder revocar tokens en el logout.

Ambas están reflejadas en `src/database/schema.sql`.
