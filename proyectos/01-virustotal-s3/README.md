# Proyecto 01: VirusTotal S3 Scanner — Design Review

Sistema serverless que valida archivos subidos a S3 contra VirusTotal y los clasifica automáticamente.

---

## Arquitectura del proyecto

```
Usuario sube archivo → S3 inbox → Lambda → VirusTotal API
                                              ├── Limpio     → S3 clean
                                              └── Malicioso  → S3 quarantine + alerta SNS
```

---

## Por qué este proyecto para el workshop

Es un caso de uso real con múltiples superficies de ataque que el agente puede evaluar:
- Manejo de secrets (API key de VirusTotal)
- Permisos IAM de Lambda
- Cifrado de buckets S3
- Logging y auditoría
- Comunicación con API externa

Esto lo hace ideal para demostrar el valor iterativo del Design Review.

---

## Las 3 fases del Design Review

La idea fue someter el mismo proyecto en 3 estados distintos para ver cómo evoluciona el score.

### Fase 1 — Propuesta inicial (arquitectura insegura)

Documento: [`design-review/fase-1-propuesta-inicial.md`](./design-review/fase-1-propuesta-inicial.md)

Problemas intencionales en esta versión:
- API key de VirusTotal hardcodeada en el código Lambda
- Buckets S3 sin cifrado
- Lambda con permisos `s3:*` (demasiado amplios)
- Sin logging ni CloudTrail
- Sin validación del tipo de archivo antes de enviarlo a VirusTotal

**Resultado del agente:**

| Estado | Cantidad |
|--------|----------|
| 🔴 No cumple | 9 |
| 🟡 Datos insuficientes | 0 |
| 🟢 Compliant | 0 |
| ⚪ No aplicable | 1 |

Findings detectados:
- Mejores prácticas para el registro de auditorías — No cumple
- Mejores prácticas en protección de la información — No cumple
- Mejores prácticas de autorización — No cumple
- Mejores prácticas para la seguridad por defecto — No cumple
- Mejores prácticas para la protección de secretos — No cumple
- Mejores prácticas para la protección de registros — No cumple
- Mejores prácticas para el acceso privilegiado — No cumple
- Buenas prácticas de criptografía de confianza — No cumple
- Mejores prácticas para el aislamiento de inquilinos — No aplicable

---

### Fase 2 — Propuesta mejorada (sin usar el agente)

Documento: [`design-review/fase-2-propuesta-mejorada.md`](./design-review/fase-2-propuesta-mejorada.md)

Mejoras aplicadas por el equipo antes de ver las recomendaciones del agente:
- API key movida a AWS Secrets Manager
- Buckets S3 con cifrado SSE-S3
- Permisos Lambda más restrictivos
- CloudTrail habilitado

**Resultado del agente:**

| Estado | Cantidad |
|--------|----------|
| 🔴 No cumple | 0 |
| 🟡 Datos insuficientes | 5 |
| 🟢 Compliant | 4 |
| ⚪ No aplicable | 1 |

Observación: el agente marcó 5 como "Datos insuficientes" porque el documento no describía con suficiente detalle los controles implementados. No significa que estén mal — significa que el documento no lo evidencia.

---

### Fase 3 — Aplicando recomendaciones del agente

Documento: [`design-review/fase-3-propuesta-con-recomendaciones-agente.md`](./design-review/fase-3-propuesta-con-recomendaciones-agente.md)

Mejoras adicionales basadas en los findings de la Fase 1 y las sugerencias del agente:
- Descripción explícita de todos los controles de autenticación y autorización
- Logging detallado documentado (CloudWatch + CloudTrail)
- Política de retención de logs especificada
- Controles de acceso privilegiado documentados

**Resultado del agente:**

| Estado | Cantidad |
|--------|----------|
| 🔴 No cumple | 0 |
| 🟡 Datos insuficientes | 2 |
| 🟢 Compliant | 7 |
| ⚪ No aplicable | 1 |

---

## Comparativa de las 3 fases

| Requisito | Fase 1 | Fase 2 | Fase 3 |
|-----------|--------|--------|--------|
| Registro de auditorías | 🔴 | 🟡 | 🟡 |
| Autenticación | 🔴 | 🟡 | 🟢 |
| Autorización | 🔴 | 🟡 | 🟢 |
| Protección de la información | 🔴 | 🟢 | 🟢 |
| Protección de registros | 🔴 | 🟡 | 🟢 |
| Acceso privilegiado | 🔴 | 🟢 | 🟢 |
| Protección de secretos | 🔴 | 🟡 | 🟢 |
| Seguridad por defecto | 🔴 | 🟢 | 🟢 |
| Aislamiento de inquilinos | ⚪ | ⚪ | ⚪ |
| Criptografía de confianza | 🔴 | — | 🟢 |

---

## Lecciones aprendidas

1. **El agente evalúa lo que está documentado, no lo que está implementado.** Si tu arquitectura tiene controles pero el documento no los describe, el agente los marcará como "Datos insuficientes".

2. **"Datos insuficientes" no es lo mismo que "No cumple".** Es una señal de que debes ser más explícito en tu documentación.

3. **El flujo iterativo funciona.** De 9 "No cumple" a 0 en 3 iteraciones, con el agente guiando cada mejora.

4. **El Design Review no requiere configuración adicional.** Es la feature más fácil de empezar — solo necesitas un espacio de agente y un documento.
