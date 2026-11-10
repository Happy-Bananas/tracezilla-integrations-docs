# BifrostConnect Documentation

Service-first documentation for developers integrating Shopify,
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

## The create-project script

`create-project` is published unchanged at
`https://happy-bananas.github.io/tracezilla-integrations-docs/create-project`.
Getting Started uses it to create a new project for each platform. Its tests
use local archives and the real templates, and run before every deployment:

```bash
sh tools/test-create-project
```

New projects are created from a released version of each template. To
release a template:

1. Add the release to the template's `CHANGELOG.md` and merge it.
2. Tag `main` in the template repository, for example `git tag -a v0.2.0` and
   `git push origin v0.2.0`.
3. Set `version=v0.2.0` for that platform in `create-project`.

To add a platform:

1. Add it, with its released `version`, to the `case "$platform"` list and the
   usage text in `create-project`.
2. Add a row to the platform table in `getting-started.md` (step 4).
3. Add the template to the real-template tests in `tools/test-create-project`
   and to the clone step in `.github/workflows/pages.yml`.

## Publishing

Pushes to `main` run the GitHub Pages workflow. In the GitHub repository,
select **Settings → Pages → Source → GitHub Actions** once before the first
deployment.

## License

This project is available under the [MIT License](./LICENSE).
