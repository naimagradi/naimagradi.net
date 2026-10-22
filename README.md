# naima.gradi.net

Personal website of Naïma Gradi, built with [Hugo](https://gohugo.io/) and the [Blowfish](https://blowfish.page/)
theme, in English and French.

## Development

Requires [Hugo extended](https://gohugo.io/installation/) (latest) and Git.

```sh
git clone --recurse-submodules https://github.com/mscheltienne/naima.gradi.net.git
cd naima.gradi.net
hugo server                  # http://localhost:1313/
hugo build --gc --minify     # production build into public/
uvx pre-commit run --all-files   # linters (Biome and markdownlint need Node.js)
```

Pushes to `main` are built and deployed to GitHub Pages by `.github/workflows/build.yaml`.

## Content

Every page exists in English and French (`*.en.md` / `*.fr.md`).

| What | Where |
| --- | --- |
| Name, headline, photo, links, CV path, expertise | `config/_default/languages.<lang>.toml` |
| Home page text | `content/_index.<lang>.md` (bio before `<!--more-->`) |
| Projects | `content/projects/<name>/index.<lang>.md`, with their `images/` |
| Publications, talks and posters | `data/publications.yaml`, `data/talks.yaml` (PDFs in `static/files/`) |
| CV page | `data/<lang>/cv.yaml` |
| Media | `content/media/index.<lang>.md` |
| Labels and menus | `i18n/<lang>.yaml`, `config/_default/menus.<lang>.toml` |

## Theme

Blowfish is a git submodule in `themes/blowfish` and is never edited: the site's layouts and shortcodes are in
`layouts/`, its styles in `assets/css/custom.css` and its colour palette in `assets/css/schemes/`.
