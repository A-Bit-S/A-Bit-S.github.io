# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A_Bit_S Blog is a Jekyll-based publishing platform for technical content, primarily in Chinese. The site features both individual blog posts and serialized book-length content (books with chapters). It's deployed to GitHub Pages at `https://wzxzhuxi.github.io/`.

## Core Commands

### Development
```bash
# Install dependencies (run once after clone)
bundle install

# Start dev server with live reload (default port: 4000)
bundle exec jekyll serve --livereload

# Production build
JEKYLL_ENV=production bundle exec jekyll build

# Build with full error trace
bundle exec jekyll build --trace

# Pre-deployment health check
bundle exec jekyll doctor
```

### Validation workflow
Before pushing or opening PR:
1. `bundle exec jekyll build --trace` - must complete with zero warnings
2. `bundle exec jekyll doctor` - catches broken permalinks, missing assets
3. Manual browser check of `_site/` output, verify responsive layout

## Architecture

### Collections system
The site uses two custom Jekyll collections beyond standard posts:

- **`_books/`**: Book metadata files with front matter defining `title`, `slug`, `author`, `description`, `cover`, `order`
  - Each book's `slug` field acts as the primary key linking to its chapters
  - Books appear at `/books/:slug/` via permalink pattern in `_config.yml`

- **`_chapters/`**: Individual chapter content files
  - Must specify `book: <slug>` matching a book's slug field
  - Must include `order` field to control sequence
  - Supports custom `permalink` or inherits default output path
  - The `book.html` layout queries chapters via `site.chapters | where: "book", book_slug`

### Layout hierarchy
```
_layouts/default.html          # Base wrapper with <html>, <head>, site navigation
  ├─ _layouts/home.html        # Homepage post grid, uses pagination
  ├─ _layouts/post.html        # Single article view
  ├─ _layouts/page.html        # Static pages (team.md, etc.)
  ├─ _layouts/book.html        # Book landing page, dynamically lists chapters
  └─ _layouts/chapter.html     # Individual chapter content
```

Key layout behaviors:
- `book.html:17-18` - Queries all chapters matching current book slug, sorts by `order` field
- `post.html:8-12` - Displays metadata: date, author, categories in header
- `home.html` - Renders post cards in grid, handles pagination via jekyll-paginate plugin

### Front matter conventions
All front matter keys use **snake_case** (e.g., `reading_time`, `cover_alt`), never camelCase.

**Post template** (`_posts/YYYY-MM-DD-slug.md`):
```yaml
---
layout: post
title: "Article Title"
author: author_slug        # String identifier (e.g., "zhuxi", "allen")
tags: [tag1, tag2]
excerpt: >
  1-2 sentence summary for listing pages
cover: /assets/images/posts/example.png  # Optional
---
```

Note: The `author` field is currently a simple string, not a reference to the `authors/` directory. The authors directory exists but contains only an index page.

**Book template** (`_books/<slug>.md`):
```yaml
---
title: Book Title
slug: unique-book-identifier  # Must match chapter `book` field
author: Author Name
description: Brief description for metadata/listings
cover: /assets/images/books/cover.png  # Optional
order: 1                       # Controls sort order in book listings
---
Book synopsis or reading guide...
```

**Chapter template** (`_chapters/<book-slug>-NN.md`):
```yaml
---
title: Chapter Title
book: unique-book-identifier   # MUST match parent book slug
order: 1
summary: Short blurb for navigation/ToC
permalink: /books/<book-slug>/<chapter-slug>/  # Optional custom path
---
Chapter content...
```

### Naming patterns
- Posts: `YYYY-MM-DD-slug.md` where slug is lowercase, digits, hyphens only
- Books: `<slug>.md` matching the slug field
- Chapters: Typically `<book-slug>-NN.md` or similar prefix for grouping

### Styling architecture
- Global styles: `assets/css/style.scss` (SCSS with nesting, compiled by Jekyll)
- Inline styles: Some layouts embed `<style>` blocks for component-specific rules (e.g., `book.html:45-182`)
- Class naming: BEM-style, max 2 levels of nesting
- Responsive breakpoints: Mobile handled via `@media (max-width: 768px)`

