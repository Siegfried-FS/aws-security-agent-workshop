# Arquitectura: Validación de Archivos con VirusTotal y S3

## Descripción General

Sistema serverless en AWS que valida automáticamente archivos subidos por usuarios contra la API de VirusTotal. Los archivos son clasificados y movidos a buckets separados según el resultado del análisis, y se envían alertas en caso de detección de amenazas.

---

## Componentes

### Almacenamiento (Amazon S3)

| Bucket | Propósito |
|--------|-----------|
| `virusscan-inbox` | Recibe los archivos subidos por usuarios |
| `virusscan-clean` | Archivos validados como seguros |
| `virusscan-quarantine` | Archivos detectados como maliciosos |

### Procesamiento (AWS Lambda)

- **`fn-scan-file`**: Se dispara con cada nuevo objeto en `virusscan-inbox`. Envía el archivo a VirusTotal y evalúa el resultado.
- **`fn-notify-alert`**: Envía notificaciones cuando un archivo es puesto en cuarentena.

### Notificaciones (Amazon SNS)

- Topic `virus-alerts`: Notifica al equipo de seguridad cuando se detecta un archivo malicioso.

### Secretos (AWS Secrets Manager)

- Almacena la API Key de VirusTotal de forma segura.

### Monitoreo (Amazon CloudWatch)

- Logs de cada ejecución Lambda
- Métricas: archivos escaneados, archivos en cuarentena, errores de API
- Alarmas: tasa de error > 5%, tiempo de respuesta de VirusTotal > 30s

---

## Flujo de Datos

```
Usuario
  │
  ▼
S3: virusscan-inbox  ──── S3 Event Notification
  │
  ▼
Lambda: fn-scan-file
  │
  ├── Obtiene API Key desde Secrets Manager
  ├── Descarga el archivo desde S3
  ├── Envía hash (SHA256) a VirusTotal API v3
  │     └── Si el archivo es nuevo: sube el archivo completo
  │
  ├── Resultado: LIMPIO
  │     └── Mueve archivo a S3: virusscan-clean
  │
  └── Resultado: MALICIOSO (≥ 3 engines detectan amenaza)
        ├── Mueve archivo a S3: virusscan-quarantine
        ├── Agrega metadata: motores que detectaron, nombre de amenaza, fecha
        └── Publica mensaje en SNS: virus-alerts
              └── Lambda: fn-notify-alert → Email al equipo de seguridad
```

---

## Decisiones de Diseño

### Umbral de detección
Se considera malicioso si **3 o más motores antivirus** de VirusTotal reportan una amenaza. Esto reduce falsos positivos manteniendo sensibilidad razonable.

### Análisis por hash primero
Antes de subir el archivo completo a VirusTotal, se consulta por SHA256. Si ya existe un análisis reciente (< 24h), se usa ese resultado. Esto reduce costos de API y latencia.

### Tamaño máximo de archivo
- Límite: **32MB** (límite de VirusTotal API gratuita)
- Archivos mayores: se rechaza el upload con mensaje de error al usuario

### Retención en cuarentena
- Archivos en `virusscan-quarantine` tienen lifecycle policy de **90 días**
- Después se eliminan automáticamente

---

## Seguridad

- Los 3 buckets S3 tienen **acceso público bloqueado**
- Comunicación entre servicios via **IAM roles con mínimo privilegio**
- API Key de VirusTotal almacenada en **Secrets Manager**, nunca en variables de entorno o código
- Logs de acceso habilitados en los 3 buckets
- Cifrado en reposo: **SSE-S3** en todos los buckets
- Cifrado en tránsito: HTTPS en todas las llamadas

### Permisos IAM por función

**Principio:** mínimo privilegio. Ningún componente tiene permisos de administrador. No se crean usuarios IAM — todo usa roles asumidos por los servicios.

#### Rol: `role-fn-scan-file` (asumido por Lambda fn-scan-file)

```json
{
  "s3:GetObject": "arn:aws:s3:::virusscan-inbox/*",
  "s3:DeleteObject": "arn:aws:s3:::virusscan-inbox/*",
  "s3:PutObject": "arn:aws:s3:::virusscan-clean/*",
  "s3:PutObject": "arn:aws:s3:::virusscan-quarantine/*",
  "secretsmanager:GetSecretValue": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:virustotal-api-key",
  "sns:Publish": "arn:aws:sns:us-east-1:ACCOUNT_ID:virus-alerts",
  "logs:CreateLogGroup": "*",
  "logs:CreateLogStream": "*",
  "logs:PutLogEvents": "*"
}
```

#### Rol: `role-fn-notify-alert` (asumido por Lambda fn-notify-alert)

```json
{
  "logs:CreateLogGroup": "*",
  "logs:CreateLogStream": "*",
  "logs:PutLogEvents": "*"
}
```
> Solo necesita escribir logs. SNS lo invoca directamente, no necesita permisos adicionales.

#### Bucket Policies (S3)

Cada bucket solo acepta operaciones desde los roles específicos:

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Condition": {
    "StringNotLike": {
      "aws:PrincipalArn": "arn:aws:iam::ACCOUNT_ID:role/role-fn-scan-file"
    }
  }
}
```

#### Lo que NO existe en este sistema
- No hay usuarios IAM con access keys
- No hay roles con `AdministratorAccess` o `*` en actions
- No hay credenciales en código ni variables de entorno
- No hay acceso público a ningún bucket

---

## Consideraciones Operativas

### Manejo de errores
- Si VirusTotal API no responde: reintentos con backoff exponencial (3 intentos)
- Si falla después de 3 intentos: archivo queda en `inbox`, se genera alarma en CloudWatch
- Dead Letter Queue (SQS) para eventos Lambda fallidos

### Costos estimados (uso moderado ~1000 archivos/día)
- Lambda: prácticamente gratis (free tier)
- S3: < $1/mes
- VirusTotal API: plan gratuito permite 500 requests/día; plan de pago para mayor volumen
- SNS: < $1/mes

---

## Limitaciones Conocidas

- No escanea archivos > 32MB
- Depende de disponibilidad de VirusTotal API (SLA externo)
- El análisis no es en tiempo real para archivos nuevos no conocidos por VirusTotal (puede tardar minutos)
- No hay interfaz de usuario — solo API/S3 directo

---

## Stack Tecnológico

- **Runtime Lambda**: Python 3.12
- **IaC**: AWS CDK (TypeScript)
- **VirusTotal API**: v3
- **Región**: us-east-1
