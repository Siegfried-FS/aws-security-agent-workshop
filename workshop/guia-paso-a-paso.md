# Guía Paso a Paso — AWS Security Agent

> ⚠️ **1 de abril de 2026:** El servicio pasó de Preview a GA. Esta guía fue escrita durante el Preview. Verifica la [documentación oficial](https://docs.aws.amazon.com/securityagent/latest/userguide/what-is.html) para cambios.

---

## Requisitos previos

- Cuenta AWS con acceso a `us-east-1` (N. Virginia)
- IAM Identity Center habilitado (necesario para la web app)
- Cuenta de GitHub con un repo **privado** (para Code Review y Pen Testing)
- Dominio propio con acceso a Route 53 (para Pen Testing)
- Alerta de billing configurada (recomendado)

---

## Parte 1 — Primer acceso al servicio

### Paso 1: Acceder a AWS Security Agent

1. Inicia sesión en la consola AWS
2. Asegúrate de estar en la región **us-east-1 (N. Virginia)** — única región disponible en Preview
3. En la barra de búsqueda escribe `AWS Security Agent`
4. Haz click en el servicio

Verás la pantalla de bienvenida con el botón **"Configurar AWS Security Agent"**.

> Lo que ves en esta pantalla describe las 3 capacidades del servicio:
> - Revisión del diseño (Design Review)
> - Análisis de código en PRs (Code Review)
> - Pruebas de penetración bajo demanda (Pen Testing)

### Paso 2: Crear un Espacio de Agente

Un **Espacio de Agente** es el contenedor de trabajo para una aplicación o proyecto. Agrupa todos sus design reviews, repositorios conectados y configuraciones de pen testing.

1. Click en **"Crear espacio de agente"**
2. Completa los campos:
   - **Nombre del espacio:** un nombre descriptivo (ej. `virustotal-s3-scanner`)
   - **Descripción (opcional):** describe el proyecto (ej. `Escáner de archivos maliciosos con VirusTotal y S3`)
3. Click en **Crear**

> El nombre se muestra a los usuarios en la aplicación web. Máximo 255 caracteres.

### Paso 3: Revisar los Requisitos de Seguridad

Antes de hacer cualquier review, vale la pena entender con qué criterios evalúa el agente.

1. En el menú lateral click en **"Requisitos de seguridad"**
2. Verás 10 requisitos administrados por AWS, todos activados por defecto:

| Requisito | Descripción |
|-----------|-------------|
| Mejores prácticas para el registro de auditorías | El sistema debe admitir monitorización de seguridad |
| Mejores prácticas de autenticación | Solo usuarios legítimos pueden acceder |
| Mejores prácticas de autorización | Los sistemas siguen mejores prácticas de autorización |
| Mejores prácticas en protección de la información | Datos sensibles confidenciales e inalterados |
| Mejores prácticas para la protección de registros | Integridad y confidencialidad de los registros |
| Mejores prácticas para el acceso privilegiado | Medidas adecuadas para funciones privilegiadas |
| Mejores prácticas para la protección de secretos | Credenciales permanecen confidenciales |
| Mejores prácticas para la seguridad por defecto | Configuración predeterminada segura |
| Mejores prácticas para el aislamiento de inquilinos | Separación adecuada entre sistemas |
| Buenas prácticas de criptografía de confianza | Uso correcto de criptografía |

> Puedes personalizar estos requisitos o agregar los tuyos propios. Ver [requisitos-personalizados.md](./requisitos-personalizados.md).

### Paso 4: Abrir la Aplicación Web

El agente tiene dos interfaces:
- **Consola AWS** — para configuración (espacios, integraciones, requisitos)
- **Web App** — para el uso diario (crear reviews, ver resultados, ejecutar pen tests)

Para abrir la web app:
1. Ve a **"Espacios de agente"** → click en tu espacio
2. Click en **"Iniciar la aplicación web"** (esquina superior derecha)
3. Se abre en una pestaña nueva con URL del tipo `app-XXXX.securityagent.global.app.aws/...`

> La web app requiere autenticación via IAM Identity Center. Si no tienes usuario configurado, ve primero a la sección [Agregar usuarios](#agregar-usuarios-a-la-web-app).

---

## Parte 2 — Design Review

### Paso 5: Crear un Design Review

El Design Review analiza documentos de arquitectura contra los requisitos de seguridad habilitados.

**Formatos soportados:** DOC, DOCX, JPEG, MD, PDF, PNG, TXT  
**Límites:** máximo 5 archivos, 2 MB cada uno, 6 MB en total por review

1. En la web app, click en **"Design reviews"** → **"Create design review"**
2. Completa:
   - **Design review name:** nombre único (máximo 80 caracteres)
   - **Files to review:** arrastra o selecciona tus archivos de arquitectura
3. Click en **"Start design review"**

El review aparece con estado **"In Progress"**. Tarda aproximadamente 5-10 minutos.

### Paso 6: Interpretar los Resultados

Al completarse, el review muestra un resumen con 4 categorías:

| Estado | Significado |
|--------|-------------|
| 🔴 No cumple | El requisito de seguridad se incumple o no se aborda |
| 🟡 Datos insuficientes | El documento no contiene suficiente información para evaluar |
| 🟢 Obediente / Compliant | El requisito se cumple según los archivos cargados |
| ⚪ No aplicable | El requisito no aplica a este tipo de sistema |

**Ejemplo real de este workshop (3 fases del mismo proyecto):**

| Fase | No cumple | Datos insuficientes | Compliant | No aplicable |
|------|-----------|---------------------|-----------|--------------|
| Fase 1 — Propuesta inicial insegura | 9 | 0 | 0 | 1 |
| Fase 2 — Propuesta mejorada (sin agente) | 0 | 5 | 4 | 1 |
| Fase 3 — Aplicando recomendaciones del agente | 0 | 2 | 7 | 1 |

> La progresión de 9 "No cumple" → 0 "No cumple" demuestra el valor iterativo del Design Review. El agente no solo detecta problemas, sino que sugiere cómo resolverlos.

### Paso 7: Clonar y Reutilizar un Review

Si quieres hacer una segunda iteración sobre el mismo proyecto:
1. En la lista de reviews, selecciona el review anterior
2. Click en **"Clone design review"**
3. Cambia el nombre y sube los archivos actualizados

---

## Parte 3 — Code Review

> ⚠️ **Limitación importante:** Code Review **no funciona con repositorios públicos de GitHub**. El repo debe ser privado.

### Paso 8: Conectar GitHub

1. En la consola AWS, ve a tu espacio de agente
2. En el card **"Revisión del código"** → click **"Habilitar la revisión de código"**
3. En el modal **"Agregar integración"**:
   - Si es la primera vez: selecciona **"Crear nuevo registro"** → elige **GitHub**
   - Si ya tienes un registro: selecciona **"Registros disponibles"**

**Para crear un nuevo registro de GitHub:**

**Paso 1 — Instalar y autorizar la app:**
1. Click en **"Instalar y autorizar"**
2. Se abre GitHub — elige la cuenta donde instalar la app
3. Selecciona **"Only select repositories"** (recomendado) y elige el repo específico
4. Los permisos que solicita la app:
   - Read access a administración y metadata
   - Read/write access a código, issues, pull requests y advisories
5. Click **"Install & Authorize"**
6. GitHub puede pedir confirmación de identidad (2FA / GitHub Mobile)
7. Al regresar a AWS verás: ✅ "La autorización se realizó correctamente"

**Paso 2 — Registrar detalles:**
1. **Nombre del registro:** un nombre para identificarlo (ej. `github-tu-usuario`)
   - Caracteres válidos: A-Z, a-z, 0-9, puntos, guiones bajos y guiones
2. **Tipo de cuenta de GitHub:** selecciona `Usuario` o `Organización` según corresponda
3. Click **"Conectar"**

Verás el banner verde: ✅ "La integración de GitHub se ha conectado correctamente"

### Paso 9: Conectar el Repositorio

1. En el modal de integración, selecciona el registro recién creado → **"Siguiente"**
2. **Paso 1 — Conectar GitHub repositorios:**
   - Verás la lista de repos autorizados
   - Selecciona el repo que quieres analizar
   - Click **"Siguiente"**
3. **Paso 2 — Administrar capacidades:**
   - Verás el repo con columnas "Revisión del código" y "Remediación de pruebas de penetración"
   - Si el repo es **público**: ambas columnas muestran "No se admite" — debes hacerlo privado primero
   - Si el repo es **privado**: ambas capacidades se pueden activar con toggle
   - **Configuración de revisión de código** — elige el tipo de análisis:
     - `Validación de los requisitos de seguridad` — solo verifica tus requisitos personalizados
     - `Resultados de vulnerabilidades de seguridad` — solo detecta vulnerabilidades comunes
     - `Requisitos de seguridad y resultados de vulnerabilidades` — ambos (recomendado)
4. Click **"Conectar"**

> Si el repo aparece como público y ves "No se admite", ve a GitHub → Settings → Change repository visibility → Private. Al regresar y reconectar, las capacidades se activarán automáticamente.

### Paso 10: Probar el Code Review con un Pull Request

1. En tu repo de GitHub, crea una rama nueva y haz al menos un commit
2. Abre un Pull Request hacia `main`
3. En segundos, el bot `aws-security-agent` comentará en el PR:
   > "AWS Security Agent is reviewing your pull request and will post feedback shortly."
4. Espera el resultado — puede ser:
   - `No issues identified.` — sin hallazgos
   - Lista de findings con severidad y recomendaciones
   - Error interno del servicio (ocurrió en este workshop en el 3er commit)

> El agente también envía notificaciones por email para cada análisis.

#### ⚠️ Advertencia importante — No confíes ciegamente en el agente

Durante este workshop introdujimos vulnerabilidades conocidas de forma intencional:
- Un **hardcoded API key** directamente en el código fuente
- **Logging de datos sensibles** del body completo del request en CloudWatch

El agente respondió **"No issues identified"** en ambos casos.

**Conclusión:** AWS Security Agent es una capa adicional de revisión, no un reemplazo del criterio humano. Los resultados "limpios" no garantizan que el código sea seguro.

---

## Parte 4 — Penetration Testing

> ⚠️ **1 de abril de 2026:** Penetration Testing pasó de Preview a **GA**. Esta sección fue documentada durante el Preview.

### Paso 11: Habilitar Pen Testing — Configurar Dominio

1. En la consola AWS, ve a tu espacio de agente
2. En el card **"Pruebas de penetración"** → click **"Habilitar la prueba de penetración"**
3. **Paso 1 — Configurar dominio:**
   - Click **"Agregar dominio"**
   - **Dominio:** escribe tu dominio o subdominio (ej. `app.tudominio.com`)
     - AWS recomienda usar subdominios donde tengas permiso para crear registros TXT
   - **Método de verificación:** selecciona `Registro DNS TXT`
   - Click **"Siguiente"**

### Paso 12: Verificar Propiedad del Dominio

AWS requiere verificar que eres dueño del dominio antes de ejecutar pen testing.

1. **Paso 2 — Verificar dominios:**
   - El dominio aparece con estado **"Pendiente"**
   - Verás el **Token de verificación** y el **Nombre de registro DNS** que debes crear

2. **En Route 53 (o tu proveedor DNS):**
   - Ve a Route 53 → Hosted zones → abre la zona de tu dominio
   - Click **"Create record"**
   - Completa:
     - **Record name:** `_aws_securityagent-challenge.[tu-subdominio]` (solo el prefijo, sin el dominio raíz)
     - **Record type:** `TXT`
     - **Value:** el token de verificación (formato: `aws-securityagent-domain-verification=XXXXX`)
     - **TTL:** 300
   - Click **"Create records"**

3. **De regreso en AWS Security Agent:**
   - Selecciona el dominio → click **"Verificar"**
   - Espera la propagación DNS (puede tardar unos minutos)
   - El estado cambia a ✅ **"Verificado"**

> Si el dominio de Route 53 está en otra cuenta AWS, necesitas acceso a esa cuenta para agregar el registro. No hay forma de saltarse este paso — es el mecanismo de seguridad para evitar pen testing sobre dominios ajenos.

### Paso 13: Configurar Recursos y Acceso (Opcional)

**Paso 3 — Configurar recursos y acceso** ofrece opciones opcionales para mejorar el alcance del pen test:

| Recurso | Para qué sirve |
|---------|----------------|
| VPC | Define el alcance de red para la ejecución |
| Registros de CloudWatch | Captura comportamiento de la app durante el test |
| Secretos (Secrets Manager) | Proporciona credenciales al agente |
| Funciones de Lambda | Incluye funciones en el análisis |
| Buckets de S3 | Contexto adicional para el agente |
| Acceso al servicio | Rol IAM para el agente (se crea automáticamente si no se especifica) |

Para una prueba básica, puedes dejar todo vacío y continuar.

### Paso 14: Agregar Usuarios a la Web App

Para que los usuarios puedan acceder a la web app del agente:

1. En la consola, ve a tu espacio → click **"Agregar usuarios"**
2. Verás los usuarios de **IAM Identity Center** disponibles
3. Selecciona el usuario → click **"Agregar usuarios"**

> La web app usa IAM Identity Center para autenticación. Si no tienes usuarios configurados, primero créalos en IAM Identity Center.

### Paso 15: Crear y Ejecutar un Pen Test

1. En la **web app**, click en **"Penetration tests"** → **"Create penetration test"**

2. **Paso 1 — Penetration test details:**
   - **Pentest name:** nombre descriptivo
   - **Target URLs:** URL(s) a probar (solo dominios verificados)
   - **Exclude risk types (opcional):** tipos de riesgo a excluir
   - **Out-of-scope URLs (opcional):** URLs que NO deben atacarse (ej. `/admin`)
   - **Accessible URLs (opcional):** dominios con los que interactúa la app pero no deben atacarse
   - **Custom HTTP headers (opcional):** headers personalizados para las requests
   - **Service role:** rol IAM para la ejecución (se crea automáticamente si se deja vacío)
   - **CloudWatch log group (opcional):** dónde guardar los logs del pen test
   - **Automatic code remediation:** si se activa, genera PRs automáticos con fixes

3. **Paso 2 — VPC Resources (opcional):** configura VPC para el entorno de ejecución

4. **Paso 3 — Authentication Resources (opcional):** agrega credenciales si la app requiere login

5. **Paso 4 — Additional learning resources (opcional):**
   - Sube archivos, conecta repos de GitHub o links de S3
   - Proporcionar contexto mejora la cobertura y precisión del pen test

6. Click **"Create pentest"** (o **"Create and execute"** para ejecutar inmediatamente)

### Paso 16: Iniciar el Run

1. En el panel del pen test, click **"Start run"**
2. Confirma en el modal: "Please confirm you are ready to begin the test, as no further configuration changes can be made once it has started."
3. Click **"Start run"**

El run aparece con estado **"In progress"**. Puedes monitorear el avance con **"Monitor run"**.

---

## Agregar usuarios a la web app

Si es la primera vez que configuras el servicio:

1. En la consola AWS, ve a tu espacio de agente
2. En el card **"Revisión del diseño"** → click **"Agregar usuarios"**
3. Verás la instancia de IAM Identity Center vinculada
4. Click **"Agregar usuarios de Identity Center"**
5. Selecciona el usuario → **"Agregar usuarios"**

---

## Resumen de limitaciones encontradas

| Limitación | Detalle |
|------------|---------|
| Región única | Solo `us-east-1` durante Preview |
| Repos públicos no soportados | Code Review y Pen Testing requieren repo privado |
| Verificación de dominio obligatoria | Pen Testing requiere agregar registro TXT en DNS |
| Falsos negativos en Code Review | El agente no detectó hardcoded secrets ni sensitive logging |
| Errores internos del servicio | Ocurrió en el 3er commit del code review durante el workshop |
| IAM Identity Center requerido | La web app no funciona sin SSO configurado |

---

## Recursos

- [Documentación oficial](https://docs.aws.amazon.com/securityagent/latest/userguide/what-is.html)
- [Pricing](https://aws.amazon.com/security-agent/pricing/)
- [Blog de lanzamiento](https://aws.amazon.com/blogs/aws/new-aws-security-agent-secures-applications-proactively-from-design-to-deployment-preview/)
