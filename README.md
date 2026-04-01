# AWS Security Agent — Workshop

> **[AWS User Group Playa Vicente](https://www.meetup.com/aws-user-group-playa-vicente)**

> ⚠️ **Nota importante — 1 de abril de 2026:** La capacidad de **Penetration Testing** pasó de Preview a **GA** y comenzó a generar costo desde esta fecha. **Design Review y Code Review siguen en Preview gratuito**. Verifica siempre la [página de precios](https://aws.amazon.com/security-agent/pricing/) antes de ejecutar un pen test.

Repositorio de aprendizaje y documentación del workshop de AWS Security Agent. Incluye errores encontrados, limitaciones reales y observaciones honestas del servicio — para que no cometas los mismos errores.

---

## Estructura

```
aws-security-agent-workshop/
│
├── README.md
│
├── proyectos/
│   ├── 01-virustotal-s3/            # Escáner de archivos con VirusTotal — Design Review (3 fases)
│   │   ├── README.md
│   │   └── design-review/           # Documentos subidos al agente
│   │
│   └── 02-app-vulnerable/           # App con vulnerabilidades — Code Review + Pen Testing
│       └── README.md
│
└── workshop/
    ├── guia-paso-a-paso.md          # Guía completa desde cero
    └── requisitos-personalizados.md # Cómo agregar contexto de negocio
```

---

## Proyectos

| # | Proyecto | Features probadas | Estado |
|---|----------|-------------------|--------|
| 01 | [VirusTotal S3 Scanner](./proyectos/01-virustotal-s3/) | Design Review (3 fases) | ✅ Completado |
| 02 | [App Vulnerable](./proyectos/02-app-vulnerable/) | Code Review + Pen Testing | ✅ Completado |

---

## Features de AWS Security Agent

| Feature | Descripción | Estado en este workshop |
|---------|-------------|------------------------|
| Design Review | Analiza documentos de arquitectura | ✅ Probado — 3 fases |
| Code Review | Revisa PRs en GitHub automáticamente | ✅ Probado — ver advertencia |
| Penetration Testing | Pen test on-demand contra una URL | ✅ Probado |
| Requisitos personalizados | Define tus propios estándares de seguridad | 📖 Documentado |

---

## Costos — Antes de empezar

> ⚠️ Lee esto antes de explorar el servicio.

**AWS Security Agent (GA desde abril 2026)**
- El pricing oficial puede haber cambiado al pasar a GA
- Verifica siempre en: [aws.amazon.com/security-agent/pricing](https://aws.amazon.com/security-agent/pricing/)

**Recursos que SÍ pueden generar costo:**
- EC2, Lambda, S3 que crees para los proyectos de prueba
- CloudWatch Logs

**Recomendación: configura una alerta de billing antes de empezar**

1. Consola AWS → **Billing** → **Budgets** → **Create budget**
2. Seleccionar **"Zero spend budget"** (te avisa con el primer centavo)
3. Agregar tu email → **Create budget**

---

## Observaciones del Workshop

### ✅ Lo que funciona bien
- Design reviews funcionan sin configuración adicional — listo en minutos
- La web app es más limpia que la consola para el día a día
- Pen testing con verificación DNS es sólido y el proceso es claro

### ⚠️ Limitaciones encontradas
- Solo disponible en `us-east-1` (durante Preview — verificar en GA)
- Code review y pen testing **no funcionan con repos públicos de GitHub**
- Penetration testing requiere verificar propiedad del dominio (paso manual en Route 53)

### ❌ Problemas encontrados
- El agente **no detectó** un hardcoded API key ni sensitive logging en code review
- El agente tuvo un **error interno del servicio** en el tercer commit del code review
- Ver detalles completos en [proyectos/02-app-vulnerable/README.md](./proyectos/02-app-vulnerable/README.md)

---

## Guía rápida

👉 [Guía paso a paso desde cero](./workshop/guia-paso-a-paso.md)

---

## Recursos oficiales

- [Documentación oficial](https://docs.aws.amazon.com/securityagent/latest/userguide/what-is.html)
- [Blog de lanzamiento](https://aws.amazon.com/blogs/aws/new-aws-security-agent-secures-applications-proactively-from-design-to-deployment-preview/)
- [Página del servicio](https://aws.amazon.com/security-agent/)

---

## AWS User Group Playa Vicente

| Canal | Enlace |
|-------|--------|
| 📅 Meetup | [meetup.com/aws-user-group-playa-vicente](https://www.meetup.com/aws-user-group-playa-vicente) |
| 📸 Instagram | [@aws_ug_playa_vicente](https://www.instagram.com/aws_ug_playa_vicente/) |
| ▶️ YouTube | [Canal YouTube](https://www.youtube.com/channel/UCObJL_Id1HHsx1hg0aNISlw) |
| 💬 Telegram | [t.me/AUGPlayaVicente](https://t.me/AUGPlayaVicente) |
| 💼 LinkedIn | [linkedin.com/company/111588988](https://www.linkedin.com/company/111588988) |

---

*Workshop creado con ❤️ para AWS User Group Playa Vicente*
