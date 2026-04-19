# Testing — Guía por tecnología

## Filosofía de testing

**Testa comportamiento, no implementación.**
Los tests deben sobrevivir a refactorizaciones internas.
Si un test falla porque renombraste una variable interna (sin cambiar el resultado), el test estaba mal escrito.

Prioridad de lo que testear:
1. Lógica de negocio crítica (Services, cálculos, validaciones)
2. Endpoints de API (inputs, outputs, errores)
3. Seguridad y autorización
4. Edge cases que han causado bugs reales

**NO testear:**
- Getters/setters triviales
- Framework internals (Laravel, WordPress ya los testan ellos)
- Que un componente React "renderiza sin errores" sin más

---

## Laravel — Testing

### Configuración
```bash
# Instalar PestPHP (preferido sobre PHPUnit vanilla)
composer require pestphp/pest --dev
composer require pestphp/pest-plugin-laravel --dev
php artisan pest:install
```

### Estructura
```
tests/
├── Unit/
│   ├── Services/
│   │   └── OrderServiceTest.php
│   └── Models/
│       └── ProductTest.php
└── Feature/
    ├── Api/
    │   ├── OrderApiTest.php
    │   └── AuthApiTest.php
    └── Admin/
        └── ProductManagementTest.php
```

### Patrones — Feature Tests
```php
<?php
// tests/Feature/Api/OrderApiTest.php

use App\Models\Order;
use App\Models\User;

beforeEach(function () {
    // Usar base de datos en memoria para tests
    $this->artisan('migrate:fresh');
});

it('creates an order with valid data', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user, 'sanctum')
        ->postJson('/api/v1/orders', [
            'product_id' => 1,
            'quantity'   => 2,
        ]);

    $response->assertCreated()
        ->assertJsonStructure([
            'data' => ['id', 'status', 'total', 'created_at']
        ]);

    expect(Order::count())->toBe(1);
    expect(Order::first()->user_id)->toBe($user->id);
});

it('returns 422 with invalid quantity', function () {
    $user = User::factory()->create();

    $this->actingAs($user, 'sanctum')
        ->postJson('/api/v1/orders', ['product_id' => 1, 'quantity' => -1])
        ->assertUnprocessable()
        ->assertJsonValidationErrors(['quantity']);
});

it('returns 401 for unauthenticated requests', function () {
    $this->postJson('/api/v1/orders', [])->assertUnauthorized();
});

it('returns 403 when user tries to view another users order', function () {
    $owner = User::factory()->create();
    $other = User::factory()->create();
    $order = Order::factory()->for($owner)->create();

    $this->actingAs($other, 'sanctum')
        ->getJson("/api/v1/orders/{$order->id}")
        ->assertForbidden();
});
```

### Patrones — Unit Tests
```php
<?php
// tests/Unit/Services/OrderServiceTest.php

use App\Services\OrderService;

it('calculates total with percentage discount', function () {
    $service = new OrderService();

    $total = $service->calculateTotal(
        subtotal: 100.00,
        discountCode: 'SAVE10'
    );

    expect($total)->toBe(90.00);
});

it('rejects expired discount codes', function () {
    $service = new OrderService();

    expect(fn () => $service->calculateTotal(100.00, 'EXPIRED2023'))
        ->toThrow(\App\Exceptions\InvalidDiscountException::class);
});
```

### Factories — datos realistas
```php
// database/factories/OrderFactory.php
class OrderFactory extends Factory
{
    public function definition(): array
    {
        return [
            'user_id'    => User::factory(),
            'status'     => $this->faker->randomElement(['pending', 'processing', 'completed']),
            'total'      => $this->faker->randomFloat(2, 10, 500),
            'notes'      => $this->faker->optional()->sentence(),
        ];
    }

    // Estados reutilizables
    public function completed(): static
    {
        return $this->state(['status' => 'completed']);
    }

    public function pending(): static
    {
        return $this->state(['status' => 'pending']);
    }
}
```

### Comandos
```bash
php artisan test                           # todos los tests
php artisan test --filter="OrderApi"       # filtrar por nombre
php artisan test --group=api               # por grupo
php artisan test --coverage --min=80       # cobertura mínima 80%
php artisan test --parallel               # más rápido
```

---

## WordPress — Testing

### Configuración con WP_Mock (para unit tests sin WordPress completo)
```bash
composer require --dev 10up/wp_mock
composer require --dev phpunit/phpunit
```