## Critical Files

- `_config.yml` - Site metadata, collection definitions, plugin configuration, navigation links, pagination settings
  - **Must restart `jekyll serve` after editing this file**
  - Defines `collections.books.output: true` and `collections.chapters.output: true`
  - Sets default layout per collection via `defaults` array

- `Gemfile` - Uses `github-pages` gem for GitHub Pages compatibility
  - Includes Ruby 3.4 stdlib shims (`erb`, `logger`, `csv`, `base64`, `bigdecimal`) - these gems are no longer bundled in Ruby 3.4+
  - If you get "cannot load such file" errors for these stdlib components, they're already declared in the Gemfile

- `_site/` - **NEVER edit manually**. Auto-generated output, excluded from git, disposable

- `_includes/` - Currently empty. This directory is for reusable Liquid template fragments that can be included in layouts

## Content authoring

### Markdown conventions
- Start article body at `##` (h2) - layouts provide `<h1>` from title
- Code blocks: Use triple backticks with language hint (e.g., ` ```c# `)
- Lists: Use `-` for bullets, 2-space indent for nesting
- Line length: Aim for 80-100 chars for better diffs
- Links: Prefer reference style for readability:
  ```markdown
  Text with [link][ref]

  [ref]: https://example.com
  ```

### Assets management
- Store all media/downloads under `assets/` hierarchy
- Reference with absolute paths: `/assets/images/posts/foo.png`
- Images get auto-bordered via CSS (`border: 3px solid #000`)

### Book vs post decision
- **Post**: Single-topic article, standalone content
- **Book**: Multi-chapter series requiring ordered navigation and unified branding
  - Create book file in `_books/`
  - Add chapter files in `_chapters/` with matching `book` slug
  - Book page auto-generates chapter listing via Liquid query

## Deployment

GitHub Pages deployment workflow:
1. Push to main branch
2. GitHub Actions (or Pages branch) runs `JEKYLL_ENV=production bundle exec jekyll build`
3. Publishes `_site/` directory to Pages hosting
4. Never commit `_site/` to repo (it's in `.gitignore`)

## Existing content reference

Current site structure:
- **Books**: `cpp-functional-programming` (C++ functional programming series by Zhuxi) - 10 chapters covering setup through advanced patterns
- **Posts**: Placeholder articles and `.NET Core Database` tutorial (ADO.NET patterns, transactions, stored procedures)
- **Pages**: `team.md` team roster, `books/index.md` book listing

The codebase demonstrates the collections pattern in action: `_books/cpp-functional-programming.md` defines metadata, and chapters live in `_chapters/cpp-fp-*.md` files.

## Common pitfalls

- **Forgetting to restart jekyll serve**: Required after `_config.yml` changes
- **Wrong front matter keys**: Use `book:` not `book_slug:` in chapters, use `slug:` in books
- **Mismatched slugs**: Chapter `book` field must exactly match book `slug` field
- **Missing `order` fields**: Causes unpredictable chapter sorting
- **Starting content at `#`**: Clashes with layout-provided h1, start at `##`
- **Committing `_site/`**: Build output, should never be in version control

## Testing chapter/book relationships

When adding new chapters:
1. Verify book slug matches: `grep "slug:" _books/<book>.md` and `grep "book:" _chapters/<chapter>.md`
2. Check chapter ordering: `grep "order:" _chapters/<book>-*.md`
3. Build and verify book page lists all chapters at `/books/<slug>/`
4. Confirm chapter permalinks resolve correctly

## Plugins in use

GitHub Pages-compatible plugins (defined in `_config.yml`):
- `jekyll-feed` - RSS/Atom feed generation
- `jekyll-seo-tag` - Meta tags for SEO
- `jekyll-sitemap` - XML sitemap
- `jekyll-paginate` - Pagination (configured for 6 posts per page)
