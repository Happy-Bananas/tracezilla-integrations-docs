# BifrostConnect Documentation

Service-first documentation for consultants integrating Shopify,
WooCommerce, and other external services with the tracezilla API.

## Run locally

Requirements: Docker with Docker Compose.

```bash
docker compose up --build
```

Keep the terminal open while Docker starts the site. The first build may take
a little while. The site is ready when the terminal displays:

```text
============================================================
  DOCUMENTATION IS READY
  Open http://localhost:4000/ in your browser
============================================================
```

Then open <http://localhost:4000/>. Messages beginning with `DEPRECATION
WARNING` are build warnings and do not prevent the site from starting.

Docker Compose loads `_config.local.yml` after the production configuration,
so the local site is served directly from `/`. The GitHub workflow uses only
`_config.yml` and continues to publish at `/tracezilla-integrations-docs/`.

The source directory is mounted into the container. Markdown changes are
rebuilt automatically. Restart the container after changing `_config.yml`.

Stop the site with:

```bash
docker compose down
```

## Verify

Build the production site and validate its internal links, anchors, images,
stylesheets, and scripts:

```bash
docker compose run --rm docs sh -c \
  'JEKYLL_ENV=production bundle exec jekyll build \
     --destination /tmp/tracezilla-integrations-docs-site && \
   bundle exec htmlproofer /tmp/tracezilla-integrations-docs-site \
   --disable-external \
   --no-enforce-https \
   --swap-urls "^/tracezilla-integrations-docs/:/"'
```

HTMLProofer checks generated local content without contacting external sites.
A separate temporary destination prevents a production build from overwriting
the files served at `http://localhost:4000/`.
A broken internal destination blocks the GitHub Pages deployment. External
links are excluded because rate limits and temporary third-party outages should
not block publication.

## Publishing

Pushes to `main` run the GitHub Pages workflow. In the GitHub repository,
select **Settings → Pages → Source → GitHub Actions** once before the first
deployment.

## License

This project is available under the [MIT License](./LICENSE).
