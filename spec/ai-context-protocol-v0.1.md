# AI-Context Protocol Specification

**Version:** 0.1 (Draft)
**Status:** Working Draft
**Date:** December 9, 2025
**Author:** Mark (with Claude assistance)

---

## Abstract

The AI-Context Protocol defines a standard for websites to provide AI-optimized representations of their content. This enables AI agents to efficiently discover and consume web content in a format closer to their native representational space, reducing bandwidth, processing overhead, and improving semantic understanding.

---

## 1. Introduction

### 1.1 Purpose

As AI agents increasingly browse and consume web content, a gap exists in current web standards: there is no convention for providing AI-optimized content alongside human-readable HTML. The AI-Context Protocol fills this gap by defining:

1. A discovery mechanism for AI agents to locate AI-optimized content
2. A file format for semantic content representation
3. Optional embedding formats for vector-based semantic search

### 1.2 Design Goals

- **Simplicity:** Easy for website owners to implement
- **Efficiency:** Reduce bandwidth and processing for AI agents
- **Flexibility:** Support minimal implementations and comprehensive deployments
- **Security:** Treat AI-context files as untrusted input by default
- **Openness:** No proprietary dependencies or vendor lock-in

### 1.3 Terminology

- **AI Agent:** Any automated system that consumes web content (crawlers, assistants, autonomous agents)
- **AIC File:** A JSON file containing AI-optimized content representation (`.aic.json`)
- **Embedding:** A numerical vector representation of content meaning
- **Manifest:** The `/.well-known/ai-context.json` file declaring available AI-context resources

---

## 2. Discovery Mechanism

### 2.1 Primary: Well-Known Manifest

AI agents SHOULD first request the well-known manifest:

```
GET /.well-known/ai-context.json
```

If present, this file declares all available AI-context resources for the site.

### 2.2 Fallback: Naming Convention

If no manifest exists (404 response), agents MAY attempt to locate AI-context files using the naming convention:

```
Original URL:  https://example.com/article/my-post
AIC File:      https://example.com/article/my-post.aic.json
```

### 2.3 Secondary Discovery (Optional)

For compatibility and debugging, sites MAY also declare AI-context availability via:

**HTML Link Tag:**
```html
<link rel="ai-context" type="application/json" href="/ai-context/my-post.aic.json" />
```

**HTTP Header:**
```
Link: </ai-context/my-post.aic.json>; rel="ai-context"; type="application/json"
```

These methods require fetching the original resource first and are therefore less efficient. They are intended for human developers, search engine indexing, and fallback scenarios.

---

## 3. Manifest Format

### 3.1 Location

The manifest MUST be located at:

```
/.well-known/ai-context.json
```

### 3.2 Schema

```json
{
  "version": "0.1",
  "site": "https://example.com",
  "generated_at": "2025-12-09T10:00:00Z",
  "generator": {
    "name": "generator-name",
    "version": "1.0.0"
  },
  "pages": [
    {
      "url": "/path/to/page",
      "aic": "/ai-context/page.aic.json",
      "lastmod": "2025-12-08",
      "source_hash": "sha256:abc123...",
      "embeddings_inline": true
    }
  ]
}
```

### 3.3 Field Definitions

#### Root Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | Yes | Specification version (e.g., "0.1") |
| `site` | string | Yes | Base URL of the site |
| `generated_at` | string | No | ISO 8601 timestamp of manifest generation |
| `generator` | object | No | Information about the tool that generated the manifest |
| `pages` | array | Yes | Array of page entries |

#### Page Entry

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | Path to the original page (relative to site) |
| `aic` | string | Yes | Path to the AIC file (relative to site) |
| `lastmod` | string | No | ISO 8601 date of last modification |
| `source_hash` | string | No | SHA-256 hash of source content for verification |
| `embeddings_inline` | boolean | No | If true, embeddings are included in the AIC file |
| `embeddings` | array | No | Array of external embedding files (see Section 5) |

---

## 4. AIC File Format

### 4.1 Overview

