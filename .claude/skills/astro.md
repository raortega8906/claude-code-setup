# Skill: Astro Development

> Claude carga este skill para proyectos Astro.
> Astro es el framework principal para sites de contenido y landing pages del proyecto.

## Versión y configuración
- Astro: ver `package.json`
- TypeScript: estricto (ver `tsconfig.json`)
- Integraciones: ver `astro.config.mjs`

## Principio fundamental de Astro

**Zero JS por defecto.** Astro renderiza HTML estático. El JavaScript del cliente
es opt-in y solo debe añadirse cuando hay interactividad real.

```
Estático → .astro puro
Interactivo → .astro + componente React/Vue con directiva client:*
```

## Directivas de hidratación — cuándo usar cada una

```astro
<!-- Componente siempre necesita JS (menú desplegable, modal) -->
<MiComponente client:load />

<!-- Componente visible más abajo — hidrata al llegar al viewport -->
<MiComponente client:visible />

<!-- Hidrata en el primer evento idle del navegador (baja prioridad) -->
<MiComponente client:idle />

<!-- Solo en cliente, nunca SSR (componentes con browser APIs) -->
<MiComponente client:only="react" />

<!-- NO usar client:load para componentes puramente visuales sin estado -->
```

## Estructura del proyecto Astro

```
src/
├── components/
│   ├── ui/              # Componentes reutilizables sin lógica de negocio
│   ├── sections/        # Secciones de páginas
│   └── [Feature]/       # Componentes con lógica de negocio
├── layouts/
│   ├── BaseLayout.astro
│   └── BlogLayout.astro
├── pages/
│   ├── index.astro
│   ├── blog/
│   │   ├── index.astro
│   │   └── [slug].astro
│   └── api/             # Endpoints de API (si se usa SSR)
├── content/             # Content Collections
│   └── blog/
│       └── primer-post.md
├── styles/
│   └── global.css
└── utils/               # Helpers y funciones puras
```

## Componentes Astro — patrones correctos

### Componente básico
```astro
---
// Script del servidor — se ejecuta en build time
import type { Props } from './types';
import OtroComponente from './OtroComponente.astro';

interface Props {
  titulo: string;
  descripcion?: string;
  clase?: string;
}

const { titulo, descripcion = '', clase = '' } = Astro.props;
---

<section class={`mi-seccion ${clase}`}>
  <h2>{titulo}</h2>
  {descripcion && <p>{descripcion}</p>}
  <slot />
</section>

<style>
  .mi-seccion {
    /* Scoped por defecto — no afecta a hijos */
  }
</style>
```

### Imágenes — SIEMPRE usar el componente Image
```astro
---
import { Image } from 'astro:assets';
import miImagen from '../assets/imagen.jpg';
---

<!-- ✅ Correcto: optimización automática, lazy load, dimensiones -->
<Image src={miImagen} alt="Descripción descriptiva de la imagen" />

<!-- Para imágenes remotas o dinámicas -->
<Image src="https://ejemplo.com/imagen.jpg" alt="Descripción" width={800} height={600} />

<!-- ❌ No usar img directa para assets locales -->
<img src="/imagen.jpg" alt="..." />
```

### Content Collections
```astro
---
// src/content/config.ts
import { z, defineCollection } from 'astro:content';

const blog = defineCollection({
  type: 'content',
  schema: z.object({
    titulo: z.string(),
    fecha: z.date(),
    descripcion: z.string(),
    imagen: z.string().optional(),
    draft: z.boolean().default(false),
  }),
});

export const collections = { blog };
---
```

```astro
---
// En página de listado
import { getCollection } from 'astro:content';

const posts = await getCollection('blog', ({ data }) => !data.draft);
const postOrdenados = posts.sort((a, b) => 
  b.data.fecha.valueOf() - a.data.fecha.valueOf()
);
---
```

### Layouts con SEO
```astro
---
// src/layouts/BaseLayout.astro
interface Props {
  titulo: string;
  descripcion: string;
  imagen?: string;
  noindex?: boolean;
}

const { titulo, descripcion, imagen, noindex = false } = Astro.props;
const sitio = Astro.site?.toString() ?? '';
const canonicalURL = new URL(Astro.url.pathname, Astro.site);
---

<!doctype html>
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{titulo}</title>
    <meta name="description" content={descripcion} />
    <link rel="canonical" href={canonicalURL} />
    {noindex && <meta name="robots" content="noindex" />}
    
    <!-- Open Graph -->
    <meta property="og:title" content={titulo} />
    <meta property="og:description" content={descripcion} />
    {imagen && <meta property="og:image" content={new URL(imagen, sitio)} />}
  </head>
  <body>
    <slot />
  </body>
</html>
```

## Rutas dinámicas y SSR

```astro
---
// pages/blog/[slug].astro — Generación estática
import { getCollection, getEntry } from 'astro:content';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await post.render();
---

<BaseLayout titulo={post.data.titulo} descripcion={post.data.descripcion}>
  <Content />
</BaseLayout>
```

## Endpoints de API (modo SSR)
```typescript
// src/pages/api/contacto.ts
import type { APIRoute } from 'astro';

export const POST: APIRoute = async ({ request }) => {
  const data = await request.json();
  
  // Validar y procesar
  if (!data.email || !data.mensaje) {
    return new Response(JSON.stringify({ error: 'Datos incompletos' }), {
      status: 400,
      headers: { 'Content-Type': 'application/json' },
    });
  }
  
  return new Response(JSON.stringify({ ok: true }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' },
  });
};
```

## Rendimiento — checklist
- [ ] Imágenes locales con `<Image>` (formato WebP automático)
- [ ] Fuentes con `font-display: swap`
- [ ] CSS crítico inline para above-the-fold
- [ ] Componentes React solo con `client:visible` o `client:idle` si no son críticos
- [ ] Prefetch en links importantes: `<a href="..." data-astro-prefetch>`
- [ ] Variables de entorno en `import.meta.env` (no `process.env`)

## Comandos
```bash
npm run dev              # http://localhost:4321
npm run build            # build en /dist
npm run preview          # previsualizar el build
npm run check            # TypeScript check
astro add react          # añadir integración React
astro add tailwind       # añadir Tailwind
```
