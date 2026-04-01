# Proyecto 02: Code Review — AWS User Group Playa Vicente

Demostración de **Code Review** de AWS Security Agent conectando un repositorio real de GitHub.

> El objetivo no es analizar el proyecto en sí, sino mostrar el flujo completo de la herramienta.

---

## Flujo de configuración

```
Crear espacio de agente → Conectar GitHub → Autorizar repo → Registrar integración → Code Review
```

---

## Pasos documentados

| # | Imagen | Descripción |
|---|--------|-------------|
| 01 | [ver](./imagenes/01-lista-espacios-agente.png) | Lista de espacios — se crea un segundo espacio para code review |
| 02 | [ver](./imagenes/02-crear-espacio-code-review-demo.png) | Formulario de creación con nombre `code-review-demo` |
| 03 | [ver](./imagenes/03-modal-agregar-integracion-sin-registros.png) | Modal "Agregar integración" — sin registros previos, se elige "Crear nuevo registro" |
| 04 | [ver](./imagenes/04-seleccionar-github-nuevo-registro.png) | Se selecciona GitHub como tipo de integración |
| 05 | [ver](./imagenes/05-formulario-registro-github-vacio.png) | Formulario de registro vacío — Paso 1: instalar app, Paso 2: nombre y tipo de cuenta |
| 06 | [ver](./imagenes/06-github-autorizar-app-repo-especifico.png) | En GitHub: se autoriza la app de AWS Security Agent solo para el repo seleccionado |
| 07 | [ver](./imagenes/07-github-confirmar-identidad-2fa.png) | GitHub solicita confirmación de identidad (2FA / GitHub Mobile) |
| 08 | [ver](./imagenes/08-formulario-registro-github-autorizado.png) | Regreso a AWS — Paso 1 muestra "autorización correcta", se completa nombre del registro |
| 09 | [ver](./imagenes/09-registro-nombre-github-siegfried-fs.png) | Registro nombrado `github-siegfried-fs`, tipo de cuenta: Usuario |
| 10 | [ver](./imagenes/10-integracion-github-conectada-exitosamente.png) | Confirmación: "La integración de GitHub se ha conectado correctamente" |
| 11 | [ver](./imagenes/11-code-review-necesita-configuracion.png) | Panel del espacio — Code Review muestra "Necesita configuración" |
| 12 | [ver](./imagenes/12-modal-registros-disponibles-github-siegfried-fs.png) | Modal de integración — ahora muestra el registro `github-siegfried-fs` disponible |
| 13 | [ver](./imagenes/13-conectar-repo-tiburon-seleccionado.png) | Paso 1: selección del repo Tiburon (Siegfried-FS, público) |
| 14 | [ver](./imagenes/14-administrar-capacidades-tipo-revision.png) | Paso 2: configurar tipo de revisión — se elige "Requisitos de seguridad y vulnerabilidades" |
| 15 | [ver](./imagenes/15-confirmacion-recursos-integracion-agregados.png) | Confirmación: "Recursos de integración agregados" |
| 16 | [ver](./imagenes/16-panel-code-review-sin-repositorios.png) | Panel del espacio — Code Review sigue en "Necesita configuración", aún sin repos conectados |
| 17 | [ver](./imagenes/17-error-repo-publico-no-admitido.png) | Error: repo Tiburon público — "No se admite" en Code Review y Pen Testing |
| 18 | [ver](./imagenes/18-repo-privado-code-review-activado.png) | Solución: repo cambiado a privado — ambas capacidades se activan automáticamente |
| 19 | [ver](./imagenes/19-code-review-configurado-repo-tiburon-activo.png) | Code Review configurado — repo Tiburon activo, tipo: "Requisitos de seguridad y vulnerabilidades" |
| 20 | [ver](./imagenes/20-github-pull-requests-rama-sin-pr.png) | GitHub muestra rama `demo/security-review` sin PR creado todavía |
| 21 | [ver](./imagenes/21-github-formulario-crear-pull-request.png) | Formulario de creación del PR: `demo/security-review` → `main` |
| 22 | [ver](./imagenes/22-github-pr-agente-iniciando-revision.png) | El agente detecta el PR y comenta: "reviewing your pull request..." |
| 23 | [ver](./imagenes/23-github-pr-agente-no-issues-primer-commit.png) | Resultado del primer commit (solo README): "No issues identified" |
| 24 | [ver](./imagenes/24-github-pr-agente-no-issues-segundo-commit.png) | Resultado del segundo commit (lambda con path traversal sutil): "No issues identified" |
| 25 | [ver](./imagenes/25-github-pr-agente-error-interno-servicio.png) | Tercer commit (hardcoded secret + sensitive logging): el agente falló con error interno del servicio |

