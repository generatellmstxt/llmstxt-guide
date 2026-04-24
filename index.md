# Implementing llms.txt: A developer's guide

> Practical guide for engineers shipping llms.txt files for the sites and tools they build, with code examples for Next.js, Astro, WordPress, and static sites.

## What llms.txt is, briefly

A plain-text file at `/llms.txt` describing a site for AI consumption. Modeled after `robots.txt`. Adopted by Anthropic, Mintlify, Cursor, Pydantic, FastAPI, and others. AI crawlers (GPTBot, ClaudeBot, PerplexityBot, Meta-ExternalAgent) actively consume it as of 2026.

Spec: [llmstxt.org](https://llmstxt.org)

## Why this matters for engineers

Three things:

1. **You're the one who has to ship it.** Your marketing team will hear about llms.txt and ping you to "just add the file." Knowing the spec saves a back-and-forth.
2. **It's trivial to generate at build time.** A 50-line script can produce one from your existing sitemap or content collection.
3. **It works particularly well for sites with structured content** — docs, dev tools, APIs. If you ship those, you'll see crawler activity within days of publishing.

## Minimal valid file

\`\`\`text
# Project Name

> One-sentence description of what the project does.

## Section
- [Page Title](/path): Short description of this page
\`\`\`

That's the entire spec. UTF-8, plain text, served at `/llms.txt`.

## Implementations by stack

### Next.js (App Router)

Create `app/llms.txt/route.ts`:

\`\`\`ts
import { NextResponse } from 'next/server';

export async function GET() {
  const content = `# My Project

> Short description.

## Docs
- [Getting Started](/docs/getting-started): First-run guide
- [API Reference](/docs/api): Full API surface
`;

  return new NextResponse(content, {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  });
}
\`\`\`

### Static sites (Astro, Eleventy, Jekyll)

Drop a literal `llms.txt` in the `public/` or `static/` folder. Done.

For dynamic generation in Astro, create `src/pages/llms.txt.ts`:

\`\`\`ts
import { getCollection } from 'astro:content';

export async function GET() {
  const docs = await getCollection('docs');
  const lines = ['# My Site', '', '> Short description.', '', '## Docs'];
  for (const doc of docs) {
    lines.push(`- [${doc.data.title}](/${doc.slug}): ${doc.data.description}`);
  }
  return new Response(lines.join('\n'), {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  });
}
\`\`\`

### WordPress

Two paths:

1. Drop a literal `llms.txt` file at the WordPress root via FTP/SFTP. Survives plugin updates.
2. Use a hook in `functions.php` to serve it dynamically:

\`\`\`php
add_action('init', function() {
  if (parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH) === '/llms.txt') {
    header('Content-Type: text/plain; charset=utf-8');
    echo "# My Site\n\n> Short description.\n\n## Posts\n";
    foreach (get_posts(['numberposts' => 50]) as $post) {
      echo "- [{$post->post_title}](" . get_permalink($post) . "): {$post->post_excerpt}\n";
    }
    exit;
  }
});
\`\`\`

### Express / Node

\`\`\`js
app.get('/llms.txt', (req, res) => {
  res.type('text/plain');
  res.send('# My API\n\n> REST API for X.\n\n## Endpoints\n- ...');
});
\`\`\`

## Generating from a sitemap

If you don't want to hand-author the file, generate it from your sitemap. The HTML title and meta description per page give you all the information you need.

I built a [free llms.txt generator](https://llms-txt-generator.net) that does exactly this. Source: it crawls the sitemap, fetches each page, parses `<title>` and `<meta name="description">` from real HTML, validates URLs with HEAD requests, and outputs the file. Useful when you want to ship something now without writing the script.

## Validation checklist

Before deploying, verify:

- [ ] File served at exactly `/llms.txt` (not `/llms.txt/`, not `/docs/llms.txt`)
- [ ] Content-Type is `text/plain` (not `text/html` — some servers default to HTML for `.txt`)
- [ ] No trailing whitespace on links (some parsers are strict)
- [ ] Every URL returns 200 OK
- [ ] File size under ~100 KB (use `llms-full.txt` for large content payloads)
- [ ] UTF-8 encoded with no BOM

## Monitoring crawler activity

Once shipped, grep your access logs for AI crawlers:

\`\`\`bash
grep -E "GPTBot|ClaudeBot|PerplexityBot|Meta-ExternalAgent|Amazonbot|CCBot" access.log | head
\`\`\`

You should see hits within 1–4 weeks.

## Related

- Spec: [llmstxt.org](https://llmstxt.org)
- Free generator (built by me): [llms-txt-generator.net](https://llms-txt-generator.net)
- Examples to study: `anthropic.com/llms.txt`, `mintlify.com/llms.txt`, `pydantic.dev/llms.txt`

---

Found this useful? Try the [llms.txt generator](https://llms-txt-generator.net) — free, supports 7 languages, no signup.
