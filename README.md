# tracezilla Integrations Documentation

Service-first documentation for consultants integrating Shopify,
WooCommerce, and other external services with the tracezilla API.

## Run locally

Requirements: Docker with Docker Compose.

```bash
docker compose up --build
```

Open <http://localhost:4000/>.

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

Build the same static site that GitHub Pages receives:

```bash
docker compose run --rm docs bundle exec jekyll build
```

## Publishing

Pushes to `main` run the GitHub Pages workflow. In the GitHub repository,
select **Settings → Pages → Source → GitHub Actions** once before the first
deployment.

## License

This project is available under the [MIT License](./LICENSE).