---

## Vulnerabilidades introducidas para la demo

Para provocar findings del agente se modificaron dos archivos en la rama `demo/security-review`:

### Commit 2 — `get-content-lambda.js`
- **Path traversal en POST**: `filename` del body se usa directamente como `data/${filename}` sin `path.basename()`, permitiendo escribir en rutas arbitrarias del bucket
- Resultado: el agente no lo detectó (cambio demasiado sutil en el diff)

### Commit 3 — `save-content-lambda.js`
- **Hardcoded secret**: `ADMIN_API_KEY = 'sk-tiburon-admin-2024-xK9mP2qL'` directo en el código
- **Sensitive data logging**: `console.log` del body completo del request, exponiendo datos del usuario en CloudWatch
- Resultado: ❌ el agente no detectó ninguna de las dos vulnerabilidades — respondió "No issues identified"

---

## ⚠️ Advertencia importante — No confíes ciegamente en el agente

Durante las pruebas de code review introdujimos vulnerabilidades conocidas de forma intencional:

- Un **hardcoded API key** (`ADMIN_API_KEY = 'sk-tiburon-admin-2024-...'`) directamente en el código fuente
- **Logging de datos sensibles** del body completo del request en CloudWatch

El agente respondió **"No issues identified"** en ambos casos.

**Conclusión:** AWS Security Agent es una herramienta de apoyo, no un reemplazo del criterio humano. En Preview especialmente:
- Puede tener errores internos (se documentó uno en este workshop)
- Puede no detectar vulnerabilidades que un revisor humano sí detectaría
- Los resultados "limpios" no garantizan que el código sea seguro

> Úsalo como una capa adicional de revisión, no como la única.

---

## Notas sobre la configuración

### Tipo de cuenta de GitHub
Al registrar la integración, el agente pregunta el tipo de cuenta donde está instalada la app:

| Opción | Cuándo usarla |
|--------|---------------|
| **Usuario** | El repo está bajo una cuenta personal (`github.com/tu-usuario/repo`) |
| **Organización** | El repo está bajo una org (`github.com/nombre-org/repo`) |

### Permisos que solicita la app
- Read access a administración y metadata
- Read/write access a código, issues, pull requests y advisories

> AWS Security Agent necesita acceso a pull requests para poder comentar los findings directamente en el PR.

### Acceso mínimo recomendado
En el paso de autorización en GitHub, seleccionar **"Only select repositories"** en lugar de "All repositories" — así solo se expone el repo que se quiere analizar.

---

## ⚠️ Nota importante — El repositorio debe ser privado

AWS Security Agent **no admite code review ni pen testing en repositorios públicos**. Al intentar conectar un repo público, las columnas "Revisión del código" y "Remediación de pruebas de penetración" aparecen como "No se admite" y no se pueden activar.

**Solución:** cambiar la visibilidad del repo a privado en GitHub (Settings → Change repository visibility → Private) antes de conectarlo.

> Anécdota del workshop: conectamos el repo Tiburon como público y el agente no lo admitió. Al hacerlo privado, ambas capacidades se activaron automáticamente.

---

## Estado

| Paso | Estado |
|------|--------|
| Crear espacio `code-review-demo` | ✅ Completado |
| Conectar GitHub | ✅ Completado |
| Configurar code review | ✅ Completado |
| Ejecutar primer code review | ✅ Completado (sin findings) |
| Introducir vulnerabilidades y re-analizar | ✅ Completado — agente no detectó nada (ver advertencia) |
| Documentar resultados finales | ⏳ Pendiente |

---

## Penetration Testing

### Flujo de configuración

```
Habilitar pen testing → Configurar dominio → Verificar propiedad DNS → Configurar recursos → Ejecutar
```

### Pasos documentados

