# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal portfolio website for Khairi Abidi, hosted via GitHub Pages at `abidikhairi.github.io` (deploys automatically from the `main` branch — pushing to `main` is a live deploy).

The site is a **Jekyll** site. GitHub Pages builds Jekyll natively, so there is still no CI/build step to manage — push to `main` and GitHub builds + serves it. Styling is Tailwind CSS and Font Awesome pulled in via CDN `<script>`/`<link>` tags (in `_layouts/default.html`), plus a small inline `<style>` block and inline `<script>` (mobile menu toggle, scroll-triggered navbar shadow) in the same layout file.

**Visual style is intentionally monochrome: black, navy, and neutral grays only — no other hues.** `_layouts/default.html` extends Tailwind's config with a custom `navy` color scale (`navy-50` through `navy-950`) via an inline `tailwind.config` script before the Play CDN script runs. Every accent (headings, buttons, badges, icons, links) uses `navy-*`/`black`/`white`/`gray-*` classes. Because color is no longer per-item, the `_data/*.yml` files don't carry `color` fields — skills and project tags are plain string lists, and templates hardcode the navy styling. Keep it this way: don't reintroduce a `color:` field or other hues without being asked.

## Structure

This is a **multi-page** Jekyll site — each top-level section is its own page, not a stacked single-page scroll:

- `index.html` — **About** page (`/`). Just the hero block (name/role/photo) plus a short "About Me" summary. Deliberately minimal — don't add other sections here.
- `research.html` (`/research/`), `projects.html` (`/projects/`), `skills.html` (`/skills/`), `experience.html` (`/experience/`) — one page per section at the repo root. Each is just front matter (`layout: default`, `title`, `permalink`) plus a single `{% include %}` of the matching partial in `_includes/`. There is no Contact page/section — it was removed.
- `_layouts/default.html` — shared page chrome: `<head>` (Tailwind/Font Awesome CDN tags, custom `<style>` block), nav/footer includes, and the inline `<script>` block at the bottom.
- `_includes/` — one partial per section (`hero.html`, `about.html`, `research.html`, `projects.html`, `skills.html`, `experience.html`, `nav.html`, `footer.html`), plus reusable card partials (`project-card.html`, `timeline-item.html`). The section partials (research/projects/skills/experience) all use `pt-24` top padding since each is now the first thing on its own page, directly under the fixed nav — keep that if you add another such page.
- `_data/` — **all editable content lives here as YAML**: `site.yml` (nav — now real page URLs, not anchors — hero, short about summary, research, social links, footer), `projects.yml`, `skills.yml`, `experience.yml`, `education.yml`. To update bio text, add a project, or change skills/experience/education, edit these files — not the HTML partials.
- `_config.yml` — Jekyll site config (title, description, author, markdown engine).
- `Gemfile` — pulls in the `github-pages` gem so local builds match GitHub's build environment. `Gemfile.lock` is gitignored (platform-specific; GitHub Pages' native build doesn't read it).

## Working with this repo

- To change content: edit the relevant YAML file in `_data/`. Multi-line prose fields use YAML folded scalars (`>-`).
- To change layout/markup/styling for a section: edit the matching partial in `_includes/`. Shared chrome (head, CSS, scripts) lives in `_layouts/default.html`.
- The site uses the Tailwind **Play CDN**, which JIT-compiles whatever classes it finds in the rendered DOM in the browser — not a build-time purge step. The custom `navy` palette is declared inline in `_layouts/default.html` via `tailwind.config`. If this repo ever moves off the CDN build to a compiled Tailwind pipeline, that config needs to move to a real `tailwind.config.js` and any dynamically-built class strings would need to become static/safelisted.
- No package manager/npm build step. For local preview, either:
  - Ruby + Bundler locally: `bundle install && bundle exec jekyll serve`
  - Docker (no local Ruby needed): `docker run --rm -v "$PWD":/srv/jekyll -p 4000:4000 jekyll/jekyll:latest jekyll serve --host 0.0.0.0` (note: this repo's Docker sandbox has no container networking, so builds there use `--network none` and `jekyll build` rather than `serve`)
- `_site/` and `.jekyll-cache/` are build output — gitignored, never commit them.
