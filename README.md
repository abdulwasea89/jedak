# Jedak — Abdul Wasea

A static portfolio, research index, and technical publication. GitHub Pages deployment is configured for `abdulwasea89/jedak`.

## Local development

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
python build.py
python -m http.server 8000 --directory dist
```

Open `http://localhost:8000`. The build generates all pages, local assets, metadata, sitemap, RSS, and llms.txt. GSAP is an optional pinned CDN dependency; motion falls back to the browser's animation API if unavailable. Reduced-motion preferences are respected. No scroll hijacking.

## Writing

Create a file with a lowercase hyphenated name in `content/notes/`, for example `transactional-outbox.md`:

```markdown
---
title: A transactional outbox in practice
description: Explain the failure case and the implementation.
date: 2026-11-26
tag: Distributed systems
kind: blog
published: true
reviewed: true
---

Start with the problem. Use a concrete scenario.

## Implementation

Include code, constraints and tradeoffs.

## Failure tests

Show what happens when dependencies fail.

## References

Link to primary sources.
```

`kind: essay` places a piece in the Essays archive. Set `published: false` to keep a post off the site. `published: true` with `reviewed: false` makes it visible as an editorial draft, excludes it from the sitemap, RSS and llms.txt, and sets noindex. The seven starter articles are in this state; review, revise, and fact-check them before setting `reviewed: true`. Don't claim production results unless you can substantiate them. `noindex` is not privacy; use `published: false` if you do not want the draft publicly accessible. Existing earlier sample posts remain unpublished.

Images: store them in `content/media/`, and link from a post with `![Description](../media/name.png)`.

## Edit profile and research

`content/site.json` controls projects, experience, links, research summaries and paper records. The profile introduction in `build.py` can be changed directly. Research summaries are paraphrased portfolio descriptions, not verified verbatim abstracts. `assets/site.css` controls the design; `assets/logo.svg` is the logo and favicon.

## Update the existing GitHub site

Copy the files from this release into your existing cloned `jedak` repository, keeping its `.git` folder. Review `git diff` before committing. Then:

```bash
python build.py
git add -A
git commit -m "feat: redesign portfolio and expand technical publishing"
git push origin main
```

The existing workflow deploys automatically when GitHub Pages uses GitHub Actions. Canonical URL: https://abdulwasea89.github.io/jedak/ . Change `site_url` in `content/site.json` if you switch domains. The old `writing.html` path redirects to `blogs.html`.

## Search and discoverability

Static semantic HTML, canonical URLs, titles, descriptions, Open Graph metadata, Person/WebSite/BlogPosting/ScholarlyArticle JSON-LD, sitemap, RSS and llms.txt. Drafts are excluded from publication feeds. These measures facilitate crawling but do not guarantee indexing or inclusion in AI answers. GitHub Pages project sites cannot set the domain-root robots.txt from this repository. Submit the sitemap via Search Console if you can verify the property.
