# Guía Paso a Paso — AWS Security Agent

> Sigue esta guía mientras exploras la consola por primera vez.

---

## Paso 1: Acceder al servicio

1. Ingresar a la consola AWS → región **us-east-1** (N. Virginia) — única región disponible en preview
2. Buscar **"AWS Security Agent"** en la barra de búsqueda
3. Entrar a la **Security Agent Web App** (abre en pestaña separada)

> 📸 Guardar captura en `workshop/imagenes/`

---

## Paso 2: Crear un Espacio de Agente

Un "Espacio de agente" agrupa todos tus reviews y pen tests (en la consola aparece como "Crear espacio de agente").

1. Click en **"Crear espacio de agente"**
2. Llenar:
   - **Nombre**: `virustotal-s3-scanner`
   - **Descripción**: `Escáner de archivos maliciosos con VirusTotal y S3`
3. Click en **Crear**

> 📸 Guardar captura en `proyectos/01-virustotal-s3/imagenes/`

---

## Paso 3: Revisar Requisitos de Seguridad

1. Click en **"Requisitos de seguridad"** en el menú lateral
2. Hay 10 requisitos administrados por AWS, todos activados por defecto
3. Solo revisar — no cambiar nada aún

Requisitos relevantes para nuestro proyecto (ya activos):
- `Secret Protection Best Practices` — detecta secrets expuestos
- `Privileged Access Best Practices` — detecta permisos excesivos
- `Audit Logging Best Practices` — detecta falta de logs
- `Information Protection Best Practices` — detecta datos sin cifrar

> 📸 Guardar captura en `workshop/imagenes/`

---

## Paso 4: Abrir la Aplicación Web

1. Ir a **"Espacios de agente"** → click en `virustotal-s3-scanner`
2. En el card **"Revisión del diseño"** → click **"Iniciar en la aplicación web"**
3. Se abre la Security Agent Web App en una pestaña nueva

---

## Paso 5: Design Review — Fase 1 (Propuesta Inicial)

Subimos la propuesta inicial para ver qué detecta el agente.

1. En la web app click en **"Create design review"**
2. Nombre: `Fase 1 - Propuesta inicial de arquitectura`
3. Subir: `proyectos/01-virustotal-s3/design-review/fase-1-propuesta-inicial.md`
4. Click **"Start design review"**
5. Esperar a que cambie de **In Progress** a **Completed**
6. Click **"View details"** y documentar los findings

> 📸 Guardar captura de los resultados en `proyectos/01-virustotal-s3/imagenes/`

---

## Paso 6: Design Review — Fase 2 (Propuesta Mejorada)

Con el feedback del agente, subimos la arquitectura mejorada.

1. En la web app click en **"Create design review"**
2. Nombre: `Fase 2 - Propuesta mejorada`
3. Subir: `proyectos/01-virustotal-s3/design-review/fase-2-propuesta-mejorada.md`
4. Click **"Start design review"**
5. Comparar findings con la Fase 1

> 📸 Guardar captura de los resultados en `proyectos/01-virustotal-s3/imagenes/`

---

## Paso 6: Code Review (requiere GitHub)

1. Ir a **"Code reviews"** en el menú lateral
2. Conectar repositorio de GitHub
3. Configurar qué repos monitorear
4. Abrir un PR y esperar el análisis automático

> ⚠️ Documentar si hay fricción al conectar GitHub

---

## Paso 7: Penetration Testing

1. Ir a **"Penetration tests"** → **"Create"**
2. Ingresar URL de la aplicación desplegada
3. Proporcionar contexto (código fuente, documentación)
4. Iniciar el test y revisar el reporte

> ⚠️ Requiere tener la app desplegada primero

---

## Notas del Workshop

| Paso | ¿Funcionó? | Observaciones |
|------|-----------|---------------|
| Acceso al servicio | | |
| Crear aplicación | | |
| Security Requirements | | |
| Design Review inseguro | | |
| Design Review seguro | | |
| Code Review | | |
| Pen Test | | |
