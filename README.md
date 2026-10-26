# Blog

## Local preview

Install **Hugo 0.147.9** (the standard edition is sufficient) from the [Hugo
releases](https://github.com/gohugoio/hugo/releases/tag/v0.147.9). The deployment workflow uses the
same version.

```sh
hugo server -D
```

Open the URL printed by Hugo, normally <http://localhost:1313/>. `-D` includes draft articles.
Without it, the blog initially displays “No posts yet.”

Build production files with:

```sh
hugo --gc --minify --panicOnWarning
```

The generated site is in `public/`, which is `.gitignore`d.

## Write an article

Create a page bundle so figures can live with the article:

```sh
hugo new content blog/my-post/index.md
```

Edit `content/blog/my-post/index.md`:

```yaml
---
title: 'My post'
date: 2026-10-24
description: 'A short summary for metadata and feed readers.'
draft: true
---
```

Put figures in the same folder and reference them with Markdown:

```markdown
![Describe the figure for readers who cannot see it](diagram.svg)
```

For a caption, use Hugo’s built-in figure shortcode:

```text
{{< figure src="diagram.svg" alt="A description of the diagram." caption="An optional caption." >}}
```

The folder name becomes the URL slug; an explicit `slug` metadata field can override it. Set `draft:
false` to publish. Future-dated posts are excluded until a build runs after their publication date;
there is no scheduled build.

## Bundled assets and licenses

- **Source Sans 3** (upright and italic variable WOFF2) and **Source Code Pro** (upright variable
WOFF2) come from Adobe’s [source-sans](https://github.com/adobe-fonts/source-sans) and
[source-code-pro](https://github.com/adobe-fonts/source-code-pro) repositories, under `WOFF2/VF` on
the `release` branch. SIL Open Font License notices are retained in `static/fonts/`.
- **GitHub, LinkedIn, and envelope icons** come from [Bootstrap Icons
1.11.3](https://github.com/twbs/icons/tree/v1.11.3), under the MIT license retained at
`assets/icons/LICENSE`.
