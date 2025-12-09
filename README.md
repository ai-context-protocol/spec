# AI-Context Protocol

**A web standard for AI-optimized content discovery and consumption.**

## What is this?

The AI-Context Protocol defines a standard way for websites to provide AI-optimized representations of their content alongside human-readable HTML. This enables AI agents to efficiently discover and consume web content without parsing full HTML pages.

Think of it like this: websites already provide `robots.txt` for crawlers, `sitemap.xml` for search engines, and RSS for feed readers. The AI-Context Protocol adds `.aic.json` files for AI agents.

## The Problem

AI agents browsing the web must:
1. Download full HTML pages (100KB+ with CSS, JS, navigation, ads)
2. Parse and extract the actual content
3. Process through their understanding pipeline

This is inefficient for everyone - the AI, the website, and the user waiting for results.

## The Solution

A simple convention:
```
article.html       -> for humans to read
article.aic.json   -> for AI agents to consume
```

AI agents check `/.well-known/ai-context.json` first to discover available AI-context files, then fetch only what they need.

## Quick Example

**Manifest** (`/.well-known/ai-context.json`):
```json
{
  "version": "0.1",
  "site": "https://example.com",
  "pages": [
    {
      "url": "/blog/hello-world",
      "aic": "/ai-context/hello-world.aic.json",
      "lastmod": "2025-12-08"
    }
  ]
}
```

**AIC File** (`/ai-context/hello-world.aic.json`):
```json
{
  "version": "0.1",
  "source_url": "https://example.com/blog/hello-world",
  "content": {
    "title": "Hello World",
    "summary": "An introductory post about...",
    "content_type": "article",
    "entities": ["Topic A", "Topic B"],
    "key_facts": ["Fact 1", "Fact 2"]
  }
}
```

## Specification

The full specification is available at:
- [AI-Context Protocol Specification v0.1](spec/ai-context-protocol-v0.1.md)

## Key Features

- **Simple discovery**: `/.well-known/ai-context.json` manifest
- **Flexible format**: Semantic JSON with optional vector embeddings
- **Scalable**: Small sites use one file; large sites can offer multiple embedding formats
- **Secure by design**: Untrusted by default, source verification built in
- **Open standard**: Apache 2.0 licensed, no vendor lock-in

## Project Status

**Current Phase**: Specification Draft (v0.1)

- [x] Core specification drafted
- [ ] Reference implementation (Python)
- [ ] Demonstration website
- [ ] IANA well-known URI registration

## Author

Created by Mark Mundy ([@mmundy3832](https://github.com/mmundy3832)), December 2025.

## Timestamps

Public record established December 9, 2025:
- [Specification (Archive.org)](https://web.archive.org/web/20251209184500/https://github.com/ai-context-protocol/spec/blob/main/spec/ai-context-protocol-v0.1.md)
- [README (Archive.org)](https://web.archive.org/web/20251209184812/https://github.com/ai-context-protocol/spec/blob/main/README.md)

## License

This specification is licensed under the [Apache License 2.0](LICENSE).

## Contributing

This project is in early development. Feedback and contributions welcome via issues and pull requests.
