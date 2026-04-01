# Proyecto 02: Code Review + Penetration Testing

Demostración de Code Review y Pen Testing de AWS Security Agent usando un repositorio real de GitHub y una aplicación web desplegada.

> El objetivo no es analizar el proyecto en sí, sino mostrar el flujo completo de cada feature — incluyendo los errores y limitaciones reales encontrados.

---

## Parte A — Code Review

### Flujo completo

```
Crear espacio → Conectar GitHub → Autorizar repo → Registrar integración
→ Conectar repo (debe ser privado) → Abrir PR → Agente revisa automáticamente
```

### Paso a paso documentado

#### 1. Crear el espacio de agente para code review

- Ve a **Espacios de agente** → **Crear espacio de agente**
- Nombre: `code-review-demo`
- Al crear, aparece el modal **"Agregar integración"** automáticamente
- Si no hay registros previos, la lista aparece vacía — selecciona **"Crear nuevo registro"**

#### 2. Seleccionar GitHub como integración

- En el modal, selecciona **"Crear nuevo registro"**
- Elige **GitHub** (Repositorio de origen, Seguimiento de problemas)
- Click **"Siguiente"**

#### 3. Instalar y autorizar la app de GitHub

El formulario tiene 2 pasos:

**Paso 1 — Instalar y autorizar:**
- Click en **"Instalar y autorizar"**
- Se abre GitHub con la pantalla "Install & Authorize AWS Security Agent"
- Selecciona **"Only select repositories"** y elige el repo específico
- Permisos que solicita:
  - Read access a administración y metadata
  - Read/write access a código, issues, pull requests y advisories
- Click **"Install & Authorize"**
- GitHub puede pedir confirmación de identidad (2FA)
- Al regresar: ✅ "La autorización se realizó correctamente"

**Paso 2 — Registrar detalles:**
- **Nombre del registro:** ej. `github-tu-usuario`
- **Tipo de cuenta de GitHub:** `Usuario` (si el repo está bajo tu cuenta personal) u `Organización`
- Click **"Conectar"**

Banner verde: ✅ "La integración de GitHub se ha conectado correctamente"

#### 4. Conectar el repositorio al espacio

- En el panel del espacio → **"Habilitar la revisión de código"**
- En el modal, selecciona el registro recién creado → **"Siguiente"**
- **Paso 1:** selecciona el repo de la lista
- **Paso 2 — Administrar capacidades:**

> ⚠️ **Problema encontrado aquí:** Si el repo es público, las columnas "Revisión del código" y "Remediación de pruebas de penetración" muestran **"No se admite"** y no se pueden activar.

**Solución:** cambiar el repo a privado en GitHub (Settings → Change repository visibility → Private). Al reconectar, ambas capacidades se activan automáticamente con toggle.

- Selecciona el tipo de revisión: **"Requisitos de seguridad y resultados de vulnerabilidades"** (recomendado)
- Click **"Conectar"**

Banner verde: ✅ "Recursos de integración agregados"

El panel del espacio ahora muestra **Code Review: ✅ Listo**

#### 5. Crear un Pull Request y ver el análisis

1. En tu repo de GitHub, crea una rama (ej. `demo/security-review`)
2. Haz al menos un commit con cambios de código
3. Abre un PR hacia `main`
4. En segundos, el bot `aws-security-agent` comenta en el PR:
   > "AWS Security Agent is reviewing your pull request and will post feedback shortly."
5. Espera el resultado

---

### Resultados reales del workshop

Se hicieron 3 commits en el PR para probar la detección:

**Commit 1 — Solo README:**
- Resultado: `No issues identified.`
- Esperado ✅

**Commit 2 — Lambda con path traversal sutil:**
- Vulnerabilidad: `filename` del body usado directamente como `data/${filename}` sin `path.basename()`
- Resultado: `No issues identified.`
- ❌ El agente no detectó el path traversal (cambio demasiado sutil en el diff)

**Commit 3 — Hardcoded secret + sensitive logging:**
- Vulnerabilidades introducidas:
  - `ADMIN_API_KEY = 'sk-admin-2024-xK9mP2qL'` hardcodeado en el código
  - `console.log(JSON.stringify(requestBody))` exponiendo datos del usuario en CloudWatch
- Resultado: Error interno del servicio
  > "AWS Security Agent was unable to complete this review due to an internal service error. We apologize for the inconvenience and will seek to improve this in the future."
- ❌ El agente no llegó a analizar el commit

### Notificaciones por email