### WP_Mock — testear funciones con hooks
```php
<?php
use WP_Mock\Tools\TestCase;

class MiPluginTest extends TestCase
{
    public function setUp(): void
    {
        parent::setUp();
        WP_Mock::setUp();
    }

    public function tearDown(): void
    {
        WP_Mock::tearDown();
        parent::tearDown();
    }

    public function test_sanitizes_user_input(): void
    {
        WP_Mock::userFunction('sanitize_text_field')
            ->once()
            ->with('<script>alert(1)</script>')
            ->andReturn('alert1');

        $resultado = miapp_procesar_input('<script>alert(1)</script>');

        $this->assertEquals('alert1', $resultado);
    }

    public function test_hook_registrado(): void
    {
        WP_Mock::expectActionAdded('init', 'miapp_init_function');
        miapp_register_hooks();
        WP_Mock::assertActionsCalled();
    }
}
```

---

## Astro — Testing

### Configuración con Vitest
```bash
npm install --save-dev vitest @testing-library/dom
```

### Testear utilidades y helpers
```typescript
// src/utils/formatDate.test.ts
import { describe, it, expect } from 'vitest';
import { formatDate } from './formatDate';

describe('formatDate', () => {
  it('formats date in Spanish locale', () => {
    const date = new Date('2024-01-15');
    expect(formatDate(date)).toBe('15 de enero de 2024');
  });

  it('handles invalid dates gracefully', () => {
    expect(() => formatDate(new Date('invalid'))).toThrow();
  });
});
```

### Comandos
```bash
npm run test          # vitest watch
npm run test:run      # una sola vez (para CI)
npm run test:coverage
```

---

## React / Next.js — Testing

### Configuración con Vitest + Testing Library
```bash
npm install --save-dev vitest @testing-library/react @testing-library/user-event jsdom
```

### Testear componentes
```tsx
// components/Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  it('calls onClick when clicked', async () => {
    const handleClick = vi.fn();
    render(<Button label="Guardar" onClick={handleClick} />);

    await userEvent.click(screen.getByRole('button', { name: 'Guardar' }));

    expect(handleClick).toHaveBeenCalledOnce();
  });

  it('shows loading state and disables button', () => {
    render(<Button label="Guardar" onClick={() => {}} isLoading />);

    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
    expect(button).toHaveAttribute('aria-busy', 'true');
    expect(screen.getByText('Cargando...')).toBeInTheDocument();
  });

  it('does not call onClick when disabled', async () => {
    const handleClick = vi.fn();
    render(<Button label="Guardar" onClick={handleClick} disabled />);

    await userEvent.click(screen.getByRole('button'));
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

### Testear custom hooks
```tsx
// hooks/useProductos.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { useProductos } from './useProductos';

// Mock fetch global
global.fetch = vi.fn();

it('loads products on mount', async () => {
  vi.mocked(fetch).mockResolvedValueOnce({
    ok: true,
    json: async () => [{ id: 1, nombre: 'Producto A', precio: 10 }],
  } as Response);

  const { result } = renderHook(() => useProductos());

  expect(result.current.isLoading).toBe(true);

  await waitFor(() => expect(result.current.isLoading).toBe(false));

  expect(result.current.productos).toHaveLength(1);
  expect(result.current.error).toBeNull();
});
```

### Next.js — Testear API Routes
```typescript
// app/api/contacto/route.test.ts
import { POST } from './route';

it('returns 400 with missing email', async () => {
  const request = new Request('http://localhost/api/contacto', {
    method: 'POST',
    body: JSON.stringify({ mensaje: 'Hola' }), // sin email
  });

  const response = await POST(request);
  expect(response.status).toBe(400);
});

it('returns 200 with valid data', async () => {
  const request = new Request('http://localhost/api/contacto', {
    method: 'POST',
    body: JSON.stringify({ email: 'test@test.com', mensaje: 'Hola mundo' }),
  });

  const response = await POST(request);
  expect(response.status).toBe(200);
});
```

---

## CI — GitHub Actions para tests automáticos

```yaml
# .github/workflows/tests.yml
name: Tests

on: [push, pull_request]

jobs:
  laravel:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with: { php-version: '8.3' }
      - run: composer install --no-interaction
      - run: cp .env.example .env && php artisan key:generate
      - run: php artisan test --parallel

  frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run type-check
      - run: npm run test:run
      - run: npm run build
```
