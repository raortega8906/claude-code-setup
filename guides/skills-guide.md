# Guía de Skills para Claude Code

## ¿Qué son los Skills?

Los skills son archivos Markdown en `.claude/skills/` que Claude carga **bajo demanda**,
cuando la tarea actual los requiere. A diferencia de CLAUDE.md (que carga siempre),
los skills mantienen el contexto limpio y eficiente.

```
CLAUDE.md    → Siempre cargado → Contexto universal
Skills       → Cargados cuando son relevantes → Conocimiento especializado
```

---

## Skills incluidos en esta plantilla

| Skill | Archivo | Cuándo lo usa Claude |
|-------|---------|---------------------|
| Laravel | `.claude/skills/laravel.md` | Controllers, Models, Services, tests PHP |
| WordPress | `.claude/skills/wordpress.md` | Plugins, temas hijo, hooks, AJAX, WooCommerce |
| Astro | `.claude/skills/astro.md` | Componentes .astro, Content Collections, SSG |
| React/Next.js | `.claude/skills/react-next.md` | Componentes React, App Router, TypeScript |

---

## Cómo referenciar skills desde CLAUDE.md

Añade esta sección al final de tu CLAUDE.md:

```markdown
## Skills disponibles
Para tareas especializadas, consulta los skills en `.claude/skills/`:
- `laravel.md` — Arquitectura, patrones, testing PHP
- `wordpress.md` — Plugins, tema hijo, seguridad WordPress
- `astro.md` — Componentes, imágenes, Content Collections
- `react-next.md` — React, Next.js App Router, TypeScript estricto
```

Claude los cargará automáticamente cuando detecte que son relevantes.

---

## Cómo crear un skill propio

### Estructura de un skill

```markdown
# Skill: [Nombre]

> Cuándo Claude debe cargar este skill (1-2 frases)

## Contexto / Versión
[Versión de la tecnología, configuración específica del proyecto]

## Reglas de arquitectura
[Decisiones específicas de ESTE proyecto, no docs genéricas]

## Patrones con código
[Ejemplos concretos de cómo se hace en este proyecto]

## Anti-patrones — Qué evitar
[Lo que NO se debe hacer y por qué]

## Comandos / Scripts
[Los comandos específicos que Claude necesitará ejecutar]
```

### Ejemplo: skill para una API externa

```markdown
# Skill: Integración con Stripe

> Claude carga este skill cuando trabaja con pagos, suscripciones o webhooks de Stripe.

## Versión y configuración
- stripe-php: ^13.0
- API version: 2024-06-20
- Variables de entorno: STRIPE_SECRET, STRIPE_WEBHOOK_SECRET, STRIPE_PRICE_ID_*

## Arquitectura de pagos en este proyecto
Los pagos pasan por: StripeService → PaymentController → Webhook handler
La lógica de negocio va en StripeService, no en el Controller.

## Crear checkout session
[código específico del proyecto]

## Verificar webhooks (CRÍTICO)
[siempre verificar la firma del webhook]

## Manejo de errores de Stripe
[cómo maneja el proyecto los errores de la API]
```

### Ejemplo: skill para WooCommerce

```markdown
# Skill: WooCommerce

> Claude carga este skill para personalización de WooCommerce en este proyecto.

## Versión
- WooCommerce: 8.x
- Todos los hooks de WooCommerce tienen prefijo woocommerce_

## Regla principal
Usar hooks nativos de WooCommerce SIEMPRE antes de sobreescribir templates.
Verificar en https://hookr.io si existe el hook correcto.

## Hooks más usados en este proyecto
[lista de los que el proyecto ya usa]

## Precios y descuentos
[cómo los gestiona este proyecto]
```

---

## Skills de la comunidad

La comunidad de Claude Code comparte skills reutilizables.
Puedes instalarlos con `/plugin` en Claude Code y explorar los disponibles.

Para proyectos propios, los skills personalizados en `.claude/skills/` suelen
ser más valiosos porque conocen las especificidades de tu proyecto.

---

## Cuándo un skill se convierte en CLAUDE.md

Si un skill lo usas en **absolutamente todas** las sesiones del proyecto,
considera mover las partes esenciales a CLAUDE.md (con brevedad).

Pero la mayoría de veces es mejor mantenerlos separados:
- CLAUDE.md ligero = mejor seguimiento de instrucciones
- Skills granulares = contexto relevante cuando se necesita

---

## Mantenimiento de skills

Los skills deben actualizarse cuando:
- Cambias la versión de una librería (nuevas APIs, deprecaciones)
- Detectas que Claude usa patrones incorrectos repetidamente
- El proyecto evoluciona y los patrones cambian

Señal de que un skill necesita actualización:
> Claude sigue instrucciones del skill pero el código no funciona con la versión actual.
