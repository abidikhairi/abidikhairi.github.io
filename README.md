# abidikhairi.github.io

My personal portfolio site — LLM/AI engineering and NLP-for-bioinformatics research.

Built with [Jekyll](https://jekyllrb.com/) and Tailwind CSS (via CDN), hosted on GitHub Pages.
Pushing to `main` deploys automatically — no separate build step.

## Pages

- `/` — About
- `/research/` — PhD research
- `/projects/` — Projects
- `/skills/` — Technical skills
- `/experience/` — Experience & education

## Editing content

All page content lives in `_data/*.yml` — edit those files, not the HTML, to update bio text,
projects, skills, or experience/education. See `CLAUDE.md` for the full structure.

## Local preview

No local Ruby needed — using Docker:

```bash
docker run --rm -v "$PWD":/srv/jekyll -p 4000:4000 jekyll/jekyll:latest \
  jekyll serve --host 0.0.0.0
```

Then open http://localhost:4000. Or, with Ruby + Bundler installed:

```bash
bundle install
bundle exec jekyll serve
```
