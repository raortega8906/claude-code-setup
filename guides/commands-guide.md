# Comandos personalizados de Claude Code

Los comandos viven en `.claude/commands/nombre.md`.
Se invocan con `/nombre` desde Claude Code.

---

## Cómo crear un comando

1. Crea `.claude/commands/mi-comando.md`
2. Escribe las instrucciones en lenguaje natural
3. Usa `$ARGUMENTS` para pasar parámetros opcionales
4. Úsalo con `/mi-comando [argumentos]`

---

## Comandos incluidos en esta plantilla

### `/review` — Revisión de código
**Archivo:** `.claude/commands/review.md`
```markdown
Actúa como un senior engineer revisando código.
Analiza: $ARGUMENTS

Revisa en este orden:
1. **Corrección**: ¿hace lo que dice que hace?
2. **Seguridad**: vulnerabilidades, exposición de datos, inputs sin sanitizar
3. **Rendimiento**: queries N+1, cálculos innecesarios, memory leaks
4. **Mantenibilidad**: claridad, duplicación, complejidad ciclomática
5. **Tests**: ¿qué edge cases no están cubiertos?

Para cada problema: línea exacta, descripción del riesgo y solución concreta.
No hagas comentarios positivos genéricos. Sé directo y útil.
```

### `/test` — Generar tests
**Archivo:** `.claude/commands/test.md`
```markdown
Escribe tests completos para: $ARGUMENTS

Sigue la estructura de tests del proyecto (ver CLAUDE.md).
Cubre:
- Happy path
- Edge cases y valores límite
- Casos de error esperados
- Si es Laravel: feature test + unit test por separado
- Si es React: renderizado, interacción de usuario, estados de carga/error

No escribas tests triviales (ej: "comprobar que el componente renderiza").
Cada test debe verificar comportamiento real.
```

### `/security` — Auditoría de seguridad
**Archivo:** `.claude/commands/security.md`
```markdown
Realiza una auditoría de seguridad de: $ARGUMENTS

Busca específicamente:
- Inyección: SQL, XSS, command injection
- Autenticación: tokens débiles, sesiones mal gestionadas
- Autorización: acceso a recursos sin verificar permisos
- Exposición de datos: logs, errores verbosos, campos sensibles en API
- CSRF: formularios sin nonces (WordPress) o sin tokens (Laravel)
- Dependencias: librerías desactualizadas con CVEs conocidos

Prioriza hallazgos: CRÍTICO / ALTO / MEDIO / BAJO
Para cada uno: qué es, por qué es un riesgo, cómo solucionarlo.
```

### `/deploy` — Checklist de despliegue
**Archivo:** `.claude/commands/deploy.md`
```markdown
Genera el checklist de despliegue para: $ARGUMENTS (staging o producción)

Incluye pasos específicos para el stack del proyecto:
- Backup de BD antes de migrar
- Variables de entorno a verificar
- Migraciones / actualizaciones de BD
- Limpieza de caché (Laravel, WordPress, CDN)
- Build de assets
- Verificación de permisos de archivos
- Tests de humo post-despliegue
- Plan de rollback si algo falla

Adapta según si es Laravel, WordPress o Astro/Next.js.
```

### `/explain` — Explicar código
**Archivo:** `.claude/commands/explain.md`
```markdown
Explica este código de forma clara: $ARGUMENTS

1. ¿Qué hace en términos de negocio?
2. ¿Cómo funciona técnicamente (flujo paso a paso)?
3. ¿Qué patrones de diseño usa?
4. ¿Qué podría salir mal o qué asunciones hace?

Público objetivo: desarrollador con experiencia pero que no conoce este código.
```

### `/refactor` — Refactorizar
**Archivo:** `.claude/commands/refactor.md`
```markdown
Refactoriza el siguiente código: $ARGUMENTS

Objetivos (sin cambiar el comportamiento externo):
- Aplicar principios SOLID donde aplique
- Eliminar duplicación (DRY)
- Mejorar nombres (variables, funciones, clases)
- Añadir tipos PHP 8.x / TypeScript donde falten
- Simplificar lógica compleja
- Extraer funciones/clases si hay responsabilidades múltiples

Explica cada cambio importante y por qué mejora el código.
Ejecuta los tests existentes para confirmar que nada se rompe.
```

### `/lesson` — Registrar lección aprendida
**Archivo:** `.claude/commands/lesson.md`
```markdown
Registra esta lección en tasks/lessons.md: $ARGUMENTS

Formato:
## [Fecha] — [Título breve del problema]
**Problema:** qué salió mal
**Causa raíz:** por qué pasó
**Solución:** cómo se arregló
**Regla para el futuro:** instrucción concreta para evitar que se repita

Después de registrar, confirma que la regla es suficientemente específica
para que Claude no cometa el mismo error en sesiones futuras.
```

---

## Cómo crear tus propios comandos

```markdown
# Ejemplo: .claude/commands/nuevo-modelo.md
Crea un modelo Laravel completo para la entidad: $ARGUMENTS

1. Migration con timestamps y soft deletes
2. Model con fillable, casts y relaciones básicas
3. Factory con datos realistas
4. Seeder
5. Resource API
6. Tests unitarios básicos

Sigue los patrones del archivo app/Models/User.php como referencia de estilo.
```

Después: `/nuevo-modelo Producto` → genera todo el scaffolding.
