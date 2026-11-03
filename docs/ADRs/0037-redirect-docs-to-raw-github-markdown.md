---
title: "37. Redirect documentation URLs to raw GitHub markdown"
status: Accepted
relates_to:
  - contributor-guidance
topics:
  - documentation
  - hosting
  - cloudflare
---

# 37. Redirect documentation URLs to raw GitHub markdown

Date: 2026-11-03

## Status

Accepted

## Context

The documentation site is hosted on a Cloudflare Worker with static assets
under the free tier, which limits deployments to 20,000 static asset files.
Each versioned documentation build adds approximately 1,650 files on top of a
~6,500-file base site. A rolling window of six minor versions was introduced
(#8332) and the minimum hosted version was raised from 37 to 40 to stay under
the ceiling, but the constraint reappears with every new release.

The site layout is governed by
[ADR 0019](0019-web-source-and-cloudflare-site-layout.md), which separates
browser-oriented source content (`web/`) from the Wrangler project boundary
(`cloudflare_site/`). The VitePress build pipeline (`npx mvb docs`) produces
rendered HTML, a version switcher, and client-side search — features that
depend on the static hosting model.

This ADR aims to eliminate the recurring static-asset ceiling by removing the
dependency on Cloudflare-hosted rendered documentation.

## Options

### Continue rolling-window pruning

Keep removing old versions to stay under limits. This works but requires
ongoing maintenance and discards historical documentation that users may need.

### Redirect documentation URLs to raw GitHub markdown

Use HTML meta-tag redirects so documentation URLs resolve to raw markdown
files in the repository on GitHub. This eliminates hosting costs entirely at
the expense of rendered styling, navigation, and search.

### Upgrade to a paid Cloudflare tier

Remove the asset ceiling with a paid plan. This preserves the current
experience but introduces recurring cost.

## Decision

We adopt HTML meta-tag redirects from documentation URLs to raw GitHub
markdown files to eliminate the Cloudflare free-tier static-asset constraint.

Each existing `/docs/` path will serve a lightweight HTML page containing a
`<meta http-equiv="refresh">` tag that redirects the browser to the
corresponding raw markdown file in the `fullsend-ai/fullsend` repository on
GitHub. The landing page (`web/public/index.html`), `robots.txt`, and
`llms.txt` remain served by the Cloudflare Worker as they are today.

VitePress-rendered features — client-side search, the version switcher, and
themed navigation — are no longer available through the hosted site. Readers
access markdown rendered by GitHub's own interface instead. The VitePress
build pipeline (`npx mvb docs`) and the multi-version build cache are retired
from CI.

The `.md` source files are already advertised alongside `.html` output via
`<link rel="alternate" type="text/markdown">` headers in the current build,
so this approach formalizes a path that partially exists.

## Forge scope

**GitHub:** This decision relies on GitHub's raw content serving
(`raw.githubusercontent.com`) for markdown delivery and GitHub's web UI for
rendered reading. Redirect targets point to the `fullsend-ai/fullsend`
repository.

**GitLab / Forgejo:** Not applicable. The documentation site is a single
deployment that serves docs for all forges; it is not forge-specific
infrastructure. If the project mirrors to another forge in the future, the
redirect targets would need updating.

## Consequences

- The 20,000-file static-asset ceiling is no longer a constraint; the Worker
  serves only redirect stubs and a handful of static files.
- Historical documentation versions are accessible via GitHub's branch/tag
  navigation rather than being pruned by a rolling window.
- Rendered styling, client-side search, and the version switcher are lost;
  readers rely on GitHub's markdown rendering and repository navigation.
- Search-engine indexing and link-preview metadata will degrade because
  meta-refresh redirects are not followed by all crawlers.
- The Build Site workflow simplifies significantly — the VitePress/mvb build
  pipeline and its CI cache can be removed.