El agente envía un email por cada análisis del PR (inicio de revisión + resultado). Si tienes muchos commits en un PR activo, espera varios emails.

---

### ⚠️ Advertencia — El agente no detectó vulnerabilidades obvias

Introdujimos intencionalmente:
- Un **hardcoded API key** directamente en el código fuente
- **Logging de datos sensibles** del body completo del request

El agente respondió `No issues identified` en el commit con el path traversal, y tuvo un error interno en el commit con el hardcoded secret.

**Conclusión:** AWS Security Agent es una capa adicional de revisión, no un reemplazo del criterio humano ni de herramientas especializadas como Semgrep, Snyk o Checkov.

---

## Parte B — Penetration Testing

> ⚠️ **1 de abril de 2026:** Penetration Testing pasó de Preview a **GA**. Esta sección fue documentada durante el Preview.

### Flujo completo

```
Habilitar pen testing → Configurar dominio → Verificar propiedad DNS (Route 53)
→ Configurar recursos (opcional) → Agregar usuario a web app
→ Crear pen test en web app → Ejecutar run
```

### Paso a paso documentado

#### 1. Habilitar pen testing — Configurar dominio

- En el panel del espacio → **"Habilitar la prueba de penetración"**
- **Paso 1 — Configurar dominio:**
  - Click **"Agregar dominio"**
  - Escribe tu dominio o subdominio (ej. `app.tudominio.com`)
  - Método de verificación: **Registro DNS TXT**
  - Click **"Siguiente"**

#### 2. Verificar propiedad del dominio

- **Paso 2 — Verificar dominios:**
  - El dominio aparece con estado **"Pendiente"**
  - Copia el **Token de verificación** y el **Nombre de registro DNS**

- **En Route 53:**
  - Hosted zones → abre la zona de tu dominio
  - **Create record** con:
    - Record name: `_aws_securityagent-challenge.[subdominio]`
    - Record type: `TXT`
    - Value: `aws-securityagent-domain-verification=XXXXX` (el token)
    - TTL: 300
  - Click **"Create records"**

- **De regreso en AWS Security Agent:**
  - Selecciona el dominio → click **"Verificar"**
  - Estado cambia a ✅ **"Verificado"**

> Si Route 53 está en otra cuenta AWS, necesitas acceso a esa cuenta. No hay forma de saltarse este paso.

#### 3. Configurar recursos y acceso (Paso 3 — opcional)

Opciones disponibles (todas opcionales para una prueba básica):
- VPC y subredes
- Registros de CloudWatch
- Secretos (Secrets Manager)
- Funciones de Lambda
- Buckets de S3
- Acceso al servicio (rol IAM — se crea automáticamente si se omite)
- Repositorios conectados

#### 4. Agregar usuario a la web app

- En el panel del espacio → **"Agregar usuarios"**
- Selecciona usuarios de IAM Identity Center
- Click **"Agregar usuarios"**

#### 5. Crear el pen test en la web app

En la **web app** (no en la consola):

- **Penetration tests** → **"Create penetration test"**
- **Paso 1 — Detalles:**
  - Pentest name
  - Target URLs (solo dominios verificados)
  - Exclude risk types (opcional)
  - Out-of-scope URLs (opcional) — ej. `/admin`
  - Accessible URLs (opcional)
  - Custom HTTP headers (opcional)
  - Service role (se crea automáticamente si se deja vacío)
  - CloudWatch log group (opcional)
  - Automatic code remediation (genera PRs automáticos con fixes)
- **Paso 2 — VPC Resources** (opcional)
- **Paso 3 — Authentication Resources** (opcional — para apps con login)
- **Paso 4 — Additional learning resources** (opcional — mejora cobertura)
- Click **"Create pentest"**

#### 6. Ejecutar el run

- En el panel del pen test → **"Start run"**
- Confirma en el modal
- El run aparece con estado **"In progress"**
- Monitorea con **"Monitor run"**

---

## Estado del workshop

| Feature | Estado | Notas |
|---------|--------|-------|
| Code Review — Configuración | ✅ | Requirió cambiar repo a privado |
| Code Review — Primer commit | ✅ | No issues identified (esperado) |
| Code Review — Path traversal | ✅ | No detectado por el agente |
| Code Review — Hardcoded secret | ❌ | Error interno del servicio |
| Pen Testing — Verificación DNS | ✅ | Requirió acceso a Route 53 |
| Pen Testing — Run iniciado | ✅ | In progress al momento del workshop |
