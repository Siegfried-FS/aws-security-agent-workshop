# Arquitectura: File Upload Service — Fase 3

> Esta propuesta incorpora las correcciones recomendadas por AWS Security Agent en el informe de Fase 1.
> Cada sección indica qué finding resuelve.

---

## Descripción

Sistema serverless que valida archivos subidos por usuarios autenticados contra VirusTotal. Los archivos se clasifican automáticamente y se generan alertas ante amenazas detectadas.

---

## Cambios respecto a Fase 1

| Finding | Problema original | Corrección aplicada |
|---------|-------------------|---------------------|
| Authentication | Cualquiera podía subir sin autenticarse | Amazon Cognito con User Pool |
| Audit Logging | Sin logs en S3, Lambda ni CloudTrail | S3 access logs + CloudWatch + CloudTrail habilitados |
| Information Protection | Secrets en texto plano, S3 público sin cifrado | Secrets Manager + SSE-S3 + buckets privados |
| Authorization | Lambda con permisos de administrador | Rol con mínimo privilegio |
| Secure by Default | Configuración insegura por defecto | Todo privado y cifrado por defecto |
| Secret Protection | API keys hardcodeadas en variables de entorno | AWS Secrets Manager con rotación automática |
| Log Protection | Sin redacción de datos sensibles en logs | Logs con retención, acceso restringido y redacción |
| Privileged Access | Rol Lambda con AdministratorAccess | Rol con permisos específicos por recurso |
| Trusted Cryptography | Cifrado deshabilitado | SSE-S3 en todos los buckets, Secrets Manager para keys |

---

## Componentes

### Autenticación (Amazon Cognito)
- User Pool para gestión de usuarios
- Los uploads requieren token JWT válido
- Rate limiting por usuario para prevenir abuso

### Almacenamiento (Amazon S3)
- Todos los buckets: **acceso público bloqueado**, **SSE-S3 habilitado**, **access logging habilitado**

| Bucket | Propósito |
|--------|-----------|
| `virusscan-inbox` | Recibe archivos de usuarios autenticados |
| `virusscan-clean` | Archivos validados como seguros |
| `virusscan-quarantine` | Archivos detectados como maliciosos |
| `virusscan-logs` | Access logs de los 3 buckets anteriores |

### Procesamiento (AWS Lambda)

- **`fn-scan-file`**: Disparada por S3 event en inbox. Obtiene API key de Secrets Manager, escanea con VirusTotal, mueve el archivo al bucket correspondiente.

### Secretos (AWS Secrets Manager)
- API key de VirusTotal almacenada con **rotación automática**
- Sin credenciales en variables de entorno ni en código

### Monitoreo (CloudWatch + CloudTrail)
- CloudTrail habilitado para todas las llamadas a la API
- CloudWatch Logs para cada ejecución Lambda
- Logs con retención de 90 días y acceso restringido
- Redacción automática de API keys y credenciales en logs

---

## Flujo de Datos

```
Usuario autenticado (Cognito JWT)
  │
  ▼
S3: virusscan-inbox (privado, SSE-S3)
  │
  └── S3 Event → Lambda: fn-scan-file
                    │
                    ├── Obtiene API key → Secrets Manager
                    ├── Escanea con VirusTotal API
                    │
                    ├── LIMPIO → S3: virusscan-clean
                    └── MALICIOSO → S3: virusscan-quarantine + SNS alerta
```

---

## IAM — Mínimo Privilegio

### Rol `role-fn-scan-file`

```json
{
  "s3:GetObject":    "arn:aws:s3:::virusscan-inbox/*",
  "s3:DeleteObject": "arn:aws:s3:::virusscan-inbox/*",
  "s3:PutObject":    "arn:aws:s3:::virusscan-clean/*",
  "s3:PutObject":    "arn:aws:s3:::virusscan-quarantine/*",
  "secretsmanager:GetSecretValue": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:virustotal-api-key",
  "sns:Publish":     "arn:aws:sns:us-east-1:ACCOUNT_ID:virus-alerts",
  "logs:CreateLogGroup":  "*",
  "logs:CreateLogStream": "*",
  "logs:PutLogEvents":    "*"
}
```

- Sin `AdministratorAccess`
- Sin AWS credentials en variables de entorno
- Sin acceso a recursos fuera del scope del sistema

---

## Seguridad por Defecto

- Buckets S3: privados, cifrados, con access logging desde el primer deploy
- Lambda: rol con mínimo privilegio desde el primer deploy
- Autenticación: requerida antes de cualquier operación
- Secrets: en Secrets Manager, nunca en código ni variables de entorno
