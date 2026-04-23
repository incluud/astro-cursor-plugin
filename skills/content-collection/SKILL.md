---
name: content-collection
description: Set up and configure Astro content collections with the current Content Layer API and type-safe schemas. Use when creating blogs, docs, or any structured content.
---

# Content Collection Setup

## When to Use

- Setting up a new blog, documentation, or portfolio
- Creating structured content with frontmatter validation
- Migrating from `Astro.glob()` or legacy content collections
- Adding new collection types to an existing project

## Instructions

### 1. Understand the Structure

In Astro 6+, build-time content collections are configured in `src/content.config.ts`, while the content entries themselves can live in `src/content/` or another folder you point to with a loader.

Typical structure:

```text
src/
├── content.config.ts      # Collection definitions
├── content/
│   └── blog/
│       ├── post-1.md
│       ├── post-2.mdx
│       └── drafts/
│           └── draft.md
└── data/
    └── authors.json       # Single-file data collection example
```

If you need request-time fresh data, use live collections in `src/live.config.ts`. For most blogs, docs, and portfolios, build-time collections are the right default.

### 2. Define the Schema

Create `src/content.config.ts` (or `.js` / `.mjs`):

```typescript
import { defineCollection, reference } from 'astro:content'
import { glob, file } from 'astro/loaders'
import { z } from 'astro/zod'

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.{md,mdx}' }),
  schema: ({ image }) => z.object({
    title: z.string(),
    description: z.string(),
    publishDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    author: reference('authors'),
    cover: image().optional(),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false),
  }),
})

const authors = defineCollection({
  loader: file('src/data/authors.json'),
  schema: z.object({
    id: z.string(),
    name: z.string(),
    email: z.string().email().optional(),
    avatar: z.string().url().optional(),
  }),
})

export const collections = { blog, authors }
```

Notes:

- Every collection needs a `loader`
- Import `z` from `astro/zod`, not from `astro:content`
- Do not use `type: 'content'` or `type: 'data'` in Astro 6+
- When using `file()`, each entry needs a unique `id`

### 3. Common Schema Patterns

#### Images with validation

```typescript
import { defineCollection } from 'astro:content'
import { glob } from 'astro/loaders'
import { z } from 'astro/zod'

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.{md,mdx}' }),
  schema: ({ image }) => z.object({
    title: z.string(),
    cover: image(), // Validates the file exists
    coverAlt: z.string(),
  }),
})
```

#### Reference other collections

```typescript
import { defineCollection, reference } from 'astro:content'
import { glob } from 'astro/loaders'
import { z } from 'astro/zod'

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.{md,mdx}' }),
  schema: z.object({
    title: z.string(),
    author: reference('authors'), // References an authors entry by id
    relatedPosts: z.array(reference('blog')).optional(),
  }),
})
```

#### Enum for categories

```typescript
schema: z.object({
  category: z.enum(['tutorial', 'guide', 'reference', 'blog']),
  status: z.enum(['draft', 'review', 'published']).default('draft'),
})
```

If you store data entries as separate JSON files instead of one JSON file, use `glob({ base: './src/data/authors', pattern: '**/*.json' })` instead of `file()`.

### 4. Query Collections

```astro
---
import { getCollection, getEntry, render } from 'astro:content'

// Get all entries
const allPosts = await getCollection('blog')

// Filter entries
const publishedPosts = await getCollection('blog', ({ data }) => {
  return data.draft !== true
})

// Get single entry by id
const post = await getEntry('blog', 'my-post')

// Render content
const { Content, headings } = await render(post)
---

<Content />
```

Important changes in Astro 6+:

- Use `entry.id` as the slug-like identifier
- Use `render(entry)` instead of `entry.render()`
- Use `getEntry()` instead of older slug-specific helpers

### 5. Dynamic Routes

Create `src/pages/blog/[...slug].astro`:

```astro
---
import { getCollection, render } from 'astro:content'
import BlogLayout from '../../layouts/BlogLayout.astro'

export async function getStaticPaths() {
  const posts = await getCollection('blog')
  return posts.map(post => ({
    params: { slug: post.id },
    props: { post },
  }))
}

const { post } = Astro.props
const { Content } = await render(post)
---

<BlogLayout title={post.data.title}>
  <article>
    <h1>{post.data.title}</h1>
    <time datetime={post.data.publishDate.toISOString()}>
      {post.data.publishDate.toLocaleDateString()}
    </time>
    <Content />
  </article>
</BlogLayout>
```

If you need the original file location for debugging or special routing logic, use `post.filePath`. For URLs and lookups, use `post.id`.

### 6. Generate Types

Run `astro sync` or `astro dev` to generate TypeScript types:

```bash
npx astro sync
```

This creates `.astro/` with type definitions for your collections.

## Migration from Astro.glob() and Legacy Content Collections

If migrating from the old `Astro.glob()` or pre-Content Layer pattern:

1. Move `src/content/config.ts` to `src/content.config.ts`
2. Add a `loader` to every collection
3. Remove `type: 'content'` and `type: 'data'`
4. Replace `import { defineCollection, z } from 'astro:content'` with `import { defineCollection } from 'astro:content'` and `import { z } from 'astro/zod'`
5. Replace `post.slug` with `post.id`
6. Replace `post.render()` with `render(post)`
7. Replace `Astro.glob()` with `getCollection()` or `import.meta.glob()`

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `LegacyContentConfigError` | Move `src/content/config.ts` to `src/content.config.ts` |
| `ContentCollectionMissingALoaderError` | Add a `loader` to the collection definition |
| `ContentCollectionInvalidTypeError` | Remove `type: 'content'` or `type: 'data'` |
| Content entry not found by slug | Use `entry.id` instead of `entry.slug` |
| Render method missing | Import `render` from `astro:content` and call `render(entry)` |
| Images not validating | Use `schema: ({ image }) =>` function syntax |
