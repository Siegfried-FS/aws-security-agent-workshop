# Proyecto 01: VirusTotal S3 Scanner — Design Review

Sistema serverless que valida archivos subidos a S3 contra VirusTotal y los clasifica automáticamente.

```
Usuario sube archivo → S3 inbox → Lambda → VirusTotal API
                                              ├── Limpio → S3 clean
                                              └── Malicioso → S3 quarantine + alerta SNS
```

---

## Design Reviews

Se enviaron 3 versiones del documento de arquitectura al agente para ver cómo evolucionan los findings.

| Fase | Documento | Descripción | Resultado |
|------|-----------|-------------|-----------|
| Fase 1 | [fase-1-propuesta-inicial.md](./design-review/fase-1-propuesta-inicial.md) | Propuesta inicial del equipo | ❌ 9 no cumple |
| Fase 2 | [fase-2-propuesta-mejorada.md](./design-review/fase-2-propuesta-mejorada.md) | Mejorada por el equipo (sin agente) | Ver resultados |
| Fase 3 | [fase-3-propuesta-con-recomendaciones-agente.md](./design-review/fase-3-propuesta-con-recomendaciones-agente.md) | Corregida aplicando recomendaciones del agente | Ver resultados |

👉 [Comparativa de las 3 fases](./resultados/comparativa.md)

---

## Cómo reproducirlo

1. Abre AWS Security Agent → crea un espacio de agente
2. En el espacio → **Design Review** → **Create design review**
3. Sube uno de los documentos de `design-review/`
4. Espera a que cambie de **In Progress** a **Completed**
5. Compara los findings con la fase anterior
