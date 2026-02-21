---
name: add-integration
description: Guide complete integration setup including post-install configuration. Use when adding UI frameworks, adapters, or tooling to an Astro project.
---

# Add Integration

## When to Use

- Adding a UI framework (React, Vue, Svelte, etc.)
- Setting up Tailwind CSS or other styling tools
- Configuring deployment adapters (Netlify, Vercel, Cloudflare)
- Installing community integrations
- Troubleshooting integration issues

## Instructions

### 1. Identify Integration Type

| Type | Examples | Key Considerations |
|------|----------|-------------------|
| **UI Framework** | React, Vue, Svelte, Solid, Preact | Client directives, hydration strategy |
| **Styling** | Tailwind, UnoCSS | CSS setup, config files |
| **Adapter** | Netlify, Vercel, Node, Cloudflare | Output mode, environment variables |
| **Content** | MDX, Markdoc | Content collections integration |
| **Other** | Sitemap, Partytown, DB | Feature-specific config |

### 2. Installation

**Official integrations** (support `astro add`):

```bash
npx astro add react
npx astro add tailwind
npx astro add netlify
```

**Community integrations** (manual install):

```bash
npm install astro-integration-name
```

Then add to `astro.config.mjs`:

```javascript
import { defineConfig } from 'astro/config'
import integration from 'astro-integration-name'

export default defineConfig({
  integrations: [integration()],
})
```

### 3. Post-Install Configuration

#### Tailwind CSS v4

After `npx astro add tailwind`:

1. Create CSS file `src/styles/global.css`:
   ```css
   @import "tailwindcss";
   ```

2. Import in your layout:
   ```astro
   ---
   import '../styles/global.css'
   ---
   ```

3. Optional: Configure in `tailwind.config.mjs` for customization

#### React / Vue / Svelte

After adding a framework, understand client directives:

```astro
---
import Counter from './Counter.tsx'
---

<!-- No directive = server-rendered only, no JS -->
<Counter />

<!-- client:load = hydrate immediately (critical interactivity) -->
<Counter client:load />

<!-- client:idle = hydrate when browser is idle -->
<Counter client:idle />

<!-- client:visible = hydrate when scrolled into view -->
<Counter client:visible />

<!-- client:media = hydrate on media query match -->
<Counter client:media="(max-width: 768px)" />

<!-- client:only = skip server render entirely -->
<Counter client:only="react" />
```

**Best practice**: Start with `client:visible` or `client:idle`, use `client:load` only for above-the-fold critical UI.

#### Deployment Adapters

**Netlify** (`npx astro add netlify`):

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config'
import netlify from '@astrojs/netlify'

export default defineConfig({
  output: 'server', // or 'hybrid'
  adapter: netlify(),
})
```

Set environment variables in Netlify dashboard or `netlify.toml`.

**Vercel** (`npx astro add vercel`):

```javascript
import { defineConfig } from 'astro/config'
import vercel from '@astrojs/vercel'

export default defineConfig({
  output: 'server',
  adapter: vercel({
    webAnalytics: { enabled: true }, // Optional
  }),
})
```

**Cloudflare** (`npx astro add cloudflare`):

```javascript
import { defineConfig } from 'astro/config'
import cloudflare from '@astrojs/cloudflare'

export default defineConfig({
  output: 'server',
  adapter: cloudflare({
    platformProxy: { enabled: true }, // For local dev
  }),
})
```

#### MDX

After `npx astro add mdx`:

- `.mdx` files work in `src/pages/` and `src/content/`
- Import components directly in MDX files
- Configure remark/rehype plugins in `astro.config.mjs`:

```javascript
import { defineConfig } from 'astro/config'
import mdx from '@astrojs/mdx'

export default defineConfig({
  integrations: [mdx()],
  markdown: {
    remarkPlugins: [remarkPlugin],
    rehypePlugins: [rehypePlugin],
  },
})
```

### 4. Multiple Integrations

Order can matter. General guidelines:

```javascript
export default defineConfig({
  integrations: [
    // UI frameworks first
    react(),
    vue(),
    // Then styling
    tailwind(),
    // Then content
    mdx(),
    // Then utilities
    sitemap(),
  ],
})
```

### 5. Troubleshooting

| Issue | Solution |
|-------|----------|
| "Cannot find module" | Run `npm install` again, check package.json |
| Styles not loading | Check CSS import path, restart dev server |
| Hydration mismatch | Ensure server/client render same content |
| Build fails | Check adapter matches deployment platform |
| Integration not working | Verify it's in `integrations[]` array |

## Finding Integrations

- **Official**: [docs.astro.build/integrations](https://docs.astro.build/en/guides/integrations-guide/)
- **Community**: [astro.build/integrations](https://astro.build/integrations/)
