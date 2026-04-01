# Comparativa de Resultados — Design Reviews

> ¿Qué diferencia hace usar AWS Security Agent vs. mejorar por cuenta propia?

---

## Contexto importante — Cómo se hicieron estas revisiones

Todas las revisiones de este proyecto se hicieron con los **requisitos de seguridad por defecto de AWS** — sin contexto personalizado de negocio.

El servicio permite dos niveles de contexto que **no usamos aquí**:

**1. Requisitos de seguridad personalizados** (Consola AWS → Requisitos de seguridad → tab "Requisitos personalizados")
- Puedes definir estándares propios: "todos los sistemas deben usar Cognito", "prohibido S3 público", etc.
- Se aplican automáticamente a todos los design reviews y code reviews del espacio
- Hacen el análisis mucho más preciso y relevante para tu organización

**2. Contexto por proyecto en Pen Testing**
- Al crear un pen test puedes subir código fuente, documentación y describir flujos de negocio
- El agente usa ese contexto para generar ataques específicos a tu app, no genéricos

**Conclusión:** Los resultados que verás aquí son con criterios genéricos de AWS. Con contexto personalizado, el análisis sería más preciso y los findings más relevantes para el negocio.

---

## Resumen de las 3 Fases

| | Fase 1 | Fase 2 | Fase 3 |
|--|--------|--------|--------|
| **Contexto** | Propuesta inicial del equipo | Mejorada por el equipo (sin agente) | Corregida siguiendo recomendaciones del agente |
| **❌ No cumple** | 9 | 0 | 0 |
| **⚠️ Datos insuficientes** | 0 | 5 | 2 |
| **✅ Obediente** | 0 | 4 | 7 |
| **➖ No aplicable** | 1 | 1 | 1 |

---

## Análisis

### Fase 1 → Fase 2: ¿Qué resolvió el equipo solo?
El equipo eliminó todos los **No cumple** — resolvieron los problemas técnicos evidentes: cifrado, permisos, secrets. Pero dejaron 5 controles en **Datos insuficientes** porque no documentaron con suficiente detalle los controles que sí implementaron.

**Lección:** Saber hacer algo no es suficiente — hay que documentarlo para que pueda ser validado.

### Fase 2 → Fase 3: ¿Qué aportó seguir las recomendaciones del agente?
Pasó de 4 a **7 controles obedientes**. Los 2 restantes son detalles muy específicos de documentación que el equipo podría resolver en una iteración más.

**Lección:** El agente no solo detecta problemas técnicos — también detecta huecos en la documentación que un revisor humano podría pasar por alto.

---

## Ventajas y Desventajas

### Equipo sin agente
| ✅ Ventajas | ❌ Desventajas |
|------------|---------------|
| Conoce el contexto del negocio | Puede pasar por alto detalles de documentación |
| Decisiones basadas en experiencia real | Revisión toma tiempo (juntas, iteraciones) |
| Criterio para priorizar riesgos | Inconsistente entre proyectos y equipos |

### Con AWS Security Agent
| ✅ Ventajas | ❌ Desventajas |
|------------|---------------|
| Revisión en minutos | Sin contexto personalizado, evalúa con criterios genéricos |
| Consistente — mismos criterios siempre | "Datos insuficientes" si el doc no es detallado |
| Remediación específica y accionable | Depende de la calidad del documento subido |
| Escala a todos los proyectos | Solo us-east-1 en preview |

---

## Preguntas de interes

**¿El agente reemplaza al equipo de seguridad?**
No. El equipo define los estándares; el agente los valida automáticamente en cada revisión.

**¿Qué pasa si el documento está mal escrito?**
Responde "Datos insuficientes" — no inventa información. La calidad del documento importa.

**¿Cuánto tarda una revisión?**
En nuestras pruebas: menos de 10 minutos. No medimos el tiempo exacto pero fue cuestión de segundos/minutos — solo hacíamos refresh hasta ver el resultado.

**¿Los datos que subo son privados?**
Sí — AWS confirma que no se usan para entrenar modelos.

**¿Se puede configurar con contexto de mi empresa?**
Sí, pero no lo hicimos en este workshop para mantenerlo simple. Es una feature avanzada para cuando ya conoces el servicio.

**¿Qué tan diferente es Fase 2 de Fase 3?**
Técnicamente casi igual — la diferencia está en el nivel de detalle de la documentación. El agente enseña a documentar mejor, no solo a construir mejor.

---

## Conclusión

El mayor valor de AWS Security Agent no es encontrar vulnerabilidades que el equipo no conoce — es **acelerar y estandarizar** el proceso de revisión, y **forzar documentación de calidad** desde el diseño.

Un equipo experimentado llega a resultados similares, pero el agente lo hace en minutos, de forma consistente, para todos los proyectos.
