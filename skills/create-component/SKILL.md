---
name: create-component
description: Scaffold Astro components, pages, and layouts with best practices. Use when creating new .astro files or converting designs to components.
---

# Create Astro Component

## When to Use

- Creating new `.astro` components, pages, or layouts
- Scaffolding from a design or description
- Converting static HTML to Astro components
- Setting up new content collection entries

## Instructions

### 1. Determine Component Type

Ask or infer what type of file is needed:

| Type | Location | Purpose |
|------|----------|---------|
| **Component** | `src/components/` | Reusable UI element |
| **Page** | `src/pages/` | Route endpoint |
| **Layout** | `src/layouts/` | Page wrapper with common structure |
| **Content** | `src/content/` | Markdown/MDX content entry |

### 2. Gather Requirements

Before scaffolding, clarify:
- What props does it need?
- Is client-side JavaScript required?
- Does it need to fetch data?
- What styling approach (scoped, Tailwind, global)?

### 3. Scaffold with Best Practices

#### Component Template

```astro
---
interface Props {
  title: string
  variant?: 'primary' | 'secondary'
}

const { title, variant = 'primary' } = Astro.props
---

<div class:list={['component', variant]}>
  <h2>{title}</h2>
  <slot />
</div>

<style>
  .component {
    /* Scoped styles */
  }
</style>
```

#### Page Template

```astro
---
import Layout from '../layouts/Layout.astro'

const pageTitle = 'Page Title'
---

<Layout title={pageTitle}>
  <main>
    <h1>{pageTitle}</h1>
    <slot />
  </main>
</Layout>
```

#### Layout Template

```astro
---
interface Props {
  title: string
  description?: string
}

const { title, description } = Astro.props
---

<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="description" content={description} />
    <title>{title}</title>
  </head>
  <body>
    <slot />
  </body>
</html>
```

### 4. Apply Conventions

- **Naming**: PascalCase for components (`Button.astro`, `NavBar.astro`)
- **Props**: Use TypeScript interface for type safety
- **Defaults**: Provide sensible defaults for optional props
- **Slots**: Use named slots for complex layouts
- **Styles**: Prefer scoped styles, use `class:list` for conditional classes

### 5. Accessibility Checklist

Before finalizing, verify:
- [ ] Semantic HTML elements used appropriately
- [ ] Interactive elements are keyboard accessible
- [ ] Images have alt text (or `alt=""` if decorative)
- [ ] Heading hierarchy is logical
- [ ] Color contrast meets WCAG AA (4.5:1 for text)
- [ ] Focus indicators are visible

## Dynamic Routes

For pages with dynamic routes:

```astro
---
// src/pages/blog/[slug].astro
import { getCollection } from 'astro:content'

export async function getStaticPaths() {
  const posts = await getCollection('blog')
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post }
  }))
}

const { post } = Astro.props
const { Content } = await post.render()
---

<Layout title={post.data.title}>
  <article>
    <h1>{post.data.title}</h1>
    <Content />
  </article>
</Layout>
```

## Interactive Components

When client-side JS is needed:

```astro
---
import Counter from '../components/Counter.tsx'
---

<!-- Only hydrate when visible in viewport -->
<Counter client:visible initialCount={0} />
```

Choose the appropriate client directive based on priority and use case.
