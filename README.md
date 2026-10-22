# naima.gradi.net — design proposal: Blowfish (hybrid)

Source of the personal academic website of **Naïma Gradi** (psychology and mental health, University of Geneva),
built with [Hugo](https://gohugo.io/) and deployed to GitHub Pages.

> **Branch `design/blowfish-hybrid`** combines the two shortlisted proposals: it starts from
> `design/blowfish-basic` (multi-page site, simple professional look, mobile layout) and brings over what was liked
> in `design/hugoblox` (profile column with round link buttons, education tiles, interests, better use of wide
> screens), without its gradient background or its single-page home. All content is **placeholder** content. The
> sections below are inherited from `design/blowfish-basic` and still describe it unless stated otherwise.

| | This branch |
| --- | --- |
| Theme | Blowfish v3.8.0 (git submodule `themes/blowfish`), unmodified |
| Home page | Custom layout (`layouts/partials/home/custom.html`, `homepage.layout = "custom"`): profile column (photo, name, headline, affiliation, round e-mail/ORCID/Scholar/LinkedIn buttons) next to the bio, a "Download CV" button, education tiles and interest pills; below, full width, the content after `<!--more-->` in `content/_index.<lang>.md` (research, recent publications). Stacks into one column below 900px |
| Other pages | CV (`/en/cv/`, education as tiles), Publications and talks (`/en/publications/`), both at full content width |
| Page width | 88rem with 3rem gutters from 768px (Blowfish: 80rem with 8rem gutters, 65ch reading width) |
| Profile data | `[params.profile]` in `config/_default/languages.<lang>.toml`: `affiliation`, `affiliationUrl`, `cv`, `interests` |
| Default palette | B · Mint & Pine (`colorScheme = "palette-b-mint-pine"`, approved), floating demo picker for A/B/C |
| Fonts | Blowfish default (system sans-serif stack) |
| Toolchain | Hugo only |

## Contents

- [Quick start](#quick-start)
- [Repository layout](#repository-layout)
- [Editing the content](#editing-the-content)
- [This design in detail](#this-design-in-detail)
- [The four design proposals](#the-four-design-proposals)
- [Colour palettes](#colour-palettes)
- [Typography](#typography)
- [Languages (English / French)](#languages-english--french)
- [ORCID linking and search-engine metadata](#orcid-linking-and-search-engine-metadata)
- [Tooling](#tooling)
- [Continuous integration](#continuous-integration)
- [Deployment (GitHub Pages)](#deployment-github-pages)
- [Background: how we got here](#background-how-we-got-here)
- [Decision log](#decision-log)
- [Open questions and to-do before launch](#open-questions-and-to-do-before-launch)
- [Side notes on mathieu.scheltienne.net](#side-notes-on-mathieuscheltiennenet)

## Quick start

Prerequisites: [Hugo extended](https://gohugo.io/installation/) ≥ 0.162 (latest recommended) and Git. Nothing
else: Blowfish ships its compiled CSS, so neither Node.js nor Go is needed to build the site (Node is only used
by two pre-commit hooks).

```sh
git clone --recurse-submodules -b design/blowfish-basic https://github.com/mscheltienne/naima.gradi.net.git
cd naima.gradi.net
hugo server                # http://localhost:1313/ (redirects to /en/)
hugo build --gc --minify   # production build into public/
```

If the submodule is missing (cloned without `--recurse-submodules`): `git submodule update --init`.

## Repository layout

```text
.
├── .github/
│   ├── dependabot.yaml          # GitHub Actions, submodule and pre-commit updates
│   └── workflows/build.yaml     # build (all PRs/pushes) and deploy to GitHub Pages (main)
├── assets/
│   ├── css/custom.css           # styles of the CV / publication / talk components (+ demo picker)
│   ├── css/palette-picker.css   # demo only: live palette override
│   ├── css/schemes/palette-*.css  # palettes A/B/C as Blowfish colour schemes
│   └── img/avatar.png           # profile photo (placeholder silhouette)
├── config/_default/
│   ├── hugo.toml                # site-wide settings (theme, languages in sub-directories, Hugo version)
│   ├── languages.en.toml        # English: title, description, author (name, headline, bio, profile links)
│   ├── languages.fr.toml        # French equivalents
│   ├── menus.en.toml / menus.fr.toml  # header menu per language
│   └── params.toml              # Blowfish parameters (colour scheme, header, footer, home layout…)
├── content/
│   ├── _index.en.md / .fr.md    # home page text (bio, research, recent publications)
│   ├── cv/index.en.md / .fr.md  # CV page ({{< cv >}})
│   └── publications/index.en.md / .fr.md  # publications + talks ({{< publications >}}, {{< talks >}})
├── data/
│   ├── publications.yaml        # all publications (shared by both languages)
│   ├── talks.yaml               # all talks and posters (shared by both languages)
│   ├── en/cv.yaml / fr/cv.yaml  # CV entries per language
│   └── palettes.json            # demo only: palette names for the picker
├── i18n/en.yaml / fr.yaml       # strings of our components (section titles, badges…)
├── layouts/
│   ├── partials/extend-head.html    # JSON-LD Person (ORCID sameAs) + demo picker bootstrap
│   ├── partials/extend-footer.html  # demo palette picker
│   └── shortcodes/cv.html, publications.html, talks.html, i18n.html
├── themes/blowfish/             # git submodule
├── .pre-commit-config.yaml, .yamllint.yaml, .markdownlint-cli2.yaml, biome.json, pyproject.toml, .gitattributes
└── README.md
```

## Editing the content

| What | Where |
| --- | --- |
| Name, headline, short bio line, profile links (e-mail, ORCID, Scholar, LinkedIn…) | `[params.author]` in `config/_default/languages.en.toml` and `languages.fr.toml` |
| Photo | `assets/img/avatar.png` (path set by `params.author.image`) |
| Home page text | `content/_index.en.md` / `_index.fr.md` (Markdown; `{{< publications limit=3 >}}` inserts the latest papers) |
| Publications | `data/publications.yaml` — one entry per paper: `title`, `authors` (list; the entry equal to the author name is shown in bold), `year`, `venue`, `type` (`article`, `preprint`…), `doi`, optional `pdf`. **One list for both languages.** |
| Talks and posters | `data/talks.yaml` — `title`, `event`, `location`, `date`, `type` (`talk`, `poster`) |
| CV | `data/en/cv.yaml` and `data/fr/cv.yaml` — `experience`, `education`, `awards` (each: `title`, `org`, `start`, `end`, optional `summary`) and `languages` (`name`, `level`) |
| Section titles and labels of the components | `i18n/en.yaml` / `i18n/fr.yaml` |
| Menus | `config/_default/menus.<lang>.toml` |

## This design in detail

### Pages and components

- **Home**: Blowfish's `profile` layout (centred photo, name, headline, profile icons from `params.author`), the
  bio and a "Research" paragraph from `content/_index.<lang>.md`, then "Recent publications" (the three most
  recent entries of `data/publications.yaml`) and a link to the full list.
- **Publications** (`/en/publications/`): all publications, newest first, as a two-column list (year |
  title linked to the DOI, "Preprint" badge, authors with the owner in bold, venue with DOI/PDF links), followed
  by "Talks & posters" with a type badge, event, location and localised date.
- **CV** (`/en/cv/`): experience, education and awards as a year | entry list with a pastel left border, and
  languages as pills.
- Header: menu, language switch (EN/FR), light/dark toggle. Footer: copyright, light/dark toggle, scroll-to-top.
- Every page has a JSON-LD `Person` (name, job title, affiliation, `sameAs` ORCID/Scholar/LinkedIn).

### Theme installation and customisation

- Blowfish **v3.8.0** is a git submodule in `themes/blowfish` (shallow clone; `theme = "blowfish"` in
  `hugo.toml`). Dependabot proposes submodule bumps monthly (it follows the default branch, so check the Blowfish
  release notes before merging: <https://github.com/nunocoracao/blowfish/releases>).
- **The theme is never edited.** Everything specific to this site is added next to it and takes precedence
  through Hugo's lookup order:
  - `layouts/partials/extend-head.html` and `extend-footer.html` are Blowfish's official extension hooks
    (injected in every page's `<head>` and after the footer);
  - `layouts/shortcodes/*.html` (and `layouts/partials/…` in the richer variants) are our components;
  - `assets/css/custom.css` is loaded automatically by Blowfish after its own stylesheet;
  - `assets/css/schemes/<name>.css` defines colour schemes selectable with `colorScheme` in `params.toml`;
  - `assets/icons/<name>.svg` adds icons usable with Blowfish's `icon` partial/shortcode.
- Our CSS does not rely on Tailwind classes (Blowfish's CSS is precompiled, so only the classes used by the theme
  itself exist); it uses plain CSS reading the colour variables of the active scheme.
- Blowfish's attribution in the footer is disabled (`showThemeAttribution = false`); the theme allows it.

### Colour palettes in this branch

Blowfish colour schemes are CSS files defining three 10-step scales as RGB triplets: `--color-neutral-50…900`
(backgrounds and text), `--color-primary-50…900` (links, accents) and `--color-secondary-50…900`. The three
palettes are in `assets/css/schemes/palette-*.css`. Blowfish uses `neutral` (lightest) as the light background,
`neutral-700/800` for light-mode text, `primary-600` for light-mode links, and `neutral-800`, `neutral-300/400`
and `primary-400` in dark mode, so the scales were hand-tuned to put the pastels on the light end and the deep
greens where text and links need contrast.

- Default: `colorScheme = "palette-b-mint-pine"` in `config/_default/params.toml`.
- **Demo palette picker** (to be removed before launch): `assets/css/palette-picker.css` redefines the same
  variables under `html[data-palette="…"]`, `data/palettes.json` lists the palettes, and
  `layouts/partials/extend-footer.html` renders the floating "Palette" select at the bottom right (choice stored
  in `localStorage`); the matching early script is in `extend-head.html`. To finalise: set `colorScheme`, then
  delete `palette-picker.css`, `data/palettes.json`, the picker markup/script in the two partials and the
  `.palette-picker` rules in `custom.css`.
- The scales and the picker stylesheet were generated from one table of hex values (a small Node script used
  during the design phase); after choosing, edit the chosen scheme file directly.

### Caveats

- Deliberately plain: judged "too plain" during the review, which is why the scholar and landing variants exist.
- Only one CV PDF link is possible without extra work (none is shown in this variant).
- No news, no research themes, no abstract/BibTeX features (they exist in the two richer variants and can be
  ported: the components are independent partials).
- Hugo's live-reload server sometimes loses the link between translations when many files are rewritten at once
  (e.g. by a script); restart `hugo server` if the language switcher disappears. Production builds are not
  affected.

## The four design proposals

This repository currently holds **four competing design proposals**, one per branch. They share the same
generator (Hugo), the same repository layout, the same tooling, the same CI and the same placeholder content, so
they can be compared side by side and the chosen one can become `main` without restructuring anything.

| Branch | Theme | Look & feel | Toolchain |
| --- | --- | --- | --- |
| `design/hugoblox` | [Hugo Blox](https://hugoblox.com/) "Academic CV" template (`blox` module) | Rich, "academic template" look: bio card with gradient mesh, education cards, interests, citation list, CV timeline | Hugo + Node (Tailwind CSS v4 CLI) |
| `design/blowfish-basic` | [Blowfish](https://blowfish.page/) v3, `profile` home layout | Minimal and airy: centred profile, short publication list, simple CV timeline | Hugo only |
| `design/blowfish-scholar` | Blowfish v3 + custom home layout | Classic academic homepage: sticky profile card on the left, serif headings, news, research themes, publications with abstract/cite, talks | Hugo only |
| `design/blowfish-landing` | Blowfish v3 + custom home layout | Modern one-page personal site: pastel hero, stats strip, research cards, publication cards, news timeline, contact band | Hugo only |

The three Blowfish branches share the same components (publication list with abstract toggle and one-click
BibTeX, talks, CV timeline); `blowfish-scholar` and `blowfish-landing` add news posts, research themes, richer CV
sections and their own home page layout. The differences between the Blowfish branches are in `layouts/`,
`assets/css/custom.css` and `config/_default/`, plus the extra data and content of the richer variants.

### How to compare them

Each branch is checked out in its own git worktree on the machine where the proposals were created. On a new
machine:

```sh
git clone --recurse-submodules https://github.com/mscheltienne/naima.gradi.net.git
cd naima.gradi.net
# one worktree per proposal (optional, but convenient to compare them side by side)
for b in hugoblox blowfish-basic blowfish-scholar blowfish-landing; do
  git worktree add ../naima.gradi.net-worktrees/$b design/$b
  git -C ../naima.gradi.net-worktrees/$b submodule update --init
done
```

Then run `hugo server` in one worktree at a time (see [Quick start](#quick-start); use `--port` to run several at
once, memory permitting).

### Choosing and finishing

The decision was deliberately postponed until real content (bio, CV, publications) is written, because the
placeholder content makes all four look similar in quality. Once a proposal is chosen:

1. Merge (or fast-forward) its branch into `main` and delete the other `design/*` branches.
2. Pick one palette, set it as the default and **remove the demo palette picker** (see the design section below).
3. Replace the placeholder content, the avatar and the CV PDF; fill in the real ORCID, Google Scholar, e-mail and
   LinkedIn links.
4. Enable GitHub Pages (source: *GitHub Actions*) and, once a domain is chosen, configure it (see
   [Deployment](#deployment-github-pages)).
5. Rewrite this README into a short maintainer/editor guide (the research and decision log below can move to a
   `docs/` page or stay in the git history).

## Colour palettes

The brief was a **professional look with a green pastel palette**. Pastel greens are far too light to be used as
text colour (on white: `#B2D8B5` 1.57:1, `#A8D5BA` 1.63:1, `#9CAF88` 2.36:1, `#8FBC8F` 2.15:1, `#6B8F71`
3.63:1, all below the WCAG AA minimum of 4.5:1). Every palette therefore follows the same rule: **pastels only for
backgrounds, cards, tags and borders; deep greens for text and links**. Three palettes were designed and are
available in every branch through the demo palette picker:

**A · Sage & Eucalyptus** (soft, muted, slightly grey greens)

| Role | Light | Dark |
| --- | --- | --- |
| Background / surface | `#F6F8F5` / `#E6EFE8` | `#121A16` / `#1B2620` |
| Pastel accent | `#B9D4C1` | `#2C4235` |
| Text | `#1E2B24` (13.8:1 on background) | `#E3ECE5` (14.7:1) |
| Muted text | `#4B5D52` (6.6:1; 4.4:1 on the pastel, avoid there) | `#A6B8AC` (8.5:1) |
| Link | `#2F6B4F` (5.9:1; 3.97:1 on the pastel = large text only) | `#9CCFB2` (10.1:1) |
| Hover | `#1F4D38` (9.0:1) | `#C3E6D2` (13.1:1) |

**B · Mint & Pine** (crisp white base; the safest: links pass AA on every surface)

| Role | Light | Dark |
| --- | --- | --- |
| Background / surface | `#FFFFFF` / `#EEF7F2` | `#0F1714` / `#18231F` |
| Pastel accent | `#CDEBDD` | `#23372F` |
| Text | `#17241F` (16.1:1) | `#E6F0EB` (15.6:1) |
| Muted text | `#4A5E56` (6.9:1) | `#A3B8AF` (8.7:1) |
| Link | `#1B6B58` (6.4:1, 5.0:1 even on the pastel) | `#8FD6BD` (10.9:1) |
| Hover | `#124C3F` (9.9:1) | `#B9E8D6` (13.5:1) |

**C · Eucalyptus & Clay** (warm paper background with a terracotta second accent)

| Role | Light | Dark |
| --- | --- | --- |
| Background / surface | `#F8F7F2` / `#EBEFE9` | `#161B19` / `#1F2724` |
| Pastel accent | `#C6D7CD` | `#31423B` |
| Text | `#22302A` (12.8:1) | `#E4E9E5` (14.2:1) |
| Muted text | `#55635C` (5.9:1) | `#AAB6B0` (8.3:1) |
| Link | `#3A6657` (6.1:1) | `#A6CDBD` (10.0:1) |
| Hover | `#28493E` (9.3:1) | `#C9E2D7` (12.7:1) |
| Terracotta accent | `#9A4E38` (5.6:1) | `#E3A590` (8.3:1) |

Notes from the contrast checks:

- Stock theme colours were checked too: Tailwind `emerald-700 #047857` (5.48:1), `teal-700 #0f766e` (5.47:1) and
  `green-700 #15803d` (5.02:1) pass AA on white, `emerald-600 #059669` (3.77:1) fails. Hugo Blox's built-in green
  pack *Matcha* uses `lime-600 #65a30d` (3.09:1) and fails; Blowfish's *forest* scheme is neon teal
  (`primary-800 #078677` = 4.48:1), not pastel. Hence custom palettes in every branch.
- Link-versus-body-text contrast is only 1.4–2.5:1, below the 3:1 needed to distinguish links by colour alone
  (WCAG 1.4.1): **underline links in running text**.
- Pastel-versus-background contrast is ~1.3–1.6:1, so a pastel must never be the only boundary of a control.

How the palettes are implemented differs per theme (Blowfish: RGB scales `50`–`900` in
`assets/css/schemes/*.css`; Hugo Blox: theme packs in `data/themes/*.yaml`); see the design section of this
branch.

## Typography

Font pairings considered (all on Google Fonts, all with full French diacritics, all self-hostable):

1. **Source Serif 4** (headings) + **Source Sans 3** (body): sober and scholarly. *Used by `blowfish-scholar`.*
2. **Fraunces** (soft, low-contrast optical sizes) + **Atkinson Hyperlegible Next**: warm and accessibility-focused,
   a good fit for mental health. *Used by `blowfish-landing`.*
3. **Newsreader** or **Literata** + **Inter** or **Public Sans**: editorial and modern.
4. **Lora** + **Nunito Sans**: friendlier and calmer.

Hugo Blox ships typography packs instead (`academic` is used, `humanist` is close to option 2); `blowfish-basic`
keeps Blowfish's default system font stack.

**The demos load fonts from Google Fonts. The production site should self-host them** (e.g. files from
[Fontsource](https://fontsource.org/) or google-webfonts-helper in `static/fonts/` with an `@font-face` block in
`assets/css/custom.css`), because loading Google Fonts from Google's servers transmits visitors' IP addresses to
Google, which a German court ruled a GDPR violation (LG München, 2022); the same concern applies under the Swiss
nFADP. [Bunny Fonts](https://fonts.bunny.net/) is a privacy-friendly drop-in alternative.

## Languages (English / French)

Requirements: **English is the default language, French is available**, and content should be expressible in
both languages. Implementation, identical in all branches:

- **URLs**: English under `/en/`, French under `/fr/`; `/` redirects to `/en/`
  (`defaultContentLanguageInSubdir = true`). Serving English at both `/` and `/en/` is not possible in Hugo and
  would be duplicate content for search engines anyway. GitHub Pages cannot send HTTP redirects, so the redirect
  at `/` is a tiny HTML page with a `<meta http-equiv="refresh">` generated by Hugo. The alternative (English at
  `/`, French at `/fr/`) is a one-line change: `defaultContentLanguageInSubdir = false`.
- **Translated pages** sit side by side and share their page bundle (images, PDFs): `index.en.md` /
  `index.fr.md`, `_index.en.md` / `_index.fr.md`. Hugo links the two files as translations automatically; the
  theme's language switcher (header) jumps to the translation of the current page.
- **Untranslated pages** simply do not exist in the other language (no automatic fallback, no warnings — a
  deliberate choice to keep things simple).
- **Interface strings** (buttons, headings rendered by templates) come from the theme's bundled `fr` translation
  plus, for our own components, `i18n/en.yaml` / `i18n/fr.yaml` (Blowfish branches).
- **Per-language settings** (site description, author headline, profile text, menus) live in
  `config/_default/languages.<lang>.toml` and `config/_default/menus.<lang>.toml`.
- **Dates** are localised by Hugo (`septembre 2021`, `10 septembre 2025`…); the French locale is `fr-CH`.
- Language-neutral data (publication list, talks) is shared between languages where the theme allows it (all
  Blowfish branches); Hugo Blox models publications as pages, so each publication has an `.en.md` and a `.fr.md`
  file.

## ORCID linking and search-engine metadata

- Every branch shows ORCID, Google Scholar (and LinkedIn/ResearchGate/e-mail) icons in the profile.
- Every branch emits **JSON-LD structured data** describing the person, with `sameAs` pointing to ORCID, Scholar
  and the other profiles: this tells Google/Scholar that the site, the ORCID record and the papers belong to the
  same person. Blowfish branches: `layouts/partials/extend-head.html` (a `Person` with name, job title,
  affiliation, e-mail, research interests and `sameAs`). Hugo Blox generates its own JSON-LD (`Person`,
  `Article` for each publication, `WebSite`, breadcrumbs).
- **To do at launch**: add the site URL to the ORCID record ("Websites & social links") so the link goes both
  ways, and replace the placeholder ORCID iD `0000-0000-0000-0000`.
- Optional later: pull the publication list from the public ORCID API at build time (`resources.GetRemote`),
  which keeps the site in sync with ORCID at the cost of depending on ORCID's data quality and API availability
  (with a cached fallback). Not implemented.
- Optional later: Google Scholar's inclusion guidelines recommend one HTML page per paper with `citation_*` meta
  tags (`citation_title`, `citation_author`, `citation_publication_date`, `citation_pdf_url`,
  `citation_journal_title`). None of the branches emits them yet (Hugo Blox has per-paper pages but no
  `citation_*` tags; the Blowfish branches only have a list).

## Tooling

Kept deliberately small ("let's not over-build it"); mirrors the setup of
[mathieu.scheltienne.net](https://github.com/mscheltienne/mathieu.scheltienne.net).

| File | Purpose |
| --- | --- |
| `.pre-commit-config.yaml` | Hooks: **yamllint** (strict), **toml-sort** (formats/sorts the TOML config), **Biome** (lints and formats CSS/JS/JSON; `layouts/` and `themes/` are excluded because Go templates are not HTML), **markdownlint-cli2** (content and README) |
| `.yamllint.yaml` | yamllint rules (line length and document start disabled) |
| `pyproject.toml` | Python linter dependencies and the `[tool.tomlsort]` configuration (keys sorted inside tables; inline arrays such as the ordered list of profile links are **not** sorted) |
| `biome.json` | Biome configuration (2-space indent, 120 columns, double quotes) |
| `.markdownlint-cli2.yaml` | markdownlint configuration (no line-length limit, inline HTML allowed) |
| `.gitattributes` | Forces LF line endings so Windows, macOS and Linux checkouts produce identical files |
| `.github/dependabot.yaml` | Weekly GitHub Actions updates, monthly theme submodule and pre-commit hook updates (plus weekly npm updates in `design/hugoblox`) |

Running the hooks:

```sh
pip install pre-commit        # or: uv tool install pre-commit / pipx install pre-commit
pre-commit install            # run automatically on every commit
pre-commit run --all-files    # run on the whole repository
```

The Biome and markdownlint hooks need Node.js on the `PATH`. All hooks pass on every branch as committed.

Rejected or deferred on purpose: **cspell** / Vale / LanguageTool in CI (spell checking; LTeX+ in VS Code remains
a good *editor* aid for writing in both languages), **missing-translation warnings/checks** (e.g. Lunaria),
**Renovate** (overkill; Dependabot is enough), **PR preview deployments**, html-validate, Lighthouse/pa11y
accessibility CI, `gotmplfmt` (the official Go-template formatter by Hugo's lead developer; worth adding if the
custom templates grow). Note that `prettier-plugin-go-template` is unmaintained (last release 2023) and should
not be used.

## Continuous integration

`.github/workflows/build.yaml` (identical in all branches):

- Triggers: pull requests, pushes to `main`, manual dispatch.
- **Build job**: checkout with submodules → install Hugo extended **`latest`** (`peaceiris/actions-hugo@v3`) →
  set up Node and `npm ci` **only if `package-lock.json` exists** (Hugo Blox) → `actions/configure-pages` (pushes
  only) to get the URL GitHub Pages serves (custom domain or `https://<user>.github.io/<repo>/`) →
  `hugo build --gc --minify --baseURL <that URL>` → on pull requests, **lychee** checks every internal link of the
  built site offline → `actions/upload-pages-artifact`.
- **Deploy job** (pushes to `main` only): `actions/deploy-pages` to the `github-pages` environment.

**Hugo version policy** (same as mathieu.scheltienne.net): CI always installs the latest Hugo release and the
config sets a loose lower bound, `[module.hugoVersion] min = "0.162.0"`. This gives new Hugo features and fixes
for free; the risk is that Hugo is still pre-1.0 and occasionally breaks things (v0.146 rewrote the template
system, v0.153 introduced the "sites matrix" content model, v0.166 blocks org-mode content by default). A failed
build never takes the site down (deployments are atomic; the previous version stays online). If this ever
becomes a problem, pin `hugo-version` in the workflow and bump it by hand (Dependabot cannot bump a version
string inside a workflow).

The site is built as a *project page* path when there is no custom domain; all templates use `relURL` /
`relLangURL` / `.RelPermalink`, and a build with `--baseURL https://<user>.github.io/naima.gradi.net/` was
checked to produce correct links.

## Deployment (GitHub Pages)

Deployment uses the **"GitHub Actions" Pages source** (build artifact deployed by `actions/deploy-pages`), not a
`gh-pages` branch as on mathieu.scheltienne.net:

- no build output in the git history, no push token, no third-party deploy action;
- atomic deployments and optional environment protection rules (e.g. only `main` may deploy);
- **no `CNAME` or `.nojekyll` file is needed**: in this mode GitHub ignores `CNAME` files; the custom domain is
  set in *Settings → Pages*, and Jekyll never runs on an uploaded artifact;
- trade-offs: the deployed files cannot be browsed as a branch (roll back by re-running an older workflow run),
  and GitHub Pages PR previews are not possible in this mode (not wanted anyway).

One-time setup: *Settings → Pages → Build and deployment → Source: GitHub Actions*.

### Custom domain and Cloudflare DNS

Status on 2026-09-25: **`gradi.net` is not owned yet** (its nameservers are `ns1/ns2.afternic.com`, a domain
marketplace / parking service, and `naima.gradi.net` resolves to parking IPs through a wildcard). The final domain
may be different. Nothing in the repository depends on the domain: `baseURL` in `hugo.toml` is only used locally,
CI builds for whatever URL GitHub Pages reports.

Once a domain is registered and its zone is on Cloudflare (example for the subdomain `naima.gradi.net`):

| Type | Name | Content | Proxy status |
| --- | --- | --- | --- |
| `CNAME` | `naima` | `<github-user>.github.io` (no repository path) | **DNS only (grey cloud)** |
| `TXT` | `_github-pages-challenge-<github-user>` (or `_github-pages-challenge-<github-user>.naima` to verify only the subdomain) | code shown by GitHub (profile/org *Settings → Pages → Add a domain*) | n/a |

1. Verify the domain on the GitHub **account or organisation that owns the repository** (prevents domain
   takeover; keep the TXT record forever). Verifying `gradi.net` also covers its immediate subdomains.
2. Do **not** create wildcard records (`*.gradi.net`): GitHub warns they enable takeovers. The parking wildcard
   must go.
3. In the repository: *Settings → Pages → Custom domain* = `naima.gradi.net`; wait for the certificate (up to
   about an hour), then tick **Enforce HTTPS**.
4. **Keep the record DNS-only.** GitHub issues and renews its Let's Encrypt certificate with an HTTP-01 challenge;
   behind the Cloudflare proxy (orange cloud) the challenge never reaches GitHub, so issuance fails ("Enforce
   HTTPS unavailable") and renewals silently fail ~90 days later. If the proxy is ever wanted: get the
   certificate with the grey cloud first, then switch to orange with SSL mode **Full (strict)**, and expect to
   flip back to grey at each renewal. Never use SSL mode *Flexible* (redirect loop).
5. If CAA records are added, include `0 issue "letsencrypt.org"`.
6. HSTS: GitHub Pages does not send HSTS headers on custom domains and custom headers are impossible; Cloudflare's
   HSTS only applies when proxied. Never enable HSTS `includeSubDomains`/preload on the parent domain unless every
   subdomain serves HTTPS. *Enforce HTTPS* (301 redirect) is the practical baseline.
7. Analytics: **Cloudflare Web Analytics** is cookieless; with a DNS-only record it needs the manual JavaScript
   beacon (Cloudflare's automatic injection only works when proxied).

Alternative host considered and not chosen: **Cloudflare Workers static assets** (Cloudflare now steers new
projects from Pages to Workers). Free unlimited static requests, `_headers`/`_redirects` support (HSTS, CSP),
Cloudflare-managed certificates (no renewal issue), and per-branch preview URLs through Workers Builds; the cost is
a Cloudflare account/token as an extra dependency. Easy to switch to later: the build output is the same.

### Moving the repository to Naïma's account

The repository will move from `mscheltienne` to Naïma's own GitHub account. After the transfer: GitHub redirects
the old URL; re-enable Pages if needed; **verify the custom domain again from her account** (new TXT record
`_github-pages-challenge-<her-user>`) and point the `CNAME` record to `<her-user>.github.io`; without a custom
domain the site URL becomes `https://<her-user>.github.io/naima.gradi.net/`. Nothing in the repository needs to
change (the workflow and the configuration are account-agnostic).

## Background: how we got here

This section summarises the research and the discussion (2026-09-25 / 26) that led to these proposals. Figures
(versions, stars, dates) are as of 2026-09-25.

### Context and requirements

- A CV/portfolio website for **Naïma Gradi**, an academic with a background in psychology and mental health who
  has just completed her PhD in the Bavelier lab (University of Geneva). She is not a developer.
- Model: [mathieu.scheltienne.net](https://mathieu.scheltienne.net) (Hugo + the hugo-coder theme, GitHub Pages,
  Cloudflare DNS).
- Content in a **simple format (Markdown/YAML)**; **strong theming and customisation**; **high maintenance
  quality** of the generator and the theme.
- **Professional look with a green pastel palette.**
- **English and French**, English by default.
- Content is edited **through git** (no browser CMS wanted).
- Hosted on **GitHub Pages**, domain on **Cloudflare**; target address `naima.gradi.net` (domain not owned yet,
  may change); the repository will later move to her own GitHub account.

### Static site generators compared

| Generator | Health (Sept. 2026) | FR/EN support | Academic themes | Verdict |
| --- | --- | --- | --- | --- |
| **Hugo** | v0.166.0 (2026-09-09), monthly releases, ~90k★, single Go binary, fastest builds | Best native support among realistic options: `about.fr.md` suffixes or per-language directories, `translationKey`, `i18n/fr.yaml` strings with fallback, `/fr/` URLs, localised dates, sitemap with language alternates; since v0.153 one content tree can serve several languages | Many (Hugo Blox, Blowfish, …) | **Chosen** |
| Astro | 7.0 (2026-06-22, Rust compiler, Vite 8), 7.3.5 latest, ~63k★; Cloudflare acquired the company on 2026-01-16 (stays MIT) | i18n routing native, but translated strings, translation linking and hreflang are DIY; static fallback had bugs | Forkable templates (as-folio, academicpages-astro, astro_academia), none with a language switcher | Runner-up: most flexible, but more code to own and yearly major upgrades |
| Jekyll (+ al-folio) | Core: last release v4.4.1 (2025-01); the native GitHub Pages build is stuck on Jekyll 3.10.0 | Only through the Polyglot plugin (Actions build required) and the community fork multi-language-al-folio | al-folio (16k★, Tailwind v4 since v1.0, jekyll-scholar BibTeX) is the de-facto academic template | Stagnant core, Ruby toolchain friction on Windows |
| Zola | v0.23 (2026-08) switched to Tera 2 (breaking; many themes broke); new code is EUPL-1.2 | Good native support, no fallback for untranslated pages | Few, no BibTeX | No |
| Eleventy | v3.1.6 stable; Font Awesome's "Build Awesome" rename (2026-03) caused backlash and uncertainty | Basic plugin, no string translation, no fallback | None mature | No |
| Quarto | v1.10 stable (Posit); Quarto 2 (Rust rewrite) announced for late 2026 | No native multilingual sites (needs the R package babelquarto) | Few; Bootswatch themes only | Best citation support, but only if she works in R/Quarto |
| MkDocs Material / Zensical | Material in maintenance mode (critical fixes until 2027-05-05); Zensical is alpha | — | Documentation-oriented | No |
| Docusaurus | v3.10 (Meta), React | Excellent, but docs-oriented | Documentation/blog | No |

Main Hugo risk: it is still pre-1.0 and ships breaking changes (see [Continuous integration](#continuous-integration)).

### Themes surveyed

| Theme | Stars / activity | Palette | FR/EN | CV & publications |
| --- | --- | --- | --- | --- |
| **Hugo Blox "Academic CV"** (kit ~9.7k★, template ~5.1k★, MIT) | Very active; freemium (paid "Pro" templates and AI tier); frequent breaking migrations (v0.11: `blox-tailwind` renamed `blox`, repo renamed `kit`; v0.12: Hugo ≥ 0.158, front-matter renames) | 11 theme packs (Matcha is the only green one and fails contrast); any hex primary; custom packs in YAML | French strings, automatic switcher, per-language author data | **Built in**: biography, education, experience, awards, skills, languages, publications (free BibTeX import via the `academic` CLI), talks |
| **Blowfish** (~2.9k★, v3.8.0 on 2026-09-24, MIT) | Very active; v3 was backward compatible; recent accessibility work | 17 schemes (`forest`, `avocado` are green but not pastel); custom scheme = one CSS file of RGB scales | French strings, header switcher, optional language redirect | **None**: to be built (done in the Blowfish branches) |
| Congo (Blowfish's parent, 1.7k★) | Active | 7 schemes | French, switcher | None |
| hugo-coder (3.1k★) | Active | Light/dark only | French, switcher | None — **excluded** (already used for mathieu.scheltienne.net) |
| PaperMod (13.9k★), hugo-profile (1.1k★), Hextra (2.4k★) | Blog / portfolio / docs themes | CSS variables | French | None |
| Jekyll: al-folio, academicpages, minimal-mistakes | Active | academicpages/minimal-mistakes `mint` skins are teal and fail AA | **No multilingual support** (except the al-folio fork) | Rich |
| Astro academic templates (as-folio, astro_academia, myscholar) | Small (20–70★) | DaisyUI / Tailwind | **No language switcher** | Some with BibTeX |

Shortlist after discussion: **Hugo Blox Academic CV** and **Blowfish**, both considered viable (upgrade churn for
the former and custom work for the latter are both acceptable). Blowfish's plain first demo led to two richer
Blowfish variants, hence four proposals.

### Tooling research: what was adopted and why

- **GitHub Actions Pages source** instead of a `gh-pages` branch (no CNAME/`.nojekyll`, atomic, no token).
- **Hugo `latest` in CI + loose `min` in the config**, exactly like mathieu.scheltienne.net (the official Hugo
  workflow pins exact versions; that needs Renovate or manual bumps).
- **Themes as git submodules** (like mathieu.scheltienne.net) rather than Hugo modules: no Go toolchain needed
  locally or in CI, and Dependabot's `gitsubmodule` ecosystem keeps them current. Hugo Blox is officially
  distributed as Hugo modules; the submodule approach is explained in its design section.
- **Dependabot** (it now supports the `pre-commit` ecosystem, since March 2026) rather than Renovate.
- **Biome** for CSS/JS/JSON with `layouts/` excluded (Biome now parses HTML and would mangle Go templates).
- **markdownlint-cli2** for Markdown (rumdl, a fast Rust alternative with Hugo support, is a valid option too).
- **lychee** offline link check on pull requests.
- Cloudflare: DNS-only records for GitHub Pages (see [Custom domain](#custom-domain-and-cloudflare-dns)).

## Decision log

| Date | Decision |
| --- | --- |
| 2026-09-25 | No browser CMS; content edited through git. |
| 2026-09-25 | Multilingual support wanted (French/English); English is the default language. |
| 2026-09-25 | Generator: **Hugo**. Theme: **not hugo-coder**; shortlist Blowfish and Hugo Blox Academic CV, to be decided after seeing demos. |
| 2026-09-25 | Keep tooling lean: no cspell, no missing-translation warnings, no Renovate; ORCID linking wanted. |
| 2026-09-25 | Same Hugo version mechanism as mathieu.scheltienne.net: `latest` in CI and a loose `min`. |
| 2026-09-25 | URLs: `/en/` and `/fr/`, with `/` redirecting to `/en/` (English also at `/en/` for consistency). |
| 2026-09-25 | Palettes: choose after seeing examples (three palettes A/B/C proposed). |
| 2026-09-25 | Domain: `gradi.net` not owned yet; the final domain may differ. The repository will move to Naïma's account later. |
| 2026-09-26 | Blowfish demo judged too plain → two richer Blowfish variants (scholar, landing) built. |
| 2026-09-26 | All four proposals are solid; the choice waits for real content. Each becomes a `design/*` branch with the full repository structure, sharing one layout; the palette picker is kept for now. |
| 2026-10-22 | Shortlist narrowed to `blowfish-basic` and `hugoblox` (`blowfish-scholar` and `blowfish-landing` dropped). Palette **B · Mint & Pine** approved. `design/blowfish-hybrid` combines both: profile column on the home page only, system sans-serif fonts, education tiles on the home and CV pages. |

## Open questions and to-do before launch

- [ ] **Choose the design** (branch), the palette and the fonts — once real content exists.
- [ ] **PR previews**: assumed *not wanted* (the message said "PR previews are now wanted. Renovate is overkill as
  well", read as a typo for "not"); to be confirmed. If wanted: Cloudflare Workers preview uploads are the
  cleanest option with the Pages Actions source.
- [ ] **Domain**: register a domain (`gradi.net` or another), move its DNS to Cloudflare, add the records above.
- [ ] **Repository transfer** to Naïma's account; enable Pages there.
- [ ] Real content: bio (EN/FR), CV, publications, talks, photo, CV PDF(s); real ORCID/Scholar/LinkedIn/e-mail.
- [ ] Self-host the web fonts; remove the demo palette picker.
- [ ] Decide on a licence for the site content (none is set; mathieu.scheltienne.net uses MIT for code).
- [ ] Optional: Cloudflare Web Analytics beacon; Google Scholar `citation_*` meta tags; ORCID-driven publication
  list; e-mail obfuscation (Blowfish already base64-encodes the e-mail link in the profile icons).

## Side notes on mathieu.scheltienne.net

Two findings about the existing site, noted during the research:

- `mathieu.scheltienne.net` is served **through the Cloudflare proxy** (`Server: cloudflare` with GitHub headers):
  GitHub's Let's Encrypt renewals can silently fail behind the proxy (see above). Consider switching the record to
  DNS-only.
- hugo-coder compiles its SCSS with `toCSS $cssOpts`; if no transpiler is set it falls back to **LibSass**, which
  is deprecated since Hugo v0.153 and only exists in the extended edition. Worth checking whether `$cssOpts` sets
  `transpiler: dartsass` (override `layouts/_partials/head/theme-styles.html` if not).
