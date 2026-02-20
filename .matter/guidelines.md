# OpenRA Website — Project Guidelines

## Project Overview

The OpenRA Website is the official web presence for the [OpenRA](https://openra.net) open-source project, which recreates and modernizes classic real-time strategy games (Red Alert, Command & Conquer, Dune 2000). The site provides news updates, game information, download links, a server browser, player statistics, and community resources.

## Architecture

### Technology Stack

- **Static Site Generator:** [Jekyll](https://jekyllrb.com/) 3.9.0 via the `github-pages` gem
- **Languages:** HTML, Liquid templates, Markdown (for blog posts), SCSS (Sass), JavaScript (ES6, jQuery-based)
- **CSS:** Custom SCSS compiled by Jekyll's `jekyll-sass-converter`. No CSS framework; entirely custom styles.
- **JavaScript:** jQuery 3.5.1 as the primary DOM manipulation library. No JS build step.
- **Hosting:** GitHub Pages (static output deployed to `openra/openra.github.io`)

### Single Service

This is a single-service application — a Jekyll static site. In development, `bundle exec jekyll serve` runs a local web server with live-reload on port 4000.

## Key Configuration Files

| File | Purpose |
|---|---|
| `_config.yml` | Main Jekyll config: site URL, plugin list, `download_page_tags` (release/playtest versions), permalink structure, defaults |
| `Gemfile` / `Gemfile.lock` | Ruby dependencies — `github-pages` gem bundles Jekyll + plugins |
| `styles/index.scss` | Main SCSS entry point — imports all partials from `_sass/` |
| `icomoon.json` | Icon set configuration for the Icomoon SVG sprite generator |

## Important Environment Variables

- `JEKYLL_ENV` — Set to `development` or `production`
- `JEKYLL_GITHUB_TOKEN` — Required for `jekyll-github-metadata` plugin to query the GitHub API (for download page release data). Without it, the download page will show empty release information but the site will still build.

## Project Structure

```
/
├── _config.yml           # Jekyll configuration
├── _layouts/             # Page layouts (default.html, news.html)
├── _includes/            # Reusable HTML partials (head, header, footer, foot, developer-update, etc.)
├── _sass/                # SCSS source files
│   ├── variables.scss    # Color, font, spacing variables
│   ├── mixins.scss       # Reusable SCSS mixins (metalPanel, clearfix, etc.)
│   ├── extensions.scss   # SCSS placeholder selectors (%gold, %scanlines, %selectionBox, etc.)
│   ├── html.scss         # Base element styles
│   ├── buttons.scss      # Button component styles
│   ├── site.scss         # Site-level layout (header, footer, sections)
│   ├── utility.scss      # Utility classes (.text--, .color--, .u-dark-zone, etc.)
│   ├── components/       # Reusable component styles (dark-panel, minimap)
│   ├── includes/         # Styles for _includes partials
│   └── pages/            # Page-specific styles
├── styles/               # CSS entry points (index.scss, normalize.css, lite-youtube-embed.css)
├── scripts/              # JavaScript files
│   ├── global.js         # Carousel component + mobile menu toggle (loaded on every page)
│   ├── index.js          # Homepage-specific JS
│   ├── download.js       # Download page JS
│   ├── server-browser.js # Server browser jQuery "class"
│   ├── players.js        # Player stats page JS
│   └── vendor/           # Third-party JS (jQuery, Flot charts, svgxuse, popper, etc.)
├── news/_posts/          # Markdown news posts with YAML front matter
├── index.html            # Homepage
├── about.html            # About page
├── download.html         # Download page
├── games.html            # Games (server browser) page
├── community.html        # Community page
├── players.html          # Player statistics page
├── 404.html              # Custom 404 page
├── images/               # All image assets
├── fonts/                # Web fonts (Inter, Jost)
└── videos/               # Sizzle reel videos for homepage carousel
```

## Layouts

There are two layouts:

1. **`default`** — Used by most pages. Structure: `<head>` + `<header>` + `<div class="site-content">{{ content }}</div>` + `<footer>` + scripts
2. **`news`** — For individual news posts. Wraps content in a `dark-panel` component with a "View All Updates" back link and the `developer-update` include.

## Key Design Patterns

### Page Structure

Every page follows this pattern:
```html
---
title: "Page Title"
permalink: "/page-slug/"
js:
- /scripts/page-specific.js  <!-- optional -->
---

<section class="site-section">
  <div class="site-section__content">
    <h1 class="site-section__heading">Heading</h1>
    <!-- page content -->
  </div>
</section>

<hr class="site-section__divider" />

<section class="site-section">
  <div class="site-section__content">
    <!-- more content -->
  </div>
</section>
```

- Pages are composed of `<section class="site-section">` blocks separated by `<hr class="site-section__divider" />`
- Content inside sections is wrapped in `<div class="site-section__content">` (max-width: 1025px, centered)
- Section headings use `<h1 class="site-section__heading">` (centered, gold gradient text)

### BEM-like CSS Naming

The project uses a BEM-inspired naming convention:
- **Block:** `.dark-panel`, `.developer-update`, `.site-header`
- **Element:** `.dark-panel__header`, `.site-header__nav__link`
- **Modifier:** `.carousel__item--current`, `.button--primary`, `.button--large`
- **State:** `.is-active`, `.site--menu-active`

### Dark Panel Component

The `dark-panel` is the primary content container for structured data:
```html
<div class="dark-panel">
  <div class="dark-panel__header">
    <strong class="text--caps text--medium">Header Text</strong>
  </div>
  <!-- panel content -->
</div>
```
It uses the `metalPanel()` mixin — a beveled, semi-transparent dark card with grid-line background.

### SVG Icons

Icons use an SVG sprite loaded from `/images/icons/icons.svg`:
```html
<svg class="icon">
  <use xlink:href="{{ '/images/icons/icons.svg#icon-name' | relative_url }}"></use>
</svg>
```

### Buttons

Buttons use the `.button` base class with modifiers:
- `.button--primary` — Red background (for main CTAs)
- `.button--large` — Larger padding/font
- `.button--light` — Light background
- `.button--construction` — Yellow with diagonal stripe pattern (for playtest/beta)
- Platform-colored: `.button--discord`, `.button--reddit`, `.button--youtube`, etc.
- `.button--disabled` — Grayed out, no pointer events

With download icon:
```html
<a class="button button--large button--primary" href="...">
  Download
  <svg class="button__icon icon">
    <use xlink:href="{{ '/images/icons/icons.svg#icon-download' | relative_url }}"></use>
  </svg>
</a>
```

### Utility Classes

- **Text sizes:** `.text--tiny`, `.text--small`, `.text--medium`, `.text--large`
- **Text styles:** `.text--caps` (Jost font, uppercase), `.text--small-caps`, `.text--hidden` (screen-reader only), `.text--centered`
- **Colors:** `.color--gold`, `.color--discord`, `.color--ra`, `.color--cnc`, `.color--d2k`, `.color--ts`
- **Layout:** `.u-centered-content`, `.u-dark-zone` (metalPanel), `.u-light-zone` (scanlines), `.u-two-column`, `.u-float-right`, `.u-padding`
- **Lists:** `.list--no-style`, `.list--horizontal`

### SCSS Variables

**Colors:**
- Game colors: `$color-red-alert`, `$color-command-and-conquer`, `$color-dune-2000`, `$color-tiberian-sun`
- Social: `$color-reddit`, `$color-discord`, `$color-facebook`, `$color-twitter`, `$color-youtube`
- UI: `$color-off-black`, `$color-off-white`, `$color-warn`, `$color-info`

**Fonts:**
- Heading font: `$font-jost` — `'Jost', 'Futura', sans-serif`
- Body font: `$font-inter` — `'Inter', -apple-system, ...`
- Sizes: `$font-size-tiny` (0.75rem) to `$font-size-xlarge` (1.25rem)

**Breakpoints:** 680px (mobile), 1000px (tablet) — used inline in `@media` queries (no breakpoint variables)

### SCSS Mixins & Placeholders

- `metalPanel($width)` — Beveled dark card with grid-line background
- `clearfix` — Classic clearfix
- `ellipsisOverflow` — Text truncation
- `%gold` — Gradient text effect for headings
- `%scanlines` — Light gray scanline pattern background
- `%selectionBox` — Animated selection border (RTS unit selection style)
- `%spawnpoint` — Small circular badge (used in server browser maps)

### News Posts

News posts are Markdown files in `news/_posts/` with required front matter:
```yaml
---
kind: article
title: "Post Title"
author: "Author Name"
created_at: 2025-03-30 12:00 +0000
disqus_id: "unique-disqus-id"
permalink: "/news/post-slug/"
---
```

Posts support:
- Standard Markdown
- Inline HTML (figures, images)
- `<lite-youtube videoid="VIDEO_ID"></lite-youtube>` for YouTube embeds
- Image paths use `{{ '/images/news/filename.png' | relative_url }}`

### JavaScript

- Per-page JS is specified in front matter (`js:` array) and loaded by `_includes/foot.html`
- `global.js` defines the `Carousel` constructor (jQuery-based) and mobile menu toggle — available on all pages
- No JS build pipeline; scripts are plain ES6 served directly
- jQuery is the DOM manipulation library (`$` is globally available)

## Build & Run

**Install dependencies:**
```bash
bundle install
```

**Run development server:**
```bash
bundle exec jekyll serve --host 0.0.0.0 --port 4000
```

The site will be available at `http://localhost:4000`. Jekyll watches for file changes and rebuilds automatically.

**Note:** The `jekyll-github-metadata` plugin queries the GitHub API for release data used on the download page. Without a `JEKYLL_GITHUB_TOKEN`, this data will be empty but the site will still build and serve correctly.