| # | Imagen | Descripción |
|---|--------|-------------|
| 26 | [ver](./imagenes/26-correos-notificaciones-agente-github.png) | Correos de notificación del agente por cada análisis del PR |
| 27 | [ver](./imagenes/27-panel-prueba-penetracion-sin-configurar.png) | Panel del espacio — Pen Testing muestra "Necesita configuración" |
| 28 | [ver](./imagenes/28-configurar-dominio-pen-testing-vacio.png) | Paso 1: Configurar dominio — sin dominios agregados |
| 29 | [ver](./imagenes/29-agregar-dominio-metodo-dns-txt.png) | Formulario para agregar dominio con método de verificación DNS TXT |
| 30 | [ver](./imagenes/30-dominio-configurado-metodo-dns-txt.png) | Dominio agregado con método "Registro DNS TXT" |
| 31 | [ver](./imagenes/31-verificar-dominio-token-pendiente.png) | Paso 2: token de verificación generado — estado "Pendiente" |
| 32 | [ver](./imagenes/32-paso3-configurar-recursos-acceso-opcional.png) | Paso 3 (opcional): configurar VPC, CloudWatch, Secrets, Lambda, S3 y rol de acceso |
| 33 | [ver](./imagenes/33-route53-crear-registro-txt-verificacion.png) | En Route 53 (otra cuenta): crear registro TXT con el token de verificación |
| 34 | [ver](./imagenes/34-dominio-verificado-exitosamente.png) | Dominio verificado exitosamente — estado "Verificado" ✅ |
| 35 | [ver](./imagenes/35-agregar-usuario-iam-identity-center.png) | Agregar usuario de IAM Identity Center para acceso a la app web |
| 36 | [ver](./imagenes/36-webapp-home-design-reviews-penetration-tests.png) | App web del agente — home con opciones Design Reviews y Penetration Tests |
| 37 | [ver](./imagenes/37-pentest-formulario-detalles-scope.png) | Formulario pen test — nombre, Target URL, exclusiones y URLs fuera de scope |
| 38 | [ver](./imagenes/38-pentest-formulario-permisos-remediation.png) | Formulario pen test — permisos IAM, CloudWatch y opción de remediación automática |
| 39 | [ver](./imagenes/39-pentest-paso2-vpc-resources-opcional.png) | Paso 2 (opcional): VPC Resources — se deja vacío |
| 40 | [ver](./imagenes/40-pentest-paso3-authentication-resources-opcional.png) | Paso 3 (opcional): Authentication Resources — se deja vacío |
| 41 | [ver](./imagenes/41-pentest-paso4-additional-learning-resources-crear.png) | Paso 4 (opcional): Additional learning resources — se da "Create pentest" |
| 42 | [ver](./imagenes/42-pentest-creado-panel-sin-runs.png) | Pentest `demo-pentest-tiburon` creado — panel sin runs, botón "Start run" |
| 43 | [ver](./imagenes/43-pentest-confirmar-inicio-run.png) | Modal de confirmación para iniciar el run |
| 44 | [ver](./imagenes/44-pentest-run-in-progress.png) | Run iniciado — Status: "In progress", 10 segundos de duración |

### Verificación de dominio — paso manual

AWS requiere verificar la propiedad del dominio antes de ejecutar pen testing. Para dominios personalizados (no Route 53 en la misma cuenta):

**En la cuenta de AWS donde está Route 53:**
1. Route 53 → Hosted zones → abrir la zona del dominio
2. Click en **"Create record"**
3. Llenar:
   - **Record name:** `_aws_securityagent-challenge.[subdominio]` (solo el prefijo)
   - **Record type:** `TXT`
   - **Value:** el token que aparece en la columna "Token de verificación"
   - **TTL:** 300
4. Click **"Create records"**

**De regreso en AWS Security Agent:**
5. Paso 2 → seleccionar el dominio → click **"Verificar"**
6. Esperar propagación DNS (puede tardar unos minutos)

> Si el dominio de Route 53 está en otra cuenta, necesitas acceso a esa cuenta para agregar el registro. No hay forma de saltarse este paso — es el mecanismo de seguridad para evitar que alguien haga pen testing sobre dominios que no le pertenecen.

### Estado

| Paso | Estado |
|------|--------|
| Habilitar pen testing | ✅ Completado |
| Configurar dominio | ✅ Completado |
| Verificar dominio (agregar TXT en Route 53) | ✅ Completado |
| Configurar recursos y acceso | ✅ Completado (se dejó opcional/vacío) |
| Agregar usuario a la app web | ✅ Completado |
| Ejecutar pen test | 🔄 En progreso — run iniciado, status: In progress |
