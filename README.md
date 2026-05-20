# Portfolio site (Vonge + Jekyll)

A personal portfolio based on the [Vonge](https://cloudcannon.com/templates/vonge/?ssg=jekyll) Jekyll template. Use it for LinkedIn, your CV, and GitHub Pages.

- **Writing** — blog-style write-ups with tags (AI, ML, Python, …)
- **Projects** — showcase GitHub repos with tags and a filter bar
- **Tag filters** — topic pills on Writing and Projects pages

## Run locally

```bash
npm install
npm run jekyll:install
npm start
```

Open [http://localhost:6060](http://localhost:6060).

## Publish to GitHub Pages

This site uses **Bookshop** (`{% bookshop_scss %}`, `jekyll-bookshop`). GitHub’s built-in Jekyll builder does **not** include those plugins. You must deploy with the **GitHub Actions** workflow in `.github/workflows/pages.yml`, not “Deploy from a branch”.

1. Create a repo (for a user site, name it `YOUR_USERNAME.github.io`).
2. Push this project to `main`.
3. **Required:** In the repo go to **Settings → Pages → Build and deployment** and set **Source** to **GitHub Actions** (not “Deploy from a branch”). Choose the **Deploy GitHub Pages** workflow if prompted.
4. Edit `site/_config.yml`:

   ```yaml
   # User site (repo named YOUR_USERNAME.github.io):
   url: "https://YOUR_USERNAME.github.io"
   baseurl: ""

   # Project site (repo named something else):
   url: "https://YOUR_USERNAME.github.io"
   baseurl: "/your-repo-name"
   ```

5. Push to `main`, or run **Actions → Deploy GitHub Pages → Run workflow**.

- User site URL: `https://YOUR_USERNAME.github.io/`
- Project site URL: `https://YOUR_USERNAME.github.io/your-repo-name/`

### Temporarily take the site offline

While `offline/ENABLED` exists, the deploy workflow publishes a simple “Back soon” page instead of the full site.

```bash
# Take down
touch offline/ENABLED && git add offline/ENABLED && git commit -m "Enable maintenance mode" && git push

# Bring back
git rm offline/ENABLED && git commit -m "Disable maintenance mode" && git push
```

### Pages build failed: `Unknown tag 'bookshop_scss'`

That log means GitHub ran its **default** Pages Jekyll build (`github-pages` gem, Jekyll 3.10, `jekyll-theme-primer`) on the raw repo. Switch **Pages → Source** to **GitHub Actions** as above, then re-run **Deploy GitHub Pages**. The Actions workflow uses Jekyll 4.3 + Bookshop and publishes the built output from `site/_site`.

## What to personalize

| What | Where |
|------|--------|
| Name & avatar | `site/_data/author.yml` |
| Site title & meta description | `site/_data/general_settings.yml` |
| GitHub & LinkedIn links | `site/_data/social_links.yml` |
| Menu | `site/_data/navigation.yml` |
| Home hero & sections | `site/collections/_pages/index.html` |
| About page | `site/collections/_pages/about.html` |
| Profile photo | `site/images/avatar.png` (hero, author byline, OG share image) |
| GitHub Pages URL | `site/_config.yml` (`url`, `baseurl`) |
| Placeholder GitHub URLs | Example projects in `site/collections/_projects/` |

### Add a write-up

Create `site/collections/_posts/YYYY-MM-DD-my-article.markdown`:

```yaml
---
date: 2025-05-19
title: My article title
description: Short summary for cards and SEO.
tags:
  - ai
  - ml
image: /images/post-1.jpg
---

Your content here (Markdown).
```

Tags power the filter bar on **Writing** and create pages like `/tag/ai`.

### Add a GitHub project

Create `site/collections/_projects/YYYY-MM-DD-repo-name.md`:

```yaml
---
date: 2025-05-19
title: My repo name
subtitle: Python · FastAPI
image: '/images/project-1.jpg'
tags:
  - ai
  - python
github_url: https://github.com/YOUR_USERNAME/my-repo
---

Short description of the project.
```

Tags power the filter bar on **Projects** (client-side filter).

### Images

Add images under `site/images/`. Reference them in front matter as `/images/your-file.jpg`.

### Optional removals

- **Newsletter** — remove `newsletter` blocks from page front matter in `site/collections/_pages/`.
- **Testimonials** — delete `site/collections/_testimonials/` or keep `show_testimonials: false` on the home page.
- **Contact form** — remove the `contact-form` block from `index.html` (or configure email in CloudCannon; locally it is display-only).

## Project structure

```
site/
  collections/
    _posts/       # Write-ups
    _projects/    # GitHub repos & portfolio items
    _pages/       # Home, About, Blog, Projects
  _data/          # Author, nav, social links
  images/
component-library/  # Vonge UI components (Bookshop)
```

## Credits

Template: [CloudCannon/vonge-jekyll-bookshop-template](https://github.com/CloudCannon/vonge-jekyll-bookshop-template) ([Vonge on CloudCannon](https://cloudcannon.com/templates/vonge/?ssg=jekyll)).
