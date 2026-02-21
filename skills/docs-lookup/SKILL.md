---
name: docs-lookup
description: Search and retrieve Astro documentation using the Astro Docs MCP. Use when answering questions about Astro APIs, configuration, integrations, or best practices.
---

# Astro Documentation Lookup

## When to Use

- User asks about Astro features, APIs, or configuration
- Need to verify correct syntax or usage patterns
- Looking up integration setup (Tailwind, MDX, adapters, etc.)
- Checking deployment or adapter documentation
- Answering "how do I..." questions about Astro

## Instructions

1. **Identify the topic** from the user's question (e.g., "content collections", "view transitions", "SSR adapters")

2. **Search the documentation** using the Astro Docs MCP:
   ```
   Use the search_astro_docs tool with relevant keywords
   ```

3. **Review results** and select the most relevant documentation sections

4. **Synthesize the answer** by:
   - Providing a concise explanation
   - Including relevant code examples from the docs
   - Linking to the full documentation when helpful

5. **Verify accuracy** - If the MCP returns no results, try:
   - Alternative search terms
   - Breaking down complex queries
   - Searching for related concepts

## Search Tips

- Use specific terms: "content collections schema" vs just "collections"
- Include version-specific terms if relevant: "Astro 5 content layer"
- Search for error messages directly when debugging
- Look for integration names: "astro tailwind integration"

## Example Queries

| User Question | Search Terms |
|--------------|--------------|
| "How do I add Tailwind?" | `tailwind integration setup` |
| "What are content collections?" | `content collections` |
| "How do I deploy to Vercel?" | `vercel adapter deployment` |
| "View transitions not working" | `view transitions troubleshooting` |

## Response Format

When answering documentation questions:

1. **Brief answer** - One or two sentences summarizing the solution
2. **Code example** - Working snippet demonstrating the concept
3. **Key details** - Important caveats or options to consider
4. **Further reading** - Link to relevant docs section if complex topic
