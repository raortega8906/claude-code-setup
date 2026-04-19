# Flujo de Trabajo Diario con Claude Code

## El ciclo correcto para cada tarea

```
1. PLANIFICAR  →  2. REVISAR PLAN  →  3. IMPLEMENTAR  →  4. VERIFICAR  →  5. APRENDER
```

Nunca saltar al paso 3 sin los pasos 1 y 2.

---

## Inicio de sesión

```bash
cd mi-proyecto
claude
```

Claude lee automáticamente tu `CLAUDE.md`. No necesitas repetir el contexto.

### Si continúas una tarea de ayer
```
Continúo con [tarea]. El estado actual está en tasks/todo.md.
Revisa los archivos modificados ayer antes de continuar.
```

### Si empiezas algo nuevo
```
Quiero implementar [descripción].
Entrevístame primero para entender los requisitos antes de planificar.
```

---

## Comandos de sesión esenciales

| Comando | Cuándo usarlo |
|---------|--------------|
| `/clear` | Entre tareas no relacionadas |
| `/compact` | Cuando el contexto es muy largo pero quieres continuar |
| `/init` | Al empezar un proyecto nuevo (genera CLAUDE.md base) |
| `/review` | Antes de hacer commit |
| `/test [archivo]` | Generar tests para código nuevo |
| `/security [código]` | Auditar antes de PR o despliegue |

---

## Flujo para una feature nueva

### Paso 1 — Planificar (sesión limpia)
```
Quiero implementar [funcionalidad]. 
Escribe el plan en tasks/todo.md con fases y subtareas.
No empieces a codificar hasta que confirme el plan.
```

### Paso 2 — Revisar el plan
Lee `tasks/todo.md`, modifica si algo está mal, luego:
```
El plan está bien. Empieza con la fase 1.
```

### Paso 3 — Implementar
Claude ejecuta. Tú supervisas. Si algo va mal dos veces:
```
/clear
```
Reescribe el prompt con más contexto.

### Paso 4 — Verificar
```
Verifica que los cambios funcionan correctamente.
Ejecuta los tests relevantes y muéstrame el output.
```

### Paso 5 — Registrar lección (si hubo correcciones)
```
Añade a tasks/lessons.md la regla para evitar [error que tuvimos].
```

---

## Gestión del contexto

### El contexto se degrada con el tiempo
Cuando una sesión es larga, Claude empieza a "olvidar" instrucciones tempranas.
Señales de que hay que limpiar:
- Claude ignora reglas del CLAUDE.md
- Vuelve a preguntar cosas que ya respondiste
- Genera código en el estilo incorrecto

**Solución:**
```bash
/clear   # limpia el contexto completamente
```
Luego resume el estado en pocas líneas.

### Compactar sin perder lo importante
```
/compact focus en: [lista de lo que importa]
Ejemplo: cambios en OrderService, estado del todo.md, archivos modificados
```

---

## Git y control de versiones

### Flujo de commits con Claude
```bash
# 1. Ver qué cambió
git diff --staged

# 2. Pedir a Claude un mensaje de commit descriptivo
# "Genera un mensaje de commit para estos cambios siguiendo Conventional Commits"

# 3. Commit
git commit -m "feat(orders): add discount code validation to order service"
```

### Conventional Commits (usar siempre)
```
feat:     nueva funcionalidad
fix:      corrección de bug
docs:     documentación
style:    formato (sin cambios de lógica)
refactor: refactorización
test:     añadir o corregir tests
chore:    tareas de mantenimiento (deps, config)
```

### Ramas
```bash
# Feature nueva
git checkout -b feat/nombre-descriptivo

# Fix urgente
git checkout -b fix/descripcion-del-bug

# NUNCA trabajar directamente en main/master
```

### Pre-commit: revisar antes de commitear
```
/review [archivos cambiados]
```
O:
```
Revisa los cambios staged (git diff --staged) antes de que haga commit.
Busca problemas de seguridad, bugs obvios y código que no sigue los patrones del proyecto.
```

---

## tasks/todo.md — plantilla

```markdown
# Tarea: [Nombre de la feature]
**Fecha inicio:** YYYY-MM-DD
**Estado:** En progreso

## Objetivo
[Una o dos frases describiendo qué hace esto y por qué]

## Fases

### Fase 1: [Nombre] ✅
- [x] Subtarea completada
- [x] Otra subtarea

### Fase 2: [Nombre] 🔄
- [x] Subtarea completada
- [ ] Subtarea pendiente
- [ ] Otra pendiente

### Fase 3: [Nombre] ⏳
- [ ] Por hacer

## Decisiones tomadas
- Usamos X en lugar de Y porque [razón]

## Resultados / Revisión
[Completar al finalizar]
```

---

## tasks/lessons.md — plantilla

```markdown
# Lecciones aprendidas

## YYYY-MM-DD — [Título breve]
**Problema:** qué salió mal exactamente  
**Causa:** por qué ocurrió  
**Solución:** cómo se arregló  
**Regla:** INSTRUCCIÓN CONCRETA para que Claude no repita esto  

---

## YYYY-MM-DD — Ejemplo: N+1 en Eloquent
**Problema:** Paginación de pedidos tardaba 8 segundos con 500 registros  
**Causa:** El controller cargaba relaciones user y products dentro de un loop  
**Solución:** Eager loading con `with(['user', 'products'])`  
**Regla:** En cualquier query de listado con relaciones, siempre usar eager loading. Antes de hacer una query con `->get()` verificar si hay relaciones que se van a acceder en el loop.  
```

---

## Sesiones paralelas (avanzado)

Para tareas independientes puedes abrir varias sesiones:
```bash
# Terminal 1 — trabajando en backend
cd mi-proyecto && claude

# Terminal 2 — trabajando en frontend (directorio diferente)
cd mi-proyecto/frontend && claude

# IMPORTANTE: que no editen los mismos archivos
```

O usar git worktrees para máximo aislamiento:
```bash
git worktree add ../mi-proyecto-feature-login feat/login
cd ../mi-proyecto-feature-login && claude
```
