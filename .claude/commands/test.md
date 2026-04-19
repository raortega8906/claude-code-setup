Escribe tests completos para: $ARGUMENTS

Ejemplos de uso:
  /test app/Services/OrderService.php
  /test el método calculateTotal del OrderService
  /test el componente Button de React
  /test el endpoint POST /api/v1/pedidos

Antes de escribir los tests, identifica:
1. Qué hace exactamente el código
2. Qué casos de uso importantes existen

Luego escribe tests cubriendo obligatoriamente:

**Happy path**
- El caso normal y esperado con datos válidos

**Edge cases**
- Valores límite (0, null, string vacío, array vacío)
- Valores extremos (número muy grande, string muy largo)
- Combinaciones inusuales pero válidas

**Casos de error**
- Qué debe fallar y con qué error exacto
- Autenticación requerida pero ausente
- Autorización: usuario sin permisos
- Datos inválidos: qué validación salta

**Específico por tipo:**

Si es Laravel (PHP/Pest):
- Feature test para el endpoint completo (actingAs, postJson, assertCreated...)
- Unit test para la lógica del Service en aislamiento
- Usar factories para datos de prueba

Si es React (Vitest + Testing Library):
- Renderizado con distintas combinaciones de props
- Interacción de usuario (userEvent.click, userEvent.type)
- Estados de carga y error
- Accesibilidad (roles ARIA correctos)

Si es una función utilitaria (TypeScript):
- Todos los tipos de entrada posibles
- Comportamiento con tipos incorrectos

No escribas tests triviales como "el componente renderiza sin errores".
Ejecuta los tests al final y muestra el resultado.
