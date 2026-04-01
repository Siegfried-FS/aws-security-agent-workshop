# Cómo agregar contexto de negocio — Requisitos de Seguridad Personalizados

> Feature avanzada. No se usó en este workshop para mantenerlo simple, pero es clave para uso en producción.

---

## ¿Para qué sirve?

Por defecto el agente evalúa con criterios genéricos de AWS. Con requisitos personalizados le dices exactamente qué estándares debe enforcar en **tu** organización — por ejemplo:
- "Todos los sistemas deben usar Cognito para autenticación"
- "Prohibido usar S3 con acceso público"
- "Toda API debe tener rate limiting"
- "Los sistemas de salud deben cumplir HIPAA"

Se configuran **una vez** y se aplican automáticamente a todos los design reviews y code reviews.

---

## Cómo llegar

1. Consola AWS → **AWS Security Agent**
2. Menú lateral → **"Requisitos de seguridad"**
3. Tab **"Requisitos de seguridad personalizados"**
4. Click **"Crear requisito de seguridad"**

---

## Campos del formulario

### Nombre del requisito *(máx. 80 caracteres)*
Nombre claro que identifique el control. Evita nombres genéricos.

```
✅ "Autenticación obligatoria con Amazon Cognito"
❌ "Seguridad de usuarios"
```

### Descripción *(máx. 500 caracteres)*
Qué enforca este control y por qué importa. Enfócate en el riesgo que mitiga.

```
Todos los sistemas que manejen usuarios finales deben usar Amazon Cognito 
como proveedor de identidad. Esto garantiza MFA, gestión centralizada de 
sesiones y cumplimiento con la política de acceso de la organización.
```

### Aplicabilidad *(máx. 10,000 caracteres)*
Define cuándo aplica y cuándo NO aplica. Esto evita falsos positivos.

```
Este control aplica a TODOS los sistemas que:
- Tengan usuarios finales que se autentiquen
- Expongan APIs accesibles desde internet
- Manejen datos de clientes

Marcar como NOT_APPLICABLE si:
- El sistema es exclusivamente interno entre servicios AWS (service-to-service)
- Es un sistema batch sin interacción de usuarios
```

### Criterios de cumplimiento *(máx. 10,000 caracteres)*
La parte más importante. Define qué es cumplir y qué es no cumplir. Sé específico.

```
Un diseño ES CUMPLIENTE si demuestra:
- Uso explícito de Amazon Cognito User Pool para autenticación de usuarios
- Tokens JWT validados en cada request
- Configuración de MFA habilitada

Un diseño NO ES CUMPLIENTE si:
- Implementa autenticación propia (custom auth) sin Cognito
- Permite acceso sin autenticación a endpoints con datos de usuarios
- Usa usuario/contraseña sin MFA para acceso a datos sensibles
```

### Guía de remediación *(opcional, máx. 10,000 caracteres)*
Pasos concretos para corregir la violación. Incluye ejemplos y links internos.

```
Para implementar Cognito:
1. Crear un User Pool en us-east-1
2. Configurar MFA como obligatorio
3. Integrar el SDK de Cognito en la aplicación
4. Validar el JWT en cada endpoint con el middleware estándar de la organización

Referencia interna: [link a tu wiki/confluence]
Documentación AWS: https://docs.aws.amazon.com/cognito/
```

---

## Tips

- **Applicability bien definida** es lo más importante — si es muy amplia, el agente marcará falsos positivos
- Puedes partir de un requisito administrado por AWS y personalizarlo (botón "Personalizar" en la tab de requisitos administrados)
- Los cambios aplican solo a revisiones **nuevas** — las existentes no se ven afectadas
- Puedes tener múltiples requisitos habilitados al mismo tiempo

---

## Ejemplo para el proyecto VirusTotal S3

Si quisiéramos configurar el contexto de este proyecto, crearíamos requisitos como:

| Requisito | Descripción |
|-----------|-------------|
| Autenticación con Cognito | Todo upload debe requerir JWT de Cognito |
| Secrets en Secrets Manager | Prohibido hardcodear API keys o credenciales |
| S3 siempre privado | Ningún bucket puede tener acceso público |
| Escaneo antes de procesar | Todo archivo debe pasar por VirusTotal antes de moverse a clean |
