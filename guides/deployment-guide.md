# Despliegue — Guía por tecnología

## Principios generales

1. **Nunca desplegar directamente a producción sin pasar por staging**
2. **Siempre backup de BD antes de migrar**
3. **Tener plan de rollback preparado antes de empezar**
4. **Verificar variables de entorno antes del despliegue**
5. **Monitorizar logs los primeros 10 minutos post-despliegue**

---

## Laravel — Despliegue

### Variables de entorno en producción
```bash
# Verificar que existen en el servidor (nunca en el repo)
APP_ENV=production
APP_DEBUG=false          # ← CRÍTICO: false en producción
APP_KEY=base64:...
DB_HOST=
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=
CACHE_DRIVER=redis       # redis o memcached, no file en producción
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis
MAIL_MAILER=smtp
# ... claves de servicios externos
```

### Script de despliegue (ejecutar en orden)
```bash
#!/bin/bash
set -e  # Parar si cualquier comando falla

echo "📦 1. Backup de base de datos"
php artisan db:backup  # o mysqldump directamente

echo "🔽 2. Poner en mantenimiento"
php artisan down --secret="mi-token-secreto"
# Acceso durante mantenimiento: /mi-token-secreto

echo "📥 3. Actualizar código"
git pull origin main

echo "📚 4. Instalar dependencias (sin dev)"
composer install --no-dev --optimize-autoloader

echo "🗃️ 5. Migraciones"
php artisan migrate --force  # --force requerido en producción

echo "🧹 6. Limpiar y cachear"
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache

echo "🔗 7. Permisos de storage"
chmod -R 775 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache

echo "⬆️ 8. Levantar aplicación"
php artisan up

echo "✅ Despliegue completado"
```

### Tests de humo post-despliegue
```bash
# Verificar que la app responde
curl -f https://miapp.com/health || echo "❌ App no responde"

# Verificar que la BD conecta
php artisan tinker --execute="DB::connection()->getPdo(); echo 'DB OK';"

# Verificar jobs en cola
php artisan queue:monitor
```

### Rollback
```bash
# Revertir última migración
php artisan migrate:rollback

# O restaurar backup de BD
mysql -u user -p database < backup_YYYYMMDD.sql

# Revertir código al commit anterior
git revert HEAD --no-edit
git push origin main
```

---

## WordPress — Despliegue

### Variables de entorno / wp-config.php
```php
// wp-config.php en producción — NUNCA en git
define('WP_DEBUG', false);
define('WP_DEBUG_LOG', false);
define('DISALLOW_FILE_EDIT', true);    // deshabilitar editor de código en admin
define('FORCE_SSL_ADMIN', true);
define('WP_AUTO_UPDATE_CORE', 'minor'); // solo updates de seguridad automáticos

// Claves únicas — generar en: https://api.wordpress.org/secret-key/1.1/salt/
define('AUTH_KEY', '...');
// etc.
```

### Checklist de despliegue WordPress
```bash
# 1. Backup completo
wp db export backup-$(date +%Y%m%d).sql
tar -czf wp-content-backup-$(date +%Y%m%d).tar.gz wp-content/

# 2. Subir cambios (solo los archivos del tema hijo o plugin)
# Nunca sobreescribir wp-config.php, wp-content/uploads/
rsync -avz --exclude='uploads/' wp-content/ servidor:/var/www/wp-content/

# 3. Actualizar permisos
find /var/www/wordpress -type f -exec chmod 644 {} \;
find /var/www/wordpress -type d -exec chmod 755 {} \;
chmod 600 /var/www/wordpress/wp-config.php

# 4. Limpiar caché
wp cache flush
wp rewrite flush

# 5. Activar/verificar plugins y tema
wp theme status
wp plugin status

# 6. Verificar que el sitio responde
curl -f https://misite.com || echo "❌ Sitio no responde"
```

### WP-CLI en producción (útil para migraciones de datos)
```bash
# Buscar y reemplazar URLs (staging → producción)
wp search-replace 'https://staging.misite.com' 'https://misite.com' --dry-run
wp search-replace 'https://staging.misite.com' 'https://misite.com'

# Crear usuario admin de emergencia
wp user create admin-emergency admin@misite.com --role=administrator --user_pass=TempPass123!
```

---

## Astro — Despliegue

### Build y verificación
```bash
npm run build          # genera /dist
npm run preview        # verifica el build localmente antes de subir

# Verificar que el build no tiene errores TypeScript
npm run check
```

### Despliegue estático (Netlify, Vercel, Cloudflare Pages)
```bash
# Netlify CLI
netlify build
netlify deploy --prod

# Variables de entorno en Netlify/Vercel:
# Se configuran en el dashboard, NO en el repo
```

### astro.config.mjs para producción
```javascript
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://misite.com',  // ← CRÍTICO para sitemap y URLs absolutas
  build: {
    inlineStylesheets: 'auto',
  },
  // ...integraciones
});
```

### Variables de entorno en Astro
```bash
# .env (en .gitignore)
PUBLIC_API_URL=https://api.misite.com   # PUBLIC_ → disponible en cliente
API_SECRET_KEY=...                       # sin PUBLIC_ → solo servidor

# En código
const apiUrl = import.meta.env.PUBLIC_API_URL;
const secret = import.meta.env.API_SECRET_KEY; // solo en archivos .astro o endpoints
```

---

## Next.js — Despliegue

### Vercel (recomendado para Next.js)
```bash
npm install -g vercel
vercel          # deploy preview
vercel --prod   # deploy producción
```

### Variables de entorno
```bash
# .env.local (en .gitignore)
NEXT_PUBLIC_API_URL=https://api.misite.com  # NEXT_PUBLIC_ → disponible en cliente
DATABASE_URL=...                             # solo servidor
NEXTAUTH_SECRET=...
```

### next.config.js para producción
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Headers de seguridad
  async headers() {
    return [{
      source: '/:path*',
      headers: [
        { key: 'X-Frame-Options', value: 'DENY' },
        { key: 'X-Content-Type-Options', value: 'nosniff' },
        { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
      ],
    }];
  },
  // Redirecciones
  async redirects() {
    return [
      { source: '/old-path', destination: '/new-path', permanent: true },
    ];
  },
};

module.exports = nextConfig;
```

---

## GitHub Actions — CI/CD completo

```yaml
# .github/workflows/deploy.yml
name: Deploy a Producción

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: |
          composer install
          php artisan test --parallel
          npm ci && npm run type-check && npm run test:run

  deploy:
    needs: test  # Solo despliega si los tests pasan
    runs-on: ubuntu-latest
    environment: production  # Requiere aprobación manual en GitHub
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/mi-proyecto
            git pull origin main
            composer install --no-dev --optimize-autoloader
            php artisan migrate --force
            php artisan optimize:clear
            php artisan config:cache
            php artisan route:cache

      - name: Notify on success
        run: echo "✅ Desplegado en producción"
```

### Secrets necesarios en GitHub
```
Settings → Secrets and variables → Actions:
- SERVER_HOST
- SERVER_USER  
- SSH_PRIVATE_KEY
- DB_PASSWORD (si se usa en scripts)
```
