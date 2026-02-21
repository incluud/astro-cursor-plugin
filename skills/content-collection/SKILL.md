---
name: content-collection
description: Set up and configure Astro content collections with type-safe schemas. Use when creating blogs, docs, or any structured content.
---

# Content Collection Setup

## When to Use

- Setting up a new blog, documentation, or portfolio
- Creating structured content with frontmatter validation
- Migrating from `Astro.glob()` to content collections
- Adding new collection types to an existing project

## Instructions

### 1. Understand the Structure

Content collections live in `src/content/` with this structure:

```
src/content/
├── config.ts          # Schema definitions
├── blog/              # Collection folder
│   ├── post-1.md
│   ├── post-2.mdx
│   └── drafts/        # Subdirectories supported
│       └── draft.md
└── authors/           # Another collection
    └── jane.json      # JSON/YAML also supported
```

### 2. Define the Schema

Create `src/content/config.ts` (or `.js`):

```typescript
import { defineCollection, z } from 'astro:content'

const blog = defineCollection({
  type: 'content', // Markdown/MDX
  schema: z.object({
    title: z.string(),
    description: z.string(),
    publishDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    author: z.string().default('Anonymous'),
    image: z.string().optional(),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false),
  }),
})

const authors = defineCollection({
  type: 'data', // JSON/YAML
  schema: z.object({
    name: z.string(),
    email: z.string().email(),
    avatar: z.string().url().optional(),
  }),
})

export const collections = { blog, authors }
```

### 3. Common Schema Patterns

#### Images with validation

```typescript
import { defineCollection, z } from 'astro:content'

const blog = defineCollection({
  type: 'content',
  schema: ({ image }) => z.object({
    title: z.string(),
    cover: image(), // Validates image exists
    coverAlt: z.string(),
  }),
})
```

#### Reference other collections

```typescript
import { defineCollection, z, reference } from 'astro:content'

const blog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    author: reference('authors'), // Links to authors collection
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

### 4. Query Collections

```astro
---
import { getCollection, getEntry } from 'astro:content'

// Get all entries
const allPosts = await getCollection('blog')

// Filter entries
const publishedPosts = await getCollection('blog', ({ data }) => {
  return data.draft !== true
})

// Get single entry
const post = await getEntry('blog', 'my-post-slug')

// Render content
const { Content, headings } = await post.render()
---

<Content />
```

### 5. Dynamic Routes

Create `src/pages/blog/[...slug].astro`:

```astro
---
import { getCollection } from 'astro:content'
import BlogLayout from '../../layouts/BlogLayout.astro'

export async function getStaticPaths() {
  const posts = await getCollection('blog')
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post },
  }))
}

const { post } = Astro.props
const { Content } = await post.render()
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

### 6. Generate Types

Run `astro sync` or `astro dev` to generate TypeScript types:

```bash
npx astro sync
```

This creates `.astro/` with type definitions for your collections.

## Migration from Astro.glob()

If migrating from the old `Astro.glob()` pattern:

1. Move files from `src/pages/blog/*.md` to `src/content/blog/`
2. Create `src/content/config.ts` with schema
3. Replace `Astro.glob()` calls with `getCollection()`
4. Update dynamic routes to use `getStaticPaths()` pattern above

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Schema errors | Run `astro sync` to regenerate types |
| Collection not found | Check folder is directly under `src/content/` |
| Type errors | Ensure `config.ts` exports `collections` object |
| Images not validating | Use `schema: ({ image }) =>` function syntax |
