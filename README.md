# Blog

Source for <https://diogoduarte.pt/blog/>, built with [Hugo](https://gohugo.io/) and deployed to GitHub Pages on every push to `main`.

## Writing a post

1. Create `content/posts/<slug>.md` (or run `hugo new posts/<slug>.md`):

   ```markdown
   ---
   title: "My post"
   date: 2026-10-22
   ---

   Opening paragraph(s), shown as the preview on the front page.

   <!--more-->

   The rest of the post.
   ```

2. Write normal Markdown. Math uses LaTeX: `$...$` inline, `$$...$$` on its own lines for display.
   It is rendered to HTML with KaTeX at build time, so a LaTeX typo fails the build instead of showing up broken on the site.
3. Everything above `<!--more-->` is the preview on the front page; clicking the title or "Read more" opens the full post.
4. Add `draft: true` to the front matter to keep a post off the live site.

## Previewing locally

```sh
hugo server -D      # -D also shows drafts
```

Then open <http://localhost:1313/blog/>.

## Layout

- `hugo.toml`: site settings (title, URL, Markdown/math options)
- `content/posts/`: the posts
- `layouts/`: page templates (`home.html` front page, `single.html` post page, `_markup/render-passthrough.html` math)
- `assets/style.css`: all styling
- `static/katex/`: KaTeX stylesheet and fonts (v0.18.4, matching the KaTeX bundled in Hugo)
- `.github/workflows/deploy.yml`: builds and deploys to GitHub Pages
