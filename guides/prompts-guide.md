# Guía de Prompts Efectivos para Claude Code

## Principio fundamental

**No microgestiones el cómo. Describe el qué y el resultado esperado.**

❌ Mal: "Crea un archivo Controller, luego añade el método store, luego valida los campos nombre y email..."  
✅ Bien: "Crea el flujo completo para registrar usuarios: validación, creación en BD y email de bienvenida. Sigue los patrones del proyecto."

---

## 1. Prompts de planificación (SIEMPRE primero)

### Iniciar una feature nueva
```
Quiero implementar [descripción breve]. 
Entrevístame con AskUserQuestion para entender bien los requisitos:
implementación técnica, edge cases, seguridad y tradeoffs.
No hagas preguntas obvias, profundiza en lo que podría salir mal.
Cuando tengamos todo claro, escribe la especificación completa en tasks/todo.md.
```

### Revisar plan antes de ejecutar
```
Antes de escribir código, escribe el plan paso a paso en tasks/todo.md.
Espera mi aprobación antes de empezar la implementación.
```

### Segunda opinión del plan
```
Actúa como un senior engineer revisando este plan.
Identifica riesgos, problemas de arquitectura y lo que falta.
Sé crítico.
[pega el plan]
```

---

## 2. Prompts de desarrollo — Laravel

### Crear feature completa
```
Implementa el módulo de [nombre] en Laravel:
- Migration con los campos [lista]
- Model con relaciones, scopes y casts necesarios
- FormRequest con validación completa
- Controller resource con autorización por Policy
- Service class para la lógica de negocio
- Tests unitarios para el Service y feature tests para los endpoints
Sigue la estructura existente del proyecto.
```

### Refactorizar código legacy
```
Refactoriza este código para seguir los principios SOLID y los patrones 
del proyecto. No cambies el comportamiento externo. Añade tipos PHP 8.x.
Explica cada decisión importante.
[pega el código]
```

### Debugging
```
Este código lanza [error exacto]. 
Analiza la causa raíz, no el síntoma. 
Encuentra y arregla el problema de verdad.
[pega el error + código relevante]
```

---

## 3. Prompts de desarrollo — WordPress

### Crear plugin
```
Crea un plugin WordPress para [funcionalidad].
- Estructura de carpetas estándar
- Clase principal con singleton pattern
- Hooks con prefijo [mi-prefijo]_
- Sanitización de todos los inputs
- Escapado de todos los outputs
- Nonces donde haya formularios o AJAX
- Sin dependencias externas si es posible
```

### Añadir funcionalidad al tema hijo
```
En el functions.php del tema hijo, añade [funcionalidad].
- Usa el hook correcto (no hardcodees prioridades sin motivo)
- Prefija funciones con [prefijo]_
- Sanitiza inputs, escapa outputs
- Comenta el propósito del hook
```

### WooCommerce
```
Extiende WooCommerce para [funcionalidad].
- Usa los hooks nativos de WooCommerce, no sobrescribas templates si hay hook disponible
- Revisa primero si WooCommerce ya tiene esto nativo
- Documenta qué versión mínima de WooCommerce requiere
```

---

## 4. Prompts de desarrollo — Astro

### Crear página o sección
```
Crea [componente/página] en Astro.
- Contenido estático → componente .astro puro
- Si necesita interactividad → componente React con client:load solo donde sea imprescindible
- Usa el componente Image de Astro para imágenes
- Accesibilidad: roles ARIA, alt texts, contraste
- SEO: meta tags, structured data si aplica
```

### Optimización de rendimiento
```
Audita este componente Astro para rendimiento:
- Identificar renders del cliente innecesarios
- Islands architecture: ¿qué debería ser estático?
- Carga de imágenes y fuentes
- Core Web Vitals que podrían verse afectados
[pega el componente]
```

---

## 5. Prompts de desarrollo — React / Next.js

### Crear componente
```
Crea un componente React para [descripción].
Stack: TypeScript estricto, sin any explícito.
- Props tipadas con interface
- Manejo de estados de carga y error
- Accesibilidad (ARIA donde aplique)
- Sin lógica de negocio en el componente, solo presentación
```

### Next.js — Ruta o página
```
Crea la ruta [ruta] en Next.js con App Router.
- Server Component por defecto
- Client Component solo si necesita estado/interactividad
- Metadata para SEO
- Loading y error boundaries
- Tipos TypeScript para todos los datos
```

---

## 6. Prompts de testing

### Generar tests completos
```
Escribe los tests para [módulo/función].
Cubre:
1. Happy path (caso normal)
2. Edge cases (valores límite, vacíos, nulos)
3. Casos de error (qué debe fallar y cómo)
4. Si es Laravel: feature test + unit test separados
No hagas tests triviales que no aporten valor.
```

### Revisar cobertura
```
Analiza este código e identifica qué escenarios NO están cubiertos por tests.
Luego escribe los tests que faltan.
[pega el código y los tests existentes]
```

---

## 7. Prompts de seguridad

### Auditoría de seguridad
```
Audita este código buscando vulnerabilidades:
- Inyección SQL / XSS / CSRF
- Exposición de datos sensibles
- Autenticación y autorización
- Validación de inputs
- Dependencias con vulnerabilidades conocidas
Prioriza por severidad y propón fixes concretos.
[pega el código]
```

### Revisión de PR/commit
```
Revisa estos cambios como si fueras un security engineer.
Busca problemas de seguridad, no solo bugs.
Sé específico sobre qué línea tiene el problema y por qué es un riesgo.
[pega el diff]
```

---

## 8. Prompts de despliegue

### Checklist de despliegue
```
Genera un checklist de despliegue para [entorno: staging/producción].
Incluye: migraciones, caché, assets, variables de entorno, rollback plan.
Basado en el stack del proyecto.
```

---

## 9. Prompts de gestión de sesión

### Compactar sin perder contexto
```
/compact focus en [lista de lo que importa conservar: cambios en X archivo, 
estado actual de la feature Y, lista de archivos modificados]
```

### Cuando Claude comete el mismo error dos veces
1. Usa `/clear`
2. Reescribe el prompt incorporando lo que aprendiste
3. Después: `Añade una regla a tasks/lessons.md para que esto no vuelva a pasar`

---

## 10. Patrones de prompts avanzados

### Desafiar a Claude
```
Pruébame que esto funciona. Grillame sobre estos cambios 
y no me presentes el resultado hasta que pases mis preguntas.
```

### Solución elegante (tras un fix mediocre)
```
Sabiendo todo lo que sabes ahora sobre este problema, 
descarta la solución actual e implementa la solución elegante y limpia.
```

### Exploración de codebase desconocido
```
¿Qué mejorarías de este archivo? No me pidas permiso, 
dame tu opinión directa con problemas y propuestas concretas.
```
