Genera el checklist de despliegue para: $ARGUMENTS

El argumento debe indicar el entorno y el tipo de proyecto.
Ejemplos de argumentos:
  /deploy staging Laravel
  /deploy producción WordPress
  /deploy producción Astro Netlify
  /deploy staging Laravel + WordPress (monorepo)

Consulta docs/deployment.md para los comandos y procedimientos exactos de este proyecto.

Genera el checklist en este formato:

## Pre-despliegue
- [ ] Tests pasando en CI
- [ ] Backup de base de datos realizado
- [ ] Variables de entorno verificadas en servidor destino
- [ ] [otros específicos del proyecto]

## Despliegue (comandos exactos del proyecto)
- [ ] [comando 1]
- [ ] [comando 2]
- [ ] [...]

## Post-despliegue — verificación
- [ ] Respuesta HTTP 200 en URL principal
- [ ] [endpoints críticos a verificar]
- [ ] Logs sin errores durante 5 minutos

## Plan de rollback si algo falla
1. [paso 1]
2. [paso 2]

Adapta los comandos a lo que está documentado en docs/deployment.md.
Si el argumento no especifica el entorno, pregunta antes de generar el checklist.
