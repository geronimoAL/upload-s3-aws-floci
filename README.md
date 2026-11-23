# CloudDocs API

Backend NestJS para subir, consultar, descargar y eliminar documentos PDF. Los metadatos se guardan en memoria; los archivos se almacenan en S3 compatible, incluyendo Floci en desarrollo.

## Requisitos

- Node.js y pnpm.
- Floci (o un endpoint compatible con la API de S3) disponible en `http://localhost:4566`.

## Configuración

```powershell
Copy-Item .env.example .env
pnpm install
```

En `.env`, configura `AWS_ENDPOINT`, región, bucket y credenciales. Para AWS real, elimina `AWS_ENDPOINT`, configura credenciales IAM mediante el proveedor estándar de AWS y deja `AWS_S3_AUTO_CREATE_BUCKET=false`. En desarrollo local el ejemplo activa la creación automática del bucket; al iniciar, el backend lo verifica y lo crea si falta.

Variables principales:

| Variable                                     | Descripción                                    | Valor de ejemplo        |
| -------------------------------------------- | ---------------------------------------------- | ----------------------- |
| `PORT`                                       | Puerto HTTP                                    | `3001`                  |
| `CORS_ORIGIN`                                | Origen autorizado del frontend                 | `http://localhost:3000` |
| `AWS_REGION`                                 | Región del bucket                              | `us-east-1`             |
| `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | Credenciales leídas del entorno                | `test` (local)          |
| `AWS_S3_BUCKET`                              | Bucket usado por la aplicación                 | `clouddocs`             |
| `AWS_ENDPOINT`                               | Endpoint del simulador; vacío para AWS         | `http://localhost:4566` |
| `AWS_FORCE_PATH_STYLE`                       | Direccionamiento compatible con simuladores    | `true`                  |
| `AWS_S3_AUTO_CREATE_BUCKET`                  | Permite crear el bucket faltante al iniciar    | `true` (local)          |
| `AWS_MAX_FILE_SIZE`                          | Límite de subida en bytes                      | `10485760`              |
| `AWS_PRESIGNED_URL_EXPIRES_IN`               | Vigencia de URL firmada en segundos (1–604800) | `900`                   |

No se debe versionar `.env`. Para la creación automática, la identidad AWS necesita permiso `s3:CreateBucket`; si se desactiva, el bucket debe aprovisionarse previamente.

## Iniciar Floci y la API

Inicia Floci con su CLI oficial:

```powershell
floci start
```

Como alternativa, inicia la imagen Docker oficial en primer plano:

```powershell
docker run --rm --name floci -p 4566:4566 floci/floci:latest
```

Floci expone los servicios AWS en `http://localhost:4566`. Antes de probar las operaciones, confirma que el servicio responda en ese endpoint; si usa otro puerto, actualiza `AWS_ENDPOINT`. Detén el proceso de Floci con `Ctrl+C` (Docker) o el comando de parada de su CLI.

```powershell
pnpm start:dev
```

El backend verifica el bucket en el arranque. Con `AWS_S3_AUTO_CREATE_BUCKET=true`, lo crea cuando no existe; los fallos de conexión o permisos impiden el inicio y no exponen credenciales.

Swagger: `http://localhost:3001/api`

## Endpoints

`POST /documentos` recibe `multipart/form-data`: campo obligatorio `file` (PDF) y campo opcional `description`.

| Método   | Ruta                        | Función                                |
| -------- | --------------------------- | -------------------------------------- |
| `POST`   | `/documentos`               | Valida, sube y registra un PDF         |
| `GET`    | `/documentos`               | Lista metadatos registrados en memoria |
| `GET`    | `/documentos/:id`           | Obtiene un documento por UUID          |
| `GET`    | `/documentos/:id/descargar` | Descarga el archivo desde S3           |
| `GET`    | `/documentos/:id/url`       | Genera una URL de descarga temporal    |
| `DELETE` | `/documentos/:id`           | Elimina archivo y metadatos            |

La subida acepta únicamente archivos con extensión `.pdf`, MIME `application/pdf` y firma PDF, no vacíos y por debajo del límite configurado. El nombre original nunca es por sí solo la clave S3; la clave incluye un UUID. Los metadatos son temporales y se pierden al reiniciar el proceso.

## Comandos

```powershell
pnpm generate documentos
pnpm generate almacenamiento
pnpm build
pnpm test
```

El generador sobrescribe los archivos si el módulo ya existe; los dos comandos `generate` son solo para crear módulos nuevos.
