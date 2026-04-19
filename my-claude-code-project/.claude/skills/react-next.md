# Skill: React y Next.js Development

> Claude carga este skill cuando trabaja con React o Next.js.
> Stack: TypeScript estricto. Sin any explícito. App Router en Next.js.

## IMPORTANTE — Contexto de uso
React y Next.js son tecnologías secundarias en el proyecto. Si la misma
funcionalidad puede resolverse con Astro (contenido estático, landing pages,
blogs), USAR ASTRO. React/Next.js cuando se necesite:
- Estado del cliente complejo
- Autenticación y rutas protegidas
- Real-time / WebSockets
- App tipo SPA con mucha interactividad

---

## React — Patrones fundamentales

### Componentes funcionales con TypeScript
```tsx
// ✅ Correcto: tipos explícitos, sin any
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
  isLoading?: boolean;
}

export function Button({
  label,
  onClick,
  variant = 'primary',
  disabled = false,
  isLoading = false,
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || isLoading}
      className={`btn btn--${variant}`}
      aria-busy={isLoading}
    >
      {isLoading ? 'Cargando...' : label}
    </button>
  );
}
```

### Custom Hooks — separar lógica de presentación
```tsx
// hooks/useProductos.ts
import { useState, useEffect } from 'react';

interface Producto {
  id: number;
  nombre: string;
  precio: number;
}

interface UseProductosReturn {
  productos: Producto[];
  isLoading: boolean;
  error: string | null;
  refetch: () => void;
}

export function useProductos(): UseProductosReturn {
  const [productos, setProductos] = useState<Producto[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchProductos = async () => {
    try {
      setIsLoading(true);
      setError(null);
      const res = await fetch('/api/productos');
      if (!res.ok) throw new Error('Error al cargar productos');
      const data = await res.json();
      setProductos(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Error desconocido');
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => { fetchProductos(); }, []);

  return { productos, isLoading, error, refetch: fetchProductos };
}

// Componente — solo presentación
export function ListaProductos() {
  const { productos, isLoading, error, refetch } = useProductos();

  if (isLoading) return <div aria-live="polite">Cargando...</div>;
  if (error) return <div role="alert">{error} <button onClick={refetch}>Reintentar</button></div>;

  return (
    <ul>
      {productos.map(p => (
        <li key={p.id}>{p.nombre} — {p.precio}€</li>
      ))}
    </ul>
  );
}
```

### Manejo de formularios
```tsx
// Con React Hook Form (preferido para formularios complejos)
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  email: z.string().email('Email inválido'),
  nombre: z.string().min(2, 'Mínimo 2 caracteres'),
});

type FormData = z.infer<typeof schema>;

export function FormularioContacto() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: FormData) => {
    await fetch('/api/contacto', {
      method: 'POST',
      body: JSON.stringify(data),
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} aria-describedby="email-error" />
      {errors.email && <span id="email-error" role="alert">{errors.email.message}</span>}
      <button type="submit" disabled={isSubmitting}>Enviar</button>
    </form>
  );
}
```

---

## Next.js App Router — Patrones

### Server vs Client Components

```tsx
// Server Component (por defecto) — sin 'use client'
// Se ejecuta en servidor, puede hacer fetch directo, acceder a BD
async function PaginaProductos() {
  // Fetch directo al servidor, sin pasar por API externa
  const productos = await db.query('SELECT * FROM productos');
  
  return (
    <main>
      <h1>Productos</h1>
      {/* Pasar datos a componentes cliente solo lo necesario */}
      <FiltroProductos /> {/* Client Component para interactividad */}
      <ListaProductos productos={productos} /> {/* Server Component */}
    </main>
  );
}

// Client Component — solo cuando hay interactividad
'use client';
import { useState } from 'react';

function FiltroProductos() {
  const [filtro, setFiltro] = useState('');
  return <input value={filtro} onChange={e => setFiltro(e.target.value)} />;
}
```

### Estructura de app/ en Next.js
```
app/
├── (auth)/              # Route group — sin segmento en URL
│   ├── login/
│   │   └── page.tsx
│   └── layout.tsx       # Layout para rutas auth
├── (dashboard)/
│   ├── dashboard/
│   │   └── page.tsx
│   └── layout.tsx
├── api/
│   └── [ruta]/
│       └── route.ts     # API Route handlers
├── layout.tsx           # Root layout
├── page.tsx             # Home
├── loading.tsx          # Loading UI automático
├── error.tsx            # Error boundary automático
└── not-found.tsx
```

### Layouts con metadata
```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    template: '%s | Mi Sitio',
    default: 'Mi Sitio',
  },
  description: 'Descripción del sitio',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="es">
      <body>{children}</body>
    </html>
  );
}
```

### API Routes (Route Handlers)
```typescript
// app/api/productos/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const categoria = searchParams.get('categoria');
  
  // fetch data...
  
  return NextResponse.json({ productos: [] });
}

export async function POST(request: NextRequest) {
  const body = await request.json();
  
  // Validar con zod
  const result = schema.safeParse(body);
  if (!result.success) {
    return NextResponse.json({ errors: result.error.issues }, { status: 400 });
  }
  
  return NextResponse.json({ ok: true }, { status: 201 });
}
```

### Fetch y caché en Next.js
```tsx
// Revalidar cada hora
const data = await fetch('/api/data', { next: { revalidate: 3600 } });

// Siempre fresco (sin caché)
const data = await fetch('/api/data', { cache: 'no-store' });

// Caché estático (build time, como getStaticProps)
const data = await fetch('/api/data', { cache: 'force-cache' });
```

### Middleware (autenticación, redirects)
```typescript
// middleware.ts — en la raíz del proyecto
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth_token');
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

---

## TypeScript — Reglas estrictas

```typescript
// tsconfig.json — configuración mínima exigida
{
  "compilerOptions": {
    "strict": true,              // Activa: noImplicitAny, strictNullChecks, etc.
    "noUncheckedIndexedAccess": true,  // array[i] puede ser undefined
    "exactOptionalPropertyTypes": true
  }
}
```

```typescript
// ❌ Nunca
const datos: any = obtenerDatos();
function procesar(valor: any): any { ... }

// ✅ Siempre tipar
interface Datos { id: number; nombre: string }
const datos: Datos = obtenerDatos();
function procesar(valor: string): number { ... }

// ✅ Para tipos desconocidos usar unknown + type guard
function procesar(valor: unknown): string {
  if (typeof valor === 'string') return valor.toUpperCase();
  throw new Error('Tipo no soportado');
}
```

---

## Comandos
```bash
# Next.js
npm run dev          # http://localhost:3000
npm run build        # build producción
npm run start        # servidor producción
npm run lint         # ESLint
npm run type-check   # tsc --noEmit

# Librerías comunes del stack
npm install zod                           # validación de schemas
npm install react-hook-form @hookform/resolvers  # formularios
npm install @tanstack/react-query         # server state management
```
