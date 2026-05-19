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

1. Create a new repo on GitHub (e.g. `andyp-portfolio`).
2. Push this project:

   ```bash
   git remote remove origin   # if still pointing at the template repo
   git remote add origin git@github.com:YOUR_USERNAME/andyp-portfolio.git
   git push -u origin main
   ```

3. In the repo: **Settings → Pages → Build and deployment → Source: GitHub Actions**.
4. Edit `site/_config.yml`:

   ```yaml
   url: "https://YOUR_GITHUB_USERNAME.github.io"
   baseurl: "/andyp-portfolio"   # repo name; use "" for username.github.io repo
   ```

5. Push to `main`. The workflow in `.github/workflows/pages.yml` builds and deploys the site.

Your site will be at `https://YOUR_GITHUB_USERNAME.github.io/andyp-portfolio/`.

## What to personalize

| What | Where |
|------|--------|
| Name & avatar | `site/_data/author.yml` |
| Site title & meta description | `site/_data/general_settings.yml` |
| GitHub & LinkedIn links | `site/_data/social_links.yml` |
| Menu | `site/_data/navigation.yml` |
| Home hero & sections | `site/collections/_pages/index.html` |
| About page | `site/collections/_pages/about.html` |
| Profile photo | Replace `site/images/avatar.jpg`, `site/images/01.jpg` |
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
