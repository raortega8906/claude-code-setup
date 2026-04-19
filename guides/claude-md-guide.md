# Guía: Cómo escribir y mantener CLAUDE.md

## ¿Qué es CLAUDE.md?

Es el archivo que Claude lee **automáticamente al inicio de cada sesión**.
Es tu forma de darle contexto persistente sin repetirlo cada vez.

Piénsalo como el "onboarding" que le darías a un nuevo compañero de equipo:
tecnologías, estructura del proyecto, convenciones, cosas a evitar.

---

## Reglas de oro para CLAUDE.md

### 1. Menos es más — máximo 200 líneas

Investigaciones muestran que los LLMs pueden seguir ~150-200 instrucciones con fiabilidad.
El sistema de Claude Code ya usa ~50 de esas instrucciones internamente.
Eso te deja solo ~100-150 instrucciones útiles.

**Cada línea que añades dilluye las demás.**

Test para cada línea:
> "¿Cometería Claude un error sin esta instrucción?"
> Si la respuesta es no → borrala.

### 2. No copies lo que Claude ya sabe

Claude conoce PHP, Laravel, WordPress, JavaScript, etc.
No necesitas explicarle qué es un Model de Eloquent.
Solo dile **cómo lo usas TÚ** en este proyecto.

```markdown
❌ "Los Models de Eloquent representan tablas de la BD y permiten..."
✅ "Models en /app/Models. Siempre con $fillable y $casts definidos."
```

### 3. Apunta a archivos, no copies contenido

```markdown
❌ "El estilo de código es: uses tabs, max 120 chars, PSR-12..."
✅ "Formato: ejecutar ./vendor/bin/pint antes de cada commit."

❌ [pegar 50 líneas de ejemplos de código]
✅ "Para patrones de Service, ver app/Services/OrderService.php como referencia."
```

### 4. Documenta lo que Claude hace MAL, no lo que hace bien

Si Claude ya hace algo bien sin instrucción → no lo pongas.
Solo añade instrucciones cuando Claude comete errores específicos.

```markdown
❌ "Siempre usar Eloquent para queries" (Claude ya lo hace)
✅ "NUNCA usar DB::statement() sin parámetros preparados" (error real que cometió)
```

### 5. Es un documento vivo — actualízalo cuando Claude falla

```
# Flujo de actualización:
Claude comete error → tú corriges → 
"Añade a CLAUDE.md la regla para evitar esto" →
Claude escribe su propia regla
```

---

## Estructura recomendada

```markdown
# [Nombre del Proyecto]

## Stack
[Tecnologías con versiones. Breve.]

## Estructura
[Solo lo que no es obvio por el nombre de carpetas]

## Comandos esenciales
[Los 5-10 comandos que Claude necesitará ejecutar]

## Reglas de código
[Solo las que Claude incumple sin esta instrucción]

## Seguridad — jamás hacer
[Lista corta de prohibiciones absolutas]

## Skills disponibles
[Referencia a .claude/skills/ para tareas especializadas]
```

---

## Qué NO poner en CLAUDE.md

| ❌ No poner | ✅ Dónde va en cambio |
|-------------|----------------------|
| Guías completas de frameworks | Skills (`.claude/skills/`) |
| Código de ejemplo largo | Referencia a un archivo existente |
| Documentación de APIs externas | Skills o `docs/api.md` |
| Instrucciones de tareas específicas | Comandos (`.claude/commands/`) |
| Historial de decisiones | `docs/architecture.md` |
| Lecciones aprendidas | `tasks/lessons.md` |

---

## Skills vs CLAUDE.md

| | CLAUDE.md | Skills |
|---|---|---|
| **Carga** | Siempre, en cada sesión | Solo cuando son relevantes |
| **Uso** | Contexto universal del proyecto | Conocimiento especializado bajo demanda |
| **Tamaño** | < 200 líneas | Sin límite estricto |
| **Ejemplo** | Stack, comandos, reglas globales | Patrones Laravel, WordPress security, etc. |

### Cómo Claude carga un skill
```
# En CLAUDE.md tienes:
"Para desarrollo WordPress, ver .claude/skills/wordpress.md"

# Cuando pides algo de WordPress, Claude carga ese skill automáticamente
# y lo usa como contexto adicional sin saturar cada sesión
```

---

## CLAUDE.md jerárquico (proyectos complejos)

Puedes tener un CLAUDE.md en la raíz Y uno en subcarpetas:

```
proyecto/
├── CLAUDE.md              # Contexto global del proyecto
├── backend/
│   └── CLAUDE.md          # Solo reglas específicas del backend
└── frontend/
    └── CLAUDE.md          # Solo reglas específicas del frontend
```

Claude lee ambos y combina el contexto. El más específico (subcarpeta) tiene prioridad.

---

## Checklist de revisión mensual

Cada mes, revisa el CLAUDE.md con esta checklist:

- [ ] ¿Alguna instrucción es redundante porque Claude ya lo hace bien?
- [ ] ¿Hay instrucciones que deberían estar en un skill en lugar de aquí?
- [ ] ¿Los comandos siguen siendo correctos (versiones, paths)?
- [ ] ¿Las reglas de `tasks/lessons.md` más usadas deberían subir aquí?
- [ ] ¿Tiene más de 200 líneas? → Podar.

**Señal de que CLAUDE.md está demasiado largo:**
Claude empieza a ignorar instrucciones que están al medio del archivo,
o te hace preguntas cuya respuesta está escrita ahí.
