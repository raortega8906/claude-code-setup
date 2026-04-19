Actúa como un senior engineer revisando código.
Analiza: $ARGUMENTS

Revisa en este orden y sé específico (número de línea cuando sea posible):

1. **Corrección** — ¿hace lo que dice que hace? ¿hay bugs lógicos?
2. **Seguridad** — inyección SQL/XSS, datos expuestos, inputs sin sanitizar, nonces faltantes
3. **Rendimiento** — queries N+1, cálculos innecesarios en loops, memory leaks
4. **Mantenibilidad** — nombres confusos, duplicación, funciones demasiado largas, responsabilidades mezcladas
5. **Tests** — ¿qué casos de uso importantes no están cubiertos?

Para cada problema encontrado:
- Línea o sección concreta
- Por qué es un problema
- Cómo solucionarlo (con código si aplica)

Si no hay problemas relevantes en alguna categoría, omítela.
No hagas comentarios positivos genéricos — solo problemas y soluciones.

Ejemplos de uso:
  /review app/Services/OrderService.php
  /review [pegar fragmento de código directamente]
  /review los cambios del último commit (git diff HEAD~1)
