# bonos-service

Microservicio de bonos y promociones del casino VidalCasino 2.0. API REST en Python/FastAPI. Permite consultar el catálogo de bonos, reclamar bonos y ver el historial de bonos reclamados.

## Stack

- Python 3.12 + FastAPI + Uvicorn
- PostgreSQL 16 (compartida con casino-backend)
- JWT validación (HS256) — el token lo emite casino-backend
- Docker + Amazon ECR
- Kubernetes (Amazon EKS)

## Puerto

| Entorno | Puerto |
|---------|--------|
| Local | 8004 |
| Clúster | ClusterIP :8004 |

## Endpoints

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/bonos` | Catálogo de bonos activos |
| GET | `/api/bonos/mis-bonos` | Bonos reclamados por el usuario |
| POST | `/api/bonos/{codigo}/reclamar` | Reclamar un bono (acredita saldo) |
| GET | `/livez` | Liveness probe — 200 si el proceso está vivo |
| GET | `/readyz` | Readiness probe — 200 BD ok / 503 BD caída |
| GET | `/docs` | Documentación Swagger automática |

## Variables de entorno

| Variable | Descripción |
|----------|-------------|
| `JWT_SECRET` | Secreto para validar tokens JWT |
| `DB_HOST` | Host de PostgreSQL |
| `DB_PORT` | Puerto de PostgreSQL (default: 5432) |
| `DB_USER` | Usuario de PostgreSQL |
| `DB_PASSWORD` | Contraseña de PostgreSQL |
| `DB_NAME` | Nombre de la base de datos |
| `CORS_ORIGIN` | Origen permitido para CORS |

## Construcción local

```bash
docker build -t bonos-service .
docker run -p 8004:8004 --env-file .env bonos-service
```

## Despliegue en EKS

El pipeline CI/CD se dispara automáticamente con push a la rama `deploy`:

```bash
git checkout deploy
git merge dev
git push origin deploy
```

El pipeline hace:
1. Build de la imagen Docker
2. Push a Amazon ECR con tags `latest`, `v1.0.0` y `$GITHUB_SHA`
3. Deploy en EKS con `kubectl apply`

## Comandos útiles

```bash
# Ver pods
kubectl get pods | grep bonos

# Ver logs
kubectl logs deployment/bonos-service

# Ver HPA
kubectl get hpa bonos-service-hpa

# Verificar sondas
curl http://localhost:8004/livez
curl http://localhost:8004/readyz
```

## Troubleshooting

| Problema | Solución |
|----------|----------|
| CrashLoopBackOff | `kubectl logs deployment/bonos-service` |
| BD no conecta | Verificar secret `casino-secrets` y pod de postgres |
| Pipeline falla | Actualizar GitHub Secrets con nuevas credenciales del Learner Lab |

## Ramas

| Rama | Uso |
|------|-----|
| `main` | Referencia estable |
| `dev` | Desarrollo diario |
| `deploy` | Dispara el pipeline CI/CD |