# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal portfolio website for Khairi Abidi, hosted via GitHub Pages at `abidikhairi.github.io` (deploys automatically from the `main` branch — pushing to `main` is a live deploy).

The site is a **Jekyll** site. GitHub Pages builds Jekyll natively, so there is still no CI/build step to manage — push to `main` and GitHub builds + serves it. Styling is Tailwind CSS and Font Awesome pulled in via CDN `<script>`/`<link>` tags (in `_layouts/default.html`), plus a small inline `<style>` block and inline `<script>` (mobile menu toggle, smooth-scroll navigation, scroll-triggered navbar shadow) in the same layout file.

## Structure

- `index.html` — the homepage. Just front matter (`layout: default`) plus a sequence of `{% include %}` calls, one per page section.
- `_layouts/default.html` — shared page chrome: `<head>` (Tailwind/Font Awesome CDN tags, custom `<style>` block), nav/footer includes, and the inline `<script>` block at the bottom.
- `_includes/` — one partial per page section (`hero.html`, `about.html`, `research.html`, `projects.html`, `skills.html`, `experience.html`, `contact.html`, `nav.html`, `footer.html`), plus reusable card partials (`project-card.html`, `timeline-item.html`).
- `_data/` — **all editable content lives here as YAML**: `site.yml` (nav, hero, about, research, contact info, social links, footer), `projects.yml`, `skills.yml`, `experience.yml`, `education.yml`. To update bio text, add a project, or change skills/experience/education, edit these files — not the HTML partials.
- `_config.yml` — Jekyll site config (title, description, author, markdown engine).
- `Gemfile` — pulls in the `github-pages` gem so local builds match GitHub's build environment. `Gemfile.lock` is gitignored (platform-specific; GitHub Pages' native build doesn't read it).

## Working with this repo

- To change content: edit the relevant YAML file in `_data/`. Multi-line prose fields use YAML folded scalars (`>-`).
- To change layout/markup/styling for a section: edit the matching partial in `_includes/`. Shared chrome (head, CSS, scripts) lives in `_layouts/default.html`.
- Tailwind classes are built dynamically from data fields in some partials (e.g. `bg-{{ tag.color }}-100`). This works because the site uses the Tailwind **Play CDN**, which JIT-compiles classes found in the rendered DOM in the browser — not a build-time purge step. If this repo ever moves off the CDN build to a compiled Tailwind pipeline, these dynamic class strings will need to become static/safelisted.
- No package manager/npm build step. For local preview, either:
  - Ruby + Bundler locally: `bundle install && bundle exec jekyll serve`
  - Docker (no local Ruby needed): `docker run --rm -v "$PWD":/srv/jekyll -p 4000:4000 jekyll/jekyll:latest jekyll serve --host 0.0.0.0` (note: this repo's Docker sandbox has no container networking, so builds there use `--network none` and `jekyll build` rather than `serve`)
- `_site/` and `.jekyll-cache/` are build output — gitignored, never commit them.
