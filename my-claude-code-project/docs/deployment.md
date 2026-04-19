# Procedimientos de Despliegue

> Referencia rápida para despliegues. Copia los comandos exactos sin tener que recordarlos.
> Mantén esto actualizado cuando cambien los servidores o el proceso.

---

## Entornos

| Entorno | URL | Servidor | Branch |
|---------|-----|----------|--------|
| Local | http://miproyecto.test | Docker / Herd | cualquiera |
| Staging | https://staging.miproyecto.com | VPS-staging (IP: x.x.x.x) | `develop` |
| Producción | https://miproyecto.com | VPS-prod (IP: x.x.x.x) | `main` |

**Acceso SSH:**
```bash
ssh usuario@ip-servidor -i ~/.ssh/mi-clave
```

---

## Despliegue en Staging

Staging se despliega automáticamente vía GitHub Actions al hacer push a `develop`.
Ver `.github/workflows/ci.yml`.

Para despliegue manual:
```bash
ssh usuario@staging-ip
cd /var/www/miproyecto

git pull origin develop
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan optimize:clear && php artisan config:cache && php artisan route:cache
```

---

## Despliegue en Producción — Laravel

### Prerrequisitos
- [ ] Tests pasando en staging
- [ ] PR revisado y aprobado
- [ ] Avisar al equipo (si hay)
- [ ] Momento de bajo tráfico (fuera de horas pico)

### Procedimiento completo

```bash
# 1. Conectar al servidor
ssh usuario@prod-ip
cd /var/www/miproyecto

# 2. Backup de base de datos ANTES de cualquier cambio
mysqldump -u dbuser -p dbname > /backups/backup-$(date +%Y%m%d-%H%M).sql
# Verificar que el backup existe y tiene tamaño razonable
ls -lh /backups/backup-*.sql | tail -1

# 3. Modo mantenimiento (los usuarios ven página de mantenimiento)
php artisan down --secret="token-secreto-para-pasar-mantenimiento"
# Durante el mantenimiento, tú puedes acceder desde:
# https://miproyecto.com/token-secreto-para-pasar-mantenimiento

# 4. Actualizar código
git pull origin main

# 5. Dependencias PHP (sin paquetes de desarrollo)
composer install --no-dev --optimize-autoloader --no-interaction

# 6. Migraciones
php artisan migrate --force

# 7. Assets frontend (si hay build)
npm ci --omit=dev
npm run build

# 8. Limpiar y cachear configuración
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache

# 9. Permisos de storage
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache

# 10. Levantar la aplicación
php artisan up

# 11. Verificación rápida
curl -s -o /dev/null -w "%{http_code}" https://miproyecto.com
# Debe devolver 200
```

### Verificación post-despliegue
```bash
# Comprobar que no hay errores en los logs (primeros 5 minutos)
tail -f storage/logs/laravel.log

# Verificar que los workers de cola están corriendo
php artisan queue:monitor

# Test de humo: endpoints críticos
curl -f https://miproyecto.com/api/v1/health
curl -f https://miproyecto.com/api/v1/productos
```

---

## Despliegue en Producción — WordPress

### Qué SE despliega (solo cambios propios)
```
wp-content/themes/mi-tema-hijo/   ← sí
wp-content/plugins/mi-plugin/     ← sí
```

### Qué NO se despliega
```
wp-config.php                     ← nunca (config específica del servidor)
wp-content/uploads/               ← nunca (archivos de usuarios)
WordPress core (/wp-admin, etc.)  ← se actualiza desde el panel o WP-CLI
```

### Procedimiento
```bash
# 1. Backup
ssh usuario@prod-ip
cd /var/www/wordpress

wp db export /backups/wp-backup-$(date +%Y%m%d).sql
tar -czf /backups/wpcontent-$(date +%Y%m%d).tar.gz wp-content/ \
  --exclude=wp-content/uploads \
  --exclude=wp-content/cache

# 2. Subir cambios (desde local o desde CI)
rsync -avz --delete \
  wp-content/themes/mi-tema-hijo/ \
  usuario@prod-ip:/var/www/wordpress/wp-content/themes/mi-tema-hijo/

rsync -avz --delete \
  wp-content/plugins/mi-plugin/ \
  usuario@prod-ip:/var/www/wordpress/wp-content/plugins/mi-plugin/

# 3. En el servidor: limpiar caché
wp cache flush
wp rewrite flush

# 4. Verificar que el sitio responde
curl -f https://miproyecto.com
```

---

## Despliegue en Producción — Astro / Frontend estático

Astro genera archivos estáticos en `/dist`. Se despliega en Netlify/Vercel/Cloudflare Pages.

### Con Netlify CLI
```bash
# Desde local o CI
npm run build          # genera /dist
netlify deploy --prod  # despliega a producción

# Verificar URL de previsualización antes de ir a prod
netlify deploy         # genera preview URL
# revisar → si está bien:
netlify deploy --prod
```

### Variables de entorno en Netlify
Se configuran en el dashboard de Netlify, NO en el repo.
```
PUBLIC_API_URL=https://api.miproyecto.com
# ... otras variables PUBLIC_*
```

### Verificación
```bash
# Comprobar que el build fue correcto
curl -f https://miproyecto.com
curl -f https://miproyecto.com/sitemap.xml
# Verificar en DevTools que no hay errores de consola
```

---

## Plan de Rollback

### Laravel — rollback de código
```bash
# Ver commits recientes
git log --oneline -10

# Revertir al commit anterior
git revert HEAD --no-edit
git push origin main
# O si fue un deploy urgente:
git reset --hard HEAD~1
git push origin main --force  # ⚠️ solo si nadie más ha pusheado

# Rollback de migración (si la última migración causó el problema)
php artisan migrate:rollback
```

### Laravel — restaurar backup de BD
```bash
# Listar backups disponibles
ls -lh /backups/*.sql

# Restaurar (la app debe estar en modo mantenimiento)
php artisan down
mysql -u dbuser -p dbname < /backups/backup-YYYYMMDD-HHMM.sql
php artisan up
```

### WordPress — rollback
```bash
# Restaurar archivos
rsync -avz /backups/wpcontent-YYYYMMDD.tar.gz . && tar -xzf wpcontent-YYYYMMDD.tar.gz

# Restaurar BD
wp db import /backups/wp-backup-YYYYMMDD.sql
wp cache flush
```

### Astro — rollback en Netlify
En el dashboard de Netlify: `Deploys → [deploy anterior] → Publish deploy`
Es instantáneo, sin comandos.

---

## Contactos de emergencia

| Rol | Nombre | Contacto |
|-----|--------|----------|
| Hosting/servidor | [proveedor] | [email/teléfono] |
| Dominio/DNS | [registrador] | [acceso panel] |
| CDN | [Cloudflare/Netlify] | [cuenta] |