The AIC file (`.aic.json`) is the core content format. It contains semantic information about a page and optionally includes embedded vector representations.

### 4.2 Schema

```json
{
  "version": "0.1",
  "source_url": "https://example.com/article/my-post",
  "source_hash": "sha256:abc123...",
  "source_retrieved_at": "2025-12-08T10:30:00Z",
  "generated_at": "2025-12-09T10:00:00Z",
  "generator": {
    "name": "aic-gen",
    "version": "1.0.0",
    "model": "claude-3.5-sonnet"
  },
  "content": {
    "title": "Page Title",
    "summary": "Human-readable summary of the content...",
    "content_type": "article",
    "language": "en",
    "entities": ["Entity1", "Entity2"],
    "key_facts": ["Fact 1", "Fact 2"],
    "topics": ["Topic1", "Topic2"],
    "questions_answered": ["What is X?", "How does Y work?"]
  },
  "temporal": {
    "published": "2025-12-01",
    "modified": "2025-12-08",
    "events_mentioned": ["2025-11-15", "2026-01-01"]
  },
  "embeddings": [
    {
      "model": "text-embedding-3-small",
      "dimensions": 1536,
      "encoding": "base64",
      "data": "SGVsbG8gV29ybGQh..."
    }
  ]
}
```

### 4.3 Field Definitions

#### Root Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | Yes | Specification version |
| `source_url` | string | Yes | Canonical URL of the source page |
| `source_hash` | string | Recommended | SHA-256 hash of source content |
| `source_retrieved_at` | string | Recommended | When source was fetched for processing |
| `generated_at` | string | Yes | When this AIC file was generated |
| `generator` | object | Yes | Generator information |
| `content` | object | Yes | Semantic content representation |
| `temporal` | object | No | Time-related metadata |
| `embeddings` | array | No | Inline embedding vectors |
| `embeddings_external` | boolean | No | If true, embeddings available as separate files |

#### Generator Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Name of the generation tool/service |
| `version` | string | Yes | Version of the generator |
| `model` | string | No | AI model used for generation (if applicable) |
| `contact` | string | No | Security contact for the generator |

#### Content Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Page title |
| `slug` | string | No | URL-friendly identifier (e.g., "my-post") |
| `summary` | string | Yes | Human-readable content summary |
| `content_type` | string | Yes | Content classification (see 4.4) |
| `language` | string | No | ISO 639-1 language code |
| `entities` | array | No | Named entities mentioned |
| `key_facts` | array | No | Key claims or facts |
| `topics` | array | No | Topic classifications |
| `questions_answered` | array | No | Questions this content addresses |

#### Temporal Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `published` | string | No | Original publication date |
| `modified` | string | No | Last modification date |
| `events_mentioned` | array | No | Dates of events discussed in content |

#### Embedding Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | string | Yes | Embedding model identifier |
| `dimensions` | integer | Yes | Vector dimensionality |
| `encoding` | string | Yes | Data encoding ("base64", "float32", "float16") |
| `data` | string | Yes | Encoded embedding data |

### 4.4 Content Types

Recommended values for `content_type`:

- `article` - News article, blog post, essay
- `documentation` - Technical documentation, guides
- `product` - Product page, e-commerce listing
- `profile` - Person, organization, or entity profile
- `reference` - Encyclopedia, dictionary, factual reference
- `tutorial` - How-to guide, instructional content
- `discussion` - Forum thread, comments, Q&A
- `media` - Primarily audio/video content description
- `index` - List page, category page, archive

Sites MAY use custom content types prefixed with `x-` (e.g., `x-recipe`, `x-legal-document`).

---

## 5. External Embeddings

### 5.1 Purpose

For sites serving multiple embedding formats or requiring optimal transfer efficiency, embeddings MAY be provided as separate binary files.

### 5.2 Manifest Declaration

When embeddings are external, the page entry in the manifest includes an `embeddings` array:

```json
{
  "url": "/article/my-post",
  "aic": "/ai-context/my-post.aic.json",
  "embeddings_inline": false,
  "embeddings": [
    {
      "model": "text-embedding-3-small",
      "dimensions": 1536,
      "format": "binary",
      "encoding": "float32",
      "href": "/ai-context/my-post.emb"
    },
    {
      "model": "text-embedding-3-large",
      "dimensions": 3072,
      "format": "binary",
      "encoding": "float32",
      "href": "/ai-context/my-post.3large.emb"
    }
  ]
}
```

### 5.3 File Naming Recommendations

The following extensions are RECOMMENDED but not required:

- `.aic.json` - Semantic AIC file
- `.emb` - Binary embedding file (primary recommendation)
- `.emb.json` - JSON-wrapped embedding (Base64 encoded)

Sites MAY use any naming scheme provided paths are correctly declared in the manifest.

### 5.4 Binary Format

Binary embedding files contain raw vector data:

- **Encoding:** As declared in manifest (`float32`, `float16`, `int8`)
- **Byte order:** Little-endian
- **Structure:** Contiguous array of numerical values
- **Size:** `dimensions × bytes_per_value`

Example: A 1536-dimension float32 embedding = 1536 × 4 = 6,144 bytes.

---

## 6. Security Considerations

### 6.1 Trust Model

AI agents MUST treat AIC content as **untrusted input**, equivalent to raw HTML. The format provides efficiency, not trust.

### 6.2 Source Verification

To detect tampering or staleness, agents SHOULD:

1. Verify `source_url` matches the expected page
2. Optionally fetch source and compare against `source_hash`
3. Check `source_retrieved_at` for acceptable freshness

### 6.3 Content Integrity

AIC files MUST NOT contain:

- Executable code
- Template expansion or variable interpolation
- External resource loading directives (except source URL)
- Recursive or self-referential structures

### 6.4 Embedding Security

Embeddings present unique security challenges as they cannot be human-audited. Agents SHOULD:

- Implement statistical anomaly detection
- Cross-reference embedding behavior against `summary` field
- Apply rate limiting per domain
- Maintain blocklists for known-malicious generators

### 6.5 Generator Accountability

The `generator` field enables accountability. If a generator produces malicious content, it can be identified and blocklisted.

---

## 7. Conformance Levels

### 7.1 Level 1: Minimal

Site provides:
- `/.well-known/ai-context.json` manifest
- At least one `.aic.json` file per declared page
- Required fields in all files

### 7.2 Level 2: Enhanced

Level 1 requirements plus:
- `source_hash` for content verification
- At least one embedding (inline or external)

### 7.3 Level 3: Comprehensive

Level 2 requirements plus:
- Multiple embedding formats for different models
- Complete temporal metadata
- Full entity and topic extraction

---

## 8. Implementation Notes

### 8.1 Agent Indexing Strategies

Agents building search indices from AIC files may consider:

- **Manifest caching:** Cache manifests with appropriate TTL, refresh based on `generated_at`
- **Incremental updates:** Use `lastmod` to identify changed content
- **Embedding indexing:** Store embeddings in vector databases for similarity search
- **Hash verification:** Periodically verify `source_hash` to detect stale AIC files

### 8.2 Generation Strategies

Sites generating AIC files may consider:

- **Build-time generation:** Generate during site build/deploy
- **On-demand generation:** Generate when first requested, then cache
- **Periodic refresh:** Regenerate on schedule to capture updates
- **Model selection:** Choose embedding models based on target agent ecosystem

### 8.3 Caching Recommendations

- Manifests: Cache for 1-24 hours depending on site update frequency
- AIC files: Cache based on `lastmod`, invalidate when source changes
- Embeddings: Long-term cacheable if content unchanged

---

## 9. IANA Considerations

### 9.1 Well-Known URI Registration

This specification requests registration of the well-known URI:

- **URI suffix:** ai-context.json
- **Change controller:** [TBD]
- **Specification document:** This document
- **Related information:** Used for AI agent content discovery

### 9.2 Media Type

AIC files use standard JSON media type:

```
Content-Type: application/json
```

Sites MAY use a more specific type:

```
Content-Type: application/vnd.ai-context+json
```

---

## 10. Future Considerations

The following are explicitly deferred to future versions:

- Signed AIC files for verified publishers
- Transparency log infrastructure
- Multi-language content variants
- Streaming/real-time content updates
- Training acceleration applications

---

## Appendix A: Complete Example

### A.1 Manifest

**GET /.well-known/ai-context.json**

```json
{
  "version": "0.1",
  "site": "https://example.com",
  "generated_at": "2025-12-09T10:00:00Z",
  "generator": {
    "name": "example-aic-generator",
    "version": "1.0.0"
  },
  "pages": [
    {
      "url": "/blog/hello-world",
      "aic": "/ai-context/blog/hello-world.aic.json",
      "lastmod": "2025-12-08",
      "source_hash": "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
      "embeddings_inline": true
    },
    {
      "url": "/about",
      "aic": "/ai-context/about.aic.json",
      "lastmod": "2025-12-01",
      "embeddings_inline": false,
      "embeddings": [
        {
          "model": "text-embedding-3-small",
          "dimensions": 1536,
          "format": "binary",
          "encoding": "float32",
          "href": "/ai-context/about.emb"
        }
      ]
    }
  ]
}
```

### A.2 AIC File (Inline Embeddings)

**GET /ai-context/blog/hello-world.aic.json**

```json
{
  "version": "0.1",
  "source_url": "https://example.com/blog/hello-world",
  "source_hash": "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "source_retrieved_at": "2025-12-08T10:30:00Z",
  "generated_at": "2025-12-09T10:00:00Z",
  "generator": {
    "name": "example-aic-generator",
    "version": "1.0.0",
    "model": "claude-3.5-sonnet"
  },
  "content": {
    "title": "Hello World: Our First Post",
    "slug": "hello-world",
    "summary": "An introductory blog post welcoming readers to the site and outlining upcoming content plans covering technology, AI, and web development.",
    "content_type": "article",
    "language": "en",
    "entities": ["Example Corp", "AI", "Web Development"],
    "key_facts": [
      "Site launched December 2025",
      "Focus areas include AI and web technology"
    ],
    "topics": ["introduction", "announcements", "technology"],
    "questions_answered": [
      "What is this site about?",
      "What content will be published?"
    ]
  },
  "temporal": {
    "published": "2025-12-08",
    "modified": "2025-12-08"
  },
  "embeddings": [
    {
      "model": "text-embedding-3-small",
      "dimensions": 1536,
      "encoding": "base64",
      "data": "AACAPs3MTD0AAIA+..."
    }
  ]
}
```

### A.3 AIC File (External Embeddings)

**GET /ai-context/about.aic.json**

```json
{
  "version": "0.1",
  "source_url": "https://example.com/about",
  "source_hash": "sha256:d7a8fbb307d7809469ca9abcb0082e4f8d5651e46d3cdb762d02d0bf37c9e592",
  "source_retrieved_at": "2025-12-01T09:00:00Z",
  "generated_at": "2025-12-01T09:15:00Z",
  "generator": {
    "name": "example-aic-generator",
    "version": "1.0.0",
    "model": "claude-3.5-sonnet"
  },
  "content": {
    "title": "About Example Corp",
    "slug": "about",
    "summary": "Company information page describing Example Corp's mission, team, and history since founding in 2020.",
    "content_type": "profile",
    "language": "en",
    "entities": ["Example Corp", "Jane Doe", "San Francisco"],
    "key_facts": [
      "Founded in 2020",
      "Headquartered in San Francisco",
      "Team of 50 employees"
    ],
    "topics": ["company", "about", "team"]
  },
  "temporal": {
    "published": "2024-01-15",
    "modified": "2025-12-01"
  },
  "embeddings_external": true
}
```

---

## Appendix B: Revision History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-12-09 | Initial draft |

---

## Appendix C: Acknowledgments

[TBD]

---

*End of Specification*
