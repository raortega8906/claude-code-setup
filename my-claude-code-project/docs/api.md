# Contratos de API

> Documenta aquí los endpoints de la API del proyecto.
> Claude usa este archivo para saber exactamente qué devuelve cada endpoint
> y así generar código de frontend correcto sin adivinar estructuras.

**Convenciones de este proyecto:**
- Base URL: `https://api.miproyecto.com/api/v1`
- Autenticación: Bearer token (Laravel Sanctum)
- Formato: JSON siempre
- Fechas: ISO 8601 (`2024-01-15T10:30:00Z`)
- Errores: siempre con campo `message` y opcionalmente `errors`

---

## Autenticación

### POST `/auth/login`
Obtener token de acceso.

**Request:**
```json
{
  "email": "usuario@ejemplo.com",
  "password": "contraseña"
}
```

**Response 200:**
```json
{
  "token": "1|abc123...",
  "user": {
    "id": 1,
    "name": "Juan García",
    "email": "usuario@ejemplo.com",
    "role": "admin"
  }
}
```

**Response 422** — Credenciales incorrectas:
```json
{
  "message": "Las credenciales no son correctas.",
  "errors": {
    "email": ["Las credenciales no son correctas."]
  }
}
```

---

### POST `/auth/logout`
Invalidar token actual.

**Headers:** `Authorization: Bearer {token}`

**Response 200:**
```json
{ "message": "Sesión cerrada correctamente." }
```

---

## Productos

### GET `/productos`
Listado paginado de productos.

**Query params:**
- `page` — número de página (default: 1)
- `per_page` — items por página (default: 15, max: 100)
- `categoria` — slug de categoría (opcional)
- `search` — búsqueda por nombre (opcional)
- `order_by` — campo de ordenación: `nombre`, `precio`, `created_at` (default: `created_at`)
- `order_dir` — `asc` o `desc` (default: `desc`)

**Response 200:**
```json
{
  "data": [
    {
      "id": 1,
      "nombre": "Camiseta básica",
      "slug": "camiseta-basica",
      "descripcion": "Descripción corta del producto.",
      "precio": 19.99,
      "precio_oferta": null,
      "stock": 42,
      "activo": true,
      "imagen": "https://cdn.miproyecto.com/productos/camiseta.jpg",
      "categoria": {
        "id": 3,
        "nombre": "Ropa",
        "slug": "ropa"
      },
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "meta": {
    "current_page": 1,
    "last_page": 5,
    "per_page": 15,
    "total": 73
  },
  "links": {
    "first": "https://api.miproyecto.com/api/v1/productos?page=1",
    "last": "https://api.miproyecto.com/api/v1/productos?page=5",
    "prev": null,
    "next": "https://api.miproyecto.com/api/v1/productos?page=2"
  }
}
```

---

### GET `/productos/{slug}`
Detalle de un producto.

**Response 200:**
```json
{
  "data": {
    "id": 1,
    "nombre": "Camiseta básica",
    "slug": "camiseta-basica",
    "descripcion": "Descripción larga en HTML...",
    "precio": 19.99,
    "precio_oferta": 14.99,
    "stock": 42,
    "activo": true,
    "imagenes": [
      "https://cdn.miproyecto.com/productos/camiseta-1.jpg",
      "https://cdn.miproyecto.com/productos/camiseta-2.jpg"
    ],
    "categoria": { "id": 3, "nombre": "Ropa", "slug": "ropa" },
    "atributos": {
      "tallas": ["XS", "S", "M", "L", "XL"],
      "colores": ["blanco", "negro", "azul marino"]
    },
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-03-10T08:15:00Z"
  }
}
```

**Response 404:**
```json
{ "message": "Producto no encontrado." }
```

---

### POST `/productos`
Crear producto. **Requiere:** autenticación + rol `admin`.

**Request:**
```json
{
  "nombre": "Camiseta nueva",
  "descripcion": "Descripción del producto.",
  "precio": 24.99,
  "precio_oferta": null,
  "stock": 100,
  "categoria_id": 3,
  "activo": true
}
```

**Response 201:**
```json
{
  "data": { ...producto creado... },
  "message": "Producto creado correctamente."
}
```

**Response 422** — Validación:
```json
{
  "message": "Los datos proporcionados no son válidos.",
  "errors": {
    "nombre": ["El campo nombre es obligatorio."],
    "precio": ["El precio debe ser mayor que 0."]
  }
}
```

---

### PATCH `/productos/{id}`
Actualizar producto (campos parciales). **Requiere:** autenticación + rol `admin`.

**Request** — solo los campos a modificar:
```json
{
  "precio": 21.99,
  "stock": 85
}
```

**Response 200:**
```json
{
  "data": { ...producto actualizado... },
  "message": "Producto actualizado correctamente."
}
```

---

### DELETE `/productos/{id}`
Eliminar producto (soft delete). **Requiere:** autenticación + rol `admin`.

**Response 200:**
```json
{ "message": "Producto eliminado correctamente." }
```

---

## Pedidos

### POST `/pedidos`
Crear pedido. **Requiere:** autenticación.

**Request:**
```json
{
  "items": [
    { "producto_id": 1, "cantidad": 2, "talla": "M", "color": "negro" },
    { "producto_id": 5, "cantidad": 1 }
  ],
  "direccion_envio": {
    "nombre": "Juan García",
    "calle": "Calle Mayor 15, 3º B",
    "ciudad": "Madrid",
    "codigo_postal": "28001",
    "pais": "ES"
  },
  "codigo_descuento": "SAVE10"
}
```

**Response 201:**
```json
{
  "data": {
    "id": 456,
    "referencia": "PED-2024-000456",
    "estado": "pendiente",
    "subtotal": 64.97,
    "descuento": 6.50,
    "envio": 3.99,
    "total": 62.46,
    "items": [ ...array de items... ],
    "direccion_envio": { ...dirección... },
    "created_at": "2024-03-15T14:22:00Z"
  },
  "message": "Pedido creado. Procede al pago.",
  "pago_url": "https://checkout.stripe.com/..."
}
```

---

## Errores comunes (aplican a todos los endpoints)

| Código | Significado | Cuándo ocurre |
|--------|-------------|----------------|
| 400 | Bad Request | Request malformado |
| 401 | Unauthorized | Token ausente o inválido |
| 403 | Forbidden | Sin permisos para esta acción |
| 404 | Not Found | Recurso no existe |
| 422 | Unprocessable | Errores de validación |
| 429 | Too Many Requests | Rate limit superado |
| 500 | Server Error | Error interno (ver logs) |

**Estructura de error estándar:**
```json
{
  "message": "Descripción del error para el usuario.",
  "errors": {
    "campo": ["Error específico del campo."]
  }
}
```
El campo `errors` solo aparece en errores 422 (validación).

---

## Notas para Claude

- Los IDs son siempre enteros, nunca UUIDs
- Los slugs son únicos y se generan automáticamente del nombre
- `precio_oferta: null` significa sin oferta activa
- La paginación siempre incluye `data`, `meta` y `links`
- Los endpoints de escritura requieren header `Content-Type: application/json`
