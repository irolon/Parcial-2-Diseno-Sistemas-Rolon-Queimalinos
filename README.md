# FitTrainer

Plataforma para personal trainers y sus alumnos · **Web + Android** · Proyecto Final de Analista de Sistemas.

Monorepo con tres clientes/servicios:

| Carpeta | Qué es | Stack |
|---------|--------|-------|
| `backend/` | API REST (monolito modular) | Node.js + Express + Supabase/PostgreSQL |
| `frontend/` | Dashboard web (ambos roles) | React + Vite + Tailwind *(pendiente)* |
| `mobile/` | App Android nativa (ambos roles) | Kotlin + Jetpack Compose *(pendiente)* |

La documentación completa (arquitectura, contratos de API, modelo de datos, plan de
desarrollo) vive en el tablero de Notion del proyecto.

## Estado

- [x] **Fase 1 — Backend:** scaffold + módulo **Auth** (register / login / refresh / logout) + middleware JWT.
- [ ] Fase 1 — Invitaciones (QR) + onboarding de alumnos.
- [ ] Fase 2 — Rutinas y biblioteca de ejercicios.
- [ ] Fase 3 — Sesiones, tracking y progreso.

## Arranque rápido (backend)

```bash
cd backend
cp .env.example .env   # completar con credenciales de Supabase
npm install
npm run dev            # API en http://localhost:3001
```

Ver `backend/README.md` para el detalle.
