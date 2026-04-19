# 🤖 Guía Claude Code — Stack Laravel · WordPress · Astro · React · Next.js

> Guía de referencia para usar Claude Code de forma profesional en proyectos reales.
> Mantén este repositorio como plantilla base para cada nuevo proyecto.

---

## 📁 Estructura del repositorio

```
mi-proyecto/
├── CLAUDE.md                    # ← Claude lee esto en cada sesión
├── .claude/
│   ├── commands/                # Comandos slash personalizados
│   │   ├── review.md
│   │   ├── deploy.md
│   │   └── test.md
│   └── skills/                  # Skills específicos del proyecto
│       ├── laravel.md
│       ├── wordpress.md
│       ├── astro.md
│       └── react-next.md
├── tasks/
│   ├── todo.md                  # Plan de tareas activo
│   └── lessons.md               # Errores corregidos → reglas aprendidas
├── docs/
│   ├── architecture.md          # Decisiones arquitectónicas
│   ├── api.md                   # Contratos de API
│   └── deployment.md            # Procedimientos de despliegue
└── .gitignore
```

---

## 🚀 Instalación de Claude Code

```bash
# Requiere Node.js 18+
npm install -g @anthropic-ai/claude-code

# Verificar instalación
claude --version

# Iniciar en tu proyecto
cd mi-proyecto
claude

# Generar CLAUDE.md inicial (luego refinar)
/init
```

---

## 📖 Índice de esta guía

1. [CLAUDE.md — El archivo más importante](./docs/claude-md-guide.md)
2. [Prompts efectivos por tipo de tarea](./docs/prompts-guide.md)
3. [Skills por tecnología](./docs/skills-guide.md)
4. [Comandos personalizados](./docs/commands-guide.md)
5. [Testing y calidad](./docs/testing-guide.md)
6. [Seguridad](./docs/security-guide.md)
7. [Despliegue](./docs/deployment-guide.md)
8. [Flujo de trabajo diario](./docs/workflow-guide.md)

---

## ⚡ Reglas de oro (TL;DR)

| Regla | Por qué |
|-------|---------|
| Empieza siempre con Plan Mode | Evita código incorrecto desde el principio |
| CLAUDE.md < 200 líneas | Más largo = Claude ignora instrucciones |
| `/clear` entre tareas no relacionadas | Contexto limpio = mejores resultados |
| Cuando Claude falla 2 veces → `/clear` y reescribe el prompt | No contamines el contexto con intentos fallidos |
| Actualiza `tasks/lessons.md` cuando corrijas algo | Claude aprende tus patrones |
| Nunca commites secrets | Regla absoluta, sin excepciones |