# Skill: Laravel Development

> Claude carga este skill cuando trabaja en código Laravel.
> Referencia para patrones, convenciones y decisiones arquitectónicas del proyecto.

## Versión y requisitos
- Laravel: ver `composer.json`
- PHP: 8.2+ con strict_types
- Herramientas de calidad: Laravel Pint (formato), PHPStan nivel 8 (análisis)

## Arquitectura del proyecto Laravel

```
app/
├── Console/Commands/     # Artisan commands
├── Exceptions/           # Exception handlers personalizados
├── Http/
│   ├── Controllers/      # Thin controllers — solo HTTP logic
│   ├── Middleware/
│   ├── Requests/         # FormRequests para toda validación
│   └── Resources/        # API Resources para transformar datos
├── Models/               # Eloquent models
├── Policies/             # Autorización
├── Services/             # Lógica de negocio (aquí va el grueso)
├── Repositories/         # Abstracción de queries complejas (opcional)
└── Events/ Listeners/    # Sistema de eventos
```

## Reglas de arquitectura

### Controllers
- SOLO lógica HTTP: recibir request, llamar service, devolver response
- Máximo 5-7 líneas por método si es posible
- Usar FormRequest para validación (NUNCA `$request->validate()` inline en controller grande)
- Usar API Resources para transformar datos

```php
// ✅ Correcto
public function store(CreateOrderRequest $request): JsonResponse
{
    $order = $this->orderService->create($request->validated());
    return new JsonResponse(new OrderResource($order), 201);
}

// ❌ Incorrecto — lógica de negocio en controller
public function store(Request $request): JsonResponse
{
    $validated = $request->validate([...]);
    $order = Order::create($validated);
    Mail::to($order->user)->send(new OrderConfirmation($order));
    // ...20 líneas más
}
```

### Services
- Toda la lógica de negocio vive aquí
- Un Service por dominio (OrderService, UserService, etc.)
- Inyección de dependencias vía constructor
- Métodos pequeños y con nombre descriptivo

### Models
- Definir `$fillable` o `$guarded` explícitamente
- Definir `$casts` para tipos (incluido `encrypted:` para datos sensibles)
- Relaciones con tipos de retorno
- Scopes para queries reutilizables

```php
// ✅ Correcto
protected $casts = [
    'email_verified_at' => 'datetime',
    'metadata' => 'array',
    'api_key' => 'encrypted',
    'is_active' => 'boolean',
];

public function scopeActive(Builder $query): Builder
{
    return $query->where('is_active', true);
}
```

### Migrations
- Siempre incluir `down()` funcional
- Indexes en columnas de búsqueda frecuente
- Foreign keys con `constrained()` y `cascadeOnDelete()` cuando aplique
- Nunca modificar migrations ya ejecutadas en producción — crear nueva migration

### Validación
- FormRequest para toda validación de entrada
- Mensajes de error personalizados en el mismo FormRequest
- Autorización en `authorize()` del FormRequest

### Autorización
- Policies para toda lógica de autorización
- Gates para permisos simples globales
- NUNCA lógica de autorización en controllers o views

## Testing en Laravel

### Estructura
```
tests/
├── Unit/
│   ├── Services/         # Test cada Service en aislamiento
│   └── Models/           # Scopes, casts, relaciones
└── Feature/
    ├── Api/              # Tests de endpoints completos
    └── [Domain]/         # Tests de flujos completos
```

### Patrones
```php
// Feature test — probar el endpoint completo
it('creates an order successfully', function () {
    $user = User::factory()->create();
    
    $response = $this->actingAs($user)
        ->postJson('/api/orders', [
            'product_id' => 1,
            'quantity' => 2,
        ]);
    
    $response->assertCreated()
        ->assertJsonStructure(['data' => ['id', 'status', 'total']]);
    
    $this->assertDatabaseHas('orders', ['user_id' => $user->id]);
});

// Unit test — probar Service en aislamiento
it('calculates order total with discount', function () {
    $service = new OrderService();
    $total = $service->calculateTotal(items: $items, discountCode: 'SAVE10');
    expect($total)->toBe(90.00);
});
```

### Comandos de testing
```bash
php artisan test                          # todos los tests
php artisan test --filter=OrderTest      # test específico
php artisan test --coverage              # con cobertura
php artisan test --parallel              # en paralelo (más rápido)
```

## Seguridad Laravel

### Variables de entorno
- Nunca valores sensibles en código
- Usar `config()` en lugar de `env()` fuera de archivos de config
- Validar variables críticas al inicio: `php artisan config:clear`

### Queries
- Eloquent ORM o Query Builder parametrizado
- Si es raw SQL: `DB::select('SELECT * FROM users WHERE id = ?', [$id])`
- NUNCA: `DB::select("SELECT * FROM users WHERE id = $id")`

### API
- Rate limiting en rutas públicas
- Sanctum o Passport para autenticación API
- Versionar APIs: `/api/v1/`

## Comandos de desarrollo frecuentes

```bash
# Generar componentes
php artisan make:model Product -mfsc    # Model + migration + factory + seeder + controller
php artisan make:request StoreProductRequest
php artisan make:policy ProductPolicy --model=Product
php artisan make:resource ProductResource
php artisan make:service ProductService  # requiere paquete o manual

# Base de datos
php artisan migrate:fresh --seed        # reset completo (solo desarrollo)
php artisan db:seed --class=ProductSeeder

# Caché y optimización
php artisan optimize:clear              # limpiar todo
php artisan route:cache                 # caché de rutas (producción)
php artisan config:cache                # caché de config (producción)
```
