# Decisiones Arquitectónicas

> Registra aquí las decisiones importantes de arquitectura del proyecto.
> Esto ayuda a Claude a entender el POR QUÉ detrás del código, no solo el qué.
> Formato: ADR (Architecture Decision Record) simplificado.

---

## Cómo añadir una decisión

```
## [Fecha] — [Título de la decisión]
**Contexto:** por qué había que tomar una decisión
**Opciones consideradas:** qué alternativas se evaluaron
**Decisión:** qué se eligió y por qué
**Consecuencias:** qué implica esta decisión a futuro
```

---

## Ejemplo: Estructura de un proyecto Laravel + Astro

## 2025-01-15 — Separar backend (Laravel) y frontend (Astro) en repositorios distintos

**Contexto:**
El proyecto necesita un backend con lógica de negocio compleja (pedidos, usuarios, pagos)
y un frontend orientado a contenido con máximo rendimiento (blog, landing pages, catálogo).

**Opciones consideradas:**
- Monorepo con Laravel + Inertia.js → acoplamiento frontend/backend
- Laravel API + Astro en repo separado → desacoplado pero más infraestructura
- Next.js fullstack → no dominamos Next.js suficientemente para producción crítica

**Decisión:**
Laravel como API REST pura + Astro como frontend estático con llamadas a la API.
Astro se despliega en Vercel/Netlify, Laravel en servidor propio.

**Consecuencias:**
- CORS configurado en Laravel para el dominio del frontend
- Autenticación vía tokens (Sanctum), no sesiones
- El frontend no tiene acceso directo a la BD → todo pasa por la API
- Despliegues son independientes → se puede actualizar frontend sin tocar backend

---

## Ejemplo: WordPress

## 2025-02-01 — Toda la lógica de personalización en plugin propio, no en functions.php

**Contexto:**
El tema hijo acumulaba demasiada lógica en functions.php. Al cambiar de tema,
se perdería toda la funcionalidad de negocio.

**Opciones consideradas:**
- Seguir en functions.php del tema hijo → simple pero frágil
- Plugin propio por funcionalidad → más mantenible, independiente del tema
- Plugin único con toda la lógica del proyecto → equilibrio entre simplicidad y portabilidad

**Decisión:**
Un plugin principal (`mi-proyecto-core`) con toda la lógica de negocio.
El tema hijo solo controla diseño (CSS, templates Blade/PHP puros).

**Consecuencias:**
- Cambiar de tema no afecta la funcionalidad
- El plugin es más fácil de versionar y desplegar de forma independiente
- Más archivos que mantener, pero cada uno con responsabilidad clara

---

<!-- Añade aquí tus decisiones reales conforme el proyecto evoluciona -->
