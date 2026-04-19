# CLAUDE.md — Plantilla base para proyectos

> Copia este archivo a la raíz de cada proyecto y personalízalo.
> REGLA: Mantén este archivo bajo 200 líneas. Si crece más, mueve contenido a skills.

---

# Proyecto: [NOMBRE DEL PROYECTO]

## Stack principal
- **Backend:** Laravel [versión] / WordPress (plugin o tema hijo)
- **Frontend:** Astro [versión] / React / Next.js
- **Base de datos:** MySQL / MariaDB
- **Lenguajes:** PHP 8.x, TypeScript, JavaScript
- **Gestor paquetes PHP:** Composer
- **Gestor paquetes JS:** npm (NO usar yarn ni pnpm salvo que se indique)

## Estructura del proyecto
```
[Describe aquí la estructura específica de este proyecto]
Ejemplo:
- /app          → Lógica Laravel (Models, Controllers, Services)
- /resources    → Vistas Blade, assets
- /wp-content   → Plugins y temas WordPress
- /src          → Componentes Astro/React
```

## Comandos esenciales

### Laravel
```bash
php artisan serve          # servidor local
php artisan test           # ejecutar tests
php artisan migrate        # migraciones
php artisan cache:clear    # limpiar caché
./vendor/bin/pint          # formatear código (Laravel Pint)
./vendor/bin/phpstan analyse  # análisis estático
```

### WordPress
```bash
wp plugin activate [slug]  # WP-CLI
wp theme activate [slug]
wp cache flush
```

### Astro
```bash
npm run dev                # servidor local
npm run build              # build producción
npm run preview            # preview del build
npm run check              # TypeScript check
```

### React / Next.js
```bash
npm run dev
npm run build
npm run lint
npm run type-check
```

## Reglas de código

### PHP / Laravel
- PSR-12 estricto
- Tipado estricto: `declare(strict_types=1);` en todos los archivos PHP
- Usar `readonly` properties cuando sea posible (PHP 8.1+)
- Services para lógica de negocio, NO en Controllers
- Eloquent ORM siempre; raw SQL solo si hay justificación de rendimiento

### WordPress
- Código en funciones hijas SOLO en `functions.php` del tema hijo
- Hooks con prefijo del proyecto: `miprojecto_nombre_del_hook`
- Sanitizar SIEMPRE inputs: `sanitize_text_field()`, `absint()`, etc.
- Escapar SIEMPRE outputs: `esc_html()`, `esc_url()`, `esc_attr()`
- Nonces en todos los formularios y peticiones AJAX

### JavaScript / TypeScript
- TypeScript estricto: sin `any` explícito
- Componentes funcionales con hooks (React)
- Imports con paths absolutos cuando el proyecto lo configure

### Astro
- Preferir componentes `.astro` para contenido estático
- React solo para componentes con interactividad real (`client:load`)
- Imágenes siempre con el componente `<Image>` de Astro

## Seguridad — NUNCA hacer esto
- NUNCA commitear `.env`, claves API, passwords, tokens
- NUNCA usar `eval()` en PHP o JS
- NUNCA `echo $_GET['param']` sin sanitizar en WordPress
- NUNCA deshabilitar CSRF sin justificación documentada

## Skills disponibles
Para tareas especializadas, consulta los skills en `.claude/skills/`:
- `laravel.md` — Patrones específicos de Laravel
- `wordpress.md` — Desarrollo WordPress/WooCommerce
- `astro.md` — Componentes y optimización Astro
- `react-next.md` — React y Next.js patterns

## Gestión de tareas
1. Antes de codificar → escribe el plan en `tasks/todo.md`
2. Marca completado conforme avanzas
3. Al terminar → añade resumen en `tasks/todo.md`
4. Si Claude cometió un error y lo corregiste → añade la regla a `tasks/lessons.md`