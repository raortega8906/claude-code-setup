# Seguridad — Guía para Claude Code

## Reglas absolutas (no negociables)

```
NUNCA commitear: .env, claves API, passwords, tokens, certificados privados
NUNCA: eval() en PHP o JS
NUNCA: interpolación de variables en queries SQL sin preparar
NUNCA: echo de datos del usuario sin escapar (WordPress)
NUNCA: deshabilitar CSRF/nonces sin documentar el motivo
```

---

## .gitignore mínimo de seguridad

```gitignore
# Entorno
.env
.env.*
!.env.example

# Laravel
/bootstrap/cache/*.php
/storage/*.key
storage/app/
storage/logs/

# WordPress
wp-config.php
wp-content/uploads/
wp-content/cache/

# Dependencias (nunca en git)
/vendor/
/node_modules/

# Build
/public/build/
/dist/
/.next/

# Sistema
.DS_Store
Thumbs.db
*.log
```

---

## Gestión de secretos por entorno

### Desarrollo local
```bash
# .env (en .gitignore, NUNCA en git)
APP_KEY=base64:...
DB_PASSWORD=dev_password_local
STRIPE_SECRET=sk_test_...
```

### .env.example (SÍ en git — sin valores reales)
```bash
APP_KEY=
DB_PASSWORD=
STRIPE_SECRET=
STRIPE_WEBHOOK_SECRET=
MAIL_USERNAME=
MAIL_PASSWORD=
```

### Producción
- Variables de entorno del servidor (no archivos .env)
- GitHub Actions Secrets para CI/CD
- Nunca en el código fuente

---

## Seguridad por capa del stack

### Laravel
```php
// CSRF: automático con Laravel (verificar que el middleware está activo)
// En rutas API: usar Sanctum o Passport

// Rate limiting
Route::middleware(['throttle:api'])->group(function () {
    Route::post('/login', [AuthController::class, 'login']);
});

// Validación estricta
$validated = $request->validated(); // nunca $request->all() directamente

// Políticas de autorización
$this->authorize('update', $post); // en controller

// Encriptar datos sensibles
protected $casts = ['api_key' => 'encrypted'];

// Logging sin datos sensibles
Log::info('Usuario login', ['user_id' => $user->id]); // NO el password
```

### WordPress
```php
// Nonces en CADA formulario o acción AJAX
wp_nonce_field('miapp_accion', 'nonce');
check_admin_referer('miapp_accion', 'nonce'); // en handler

// Capacidades antes de cualquier acción
if (!current_user_can('manage_options')) {
    wp_die(__('Sin permisos.', 'mi-plugin'));
}

// Sanitizar TODO lo que entra
$email = sanitize_email($_POST['email'] ?? '');
$id    = absint($_GET['id'] ?? 0);

// Escapar TODO lo que sale
echo esc_html($titulo);
echo esc_url($enlace);
echo esc_attr($valor_input);

// SQL seguro
$wpdb->prepare("SELECT * FROM {$tabla} WHERE id = %d", $id);
```

### Astro / Next.js (frontend)
```typescript
// Validar en servidor, no solo en cliente
// app/api/contacto/route.ts
import { z } from 'zod';

const schema = z.object({
  email: z.string().email().max(254),
  mensaje: z.string().min(10).max(1000),
});

export async function POST(req: Request) {
  const body = await req.json();
  const result = schema.safeParse(body);
  
  if (!result.success) {
    return Response.json({ error: 'Datos inválidos' }, { status: 400 });
  }
  
  // Procesar con result.data (tipado y validado)
}

// Headers de seguridad en next.config.js
const securityHeaders = [
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
];
```

---

## Pre-commit hooks (automatizar la seguridad)

### Instalar
```bash
npm install --save-dev husky lint-staged
npx husky init
```

### .husky/pre-commit
```bash
#!/bin/sh
# Buscar posibles secrets antes de cada commit
if git diff --cached --name-only | xargs grep -l "sk_live_\|pk_live_\|AKIA\|password.*=.*['\"][^'\"]\{8\}" 2>/dev/null; then
    echo "⚠️  Posible secret detectado. Revisa los archivos antes de commitear."
    exit 1
fi

# Ejecutar tests
npm run type-check
```

### Herramienta adicional: detect-secrets
```bash
pip install detect-secrets
detect-secrets scan > .secrets.baseline
detect-secrets audit .secrets.baseline
# Añadir al pre-commit hook
```

---

## Auditoría de dependencias

```bash
# Laravel
composer audit                    # vulnerabilidades en paquetes PHP
composer outdated                 # paquetes desactualizados

# Node.js
npm audit                         # vulnerabilidades
npm audit fix                     # fix automático (no breaking)
npm outdated                      # versiones disponibles

# WordPress
wp plugin list --update=available # plugins con actualizaciones
```

### En CI/CD (GitHub Actions)
```yaml
- name: Security audit
  run: |
    composer audit --no-dev
    npm audit --audit-level=high
```

---

## Prompt de auditoría de seguridad para Claude

Cuando quieras que Claude audite tu código:

```
/security [archivo o fragmento de código]
```

O manualmente:
```
Audita este código buscando:
1. Inyección SQL, XSS, command injection
2. Datos sensibles expuestos (logs, responses, error messages)
3. Autenticación/autorización incorrecta
4. Inputs sin validar o sanitizar
5. CSRF/nonces faltantes
6. Secretos hardcodeados

Prioriza: CRÍTICO > ALTO > MEDIO > BAJO
Para cada problema: qué línea, qué riesgo, cómo arreglarlo.
```
