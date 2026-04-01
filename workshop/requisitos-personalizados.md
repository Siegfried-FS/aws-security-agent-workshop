# Requisitos de Seguridad Personalizados

AWS Security Agent incluye 10 requisitos administrados por AWS activados por defecto. Además puedes definir los tuyos propios para reflejar los estándares de tu organización.

---

## Requisitos administrados por AWS (por defecto)

Todos activados al crear un espacio de agente:

| Requisito | Descripción |
|-----------|-------------|
| Mejores prácticas para el registro de auditorías | El sistema admite monitorización de seguridad |
| Mejores prácticas de autenticación | Solo usuarios legítimos pueden acceder |
| Mejores prácticas de autorización | Los sistemas siguen mejores prácticas de autorización |
| Mejores prácticas en protección de la información | Datos sensibles confidenciales e inalterados |
| Mejores prácticas para la protección de registros | Integridad y confidencialidad de los registros |
| Mejores prácticas para el acceso privilegiado | Medidas adecuadas para funciones privilegiadas |
| Mejores prácticas para la protección de secretos | Credenciales permanecen confidenciales |
| Mejores prácticas para la seguridad por defecto | Configuración predeterminada segura |
| Mejores prácticas para el aislamiento de inquilinos | Separación adecuada entre sistemas |
| Buenas prácticas de criptografía de confianza | Uso correcto de criptografía |

---

## Cómo agregar requisitos personalizados

1. En la consola AWS → tu espacio de agente → **"Requisitos de seguridad"**
2. Click en **"Personalizar"**
3. Define tus propios requisitos con:
   - **Nombre:** identificador del requisito
   - **Descripción:** qué debe cumplir el sistema
   - **Criterios de relevancia:** cuándo aplica este requisito

### Ejemplos de requisitos personalizados útiles

```
Nombre: Cumplimiento PCI-DSS para datos de tarjetas
Descripción: Los sistemas que procesan datos de tarjetas de pago deben cumplir
con los controles de PCI-DSS aplicables, incluyendo cifrado en tránsito y en
reposo, control de acceso y registro de auditorías.

Nombre: Retención de logs mínima 90 días
Descripción: Todos los logs de aplicación y acceso deben retenerse por un
mínimo de 90 días para cumplir con los requisitos de auditoría interna.

Nombre: Autenticación multifactor obligatoria
Descripción: Todos los accesos administrativos y de usuarios privilegiados
deben requerir MFA.
```

---

## Observación del workshop

Los requisitos personalizados son especialmente útiles cuando:
- Tu organización tiene estándares propios (ISO 27001, SOC 2, PCI-DSS, etc.)
- Quieres que el agente evalúe aspectos específicos de tu industria
- Los requisitos administrados por AWS son demasiado genéricos para tu caso de uso

> Durante el workshop usamos solo los requisitos administrados por defecto. Los resultados del Design Review fueron suficientemente detallados para identificar problemas reales de arquitectura.
