# Solid Federated Catalogue Publisher

## What This Is

This tool is a standalone publisher for Solid-based DCAT catalogues. It reads an existing catalogue from a Solid Pod, previews the RDF, and publishes the original graph to a central federated catalogue.

## What This Is Not

- This is not a Solid Pod server.
- This is not a replacement for Florian Hoelken et al.'s `semantic-data-catalog`.
- This does not install code into a Pod.
- This does not replace the central federated catalogue.
- This does not implement EDC publishing yet.

## Architecture

```text
Existing Solid Pod
  catalog/cat.ttl
  catalog/ds/*.ttl
        |
        | authenticated Solid fetch
        v
Solid Publisher
        |
        | POST RDF + Solid-OIDC token
        v
Central Federated Catalogue
        |
        | registry check + SHACL validation
        v
Fuseki index
        |
        v
Discovery API/UI
```

## Relation To Florian's Semantic Data Catalog

This publisher builds on the Solid catalogue conventions and selected implementation ideas from Florian Hoelken et al.'s `semantic-data-catalog`.

Adapted ideas include Solid-OIDC session handling, catalogue path conventions, WebID conventions, DCAT modelling choices, and UI concepts. The central publishing workflow is new: it adds a participant-side push step from a Solid catalogue to a central federated catalogue.

Credit:

- Florian Hoelken et al., Semantic Data Catalog for decentralized Solid-based dataspaces
- TMDT / University of Wuppertal
- Repository: `https://github.com/tmdt-buw/semantic-data-catalog`
- License: Apache-2.0

## How It Works

1. Log in with Solid-OIDC.
2. Select a catalogue by URL, WebID-based convention, or local file.
3. Preview the RDF and DCAT counts.
4. Publish the original RDF payload to the central catalogue.
5. Inspect the central catalogue response.

## Quick Start

```bash
cp .env.example .env
npm install
npm run dev
```

Open `http://localhost:5173`.

With Docker:

```bash
docker compose up --build
```

## Configuration

| Variable | Default | Purpose |
| --- | --- | --- |
| `VITE_DEFAULT_IDP` | `https://solidcommunity.net` | Default Solid identity provider shown in the UI |
| `VITE_CENTRAL_CATALOG_URL` | `http://localhost:8000/catalog` | Default central catalogue publish endpoint |

## Running With An Existing Solid Setup

The publisher is external to the Pod. It authenticates as a WebID and fetches existing catalogue RDF. A direct catalogue URL should usually look like:

```text
https://<pod-host>/<user>/catalog/cat.ttl
```

For WebID-based resolution, the app infers:

```text
<pod-root>/catalog/cat.ttl
```

If the Pod layout differs, use the explicit catalogue URL.

## Publishing A Catalogue

Expected central request:

```http
POST /catalog
Authorization: Bearer <Solid-OIDC access token>
Content-Type: text/turtle
X-Participant-WebID: <authenticated WebID>
```

The browser uses the authenticated Solid session when available. Local file upload can preview without login, but the central catalogue may still require authentication for publishing.

## Using The CLI Publisher

```bash
python tools/publish-local-catalog/publish_local_catalog.py \
  --catalog-file data/examples/catalog-valid.ttl \
  --central-url http://localhost:8000/catalog \
  --webid https://example.org/profile/card#me \
  --token dev-token
```

The CLI exits `0` for a `2xx` central catalogue response and non-zero otherwise.

## Expected Central Catalogue Behavior

- `200`: publication accepted
- `401`: Solid login token missing, expired, or rejected
- `403`: WebID is not registered in the central catalogue's Solid registry
- `422`: RDF failed DCAT/SHACL validation
- network error: central catalogue could not be reached

## Screenshots

Place screenshots under `docs/screenshots/`. The UI is a single-page operator tool with login, source selection, RDF preview, publish controls, and result panels.

## Troubleshooting

- If login does not return a WebID, confirm the identity provider and redirect URL.
- If catalogue fetch fails, try the direct `catalog/cat.ttl` URL and confirm public/authenticated read access.
- If preview shows no datasets, confirm the RDF uses `dcat:Dataset`.
- If publishing is rejected, inspect the central catalogue response body; it should name `auth`, `registry`, `validation`, or `store`.

## Development

```bash
npm install
npm run dev
npm run test
npm run build
```

## Credits

See [CREDITS.md](CREDITS.md) and [NOTICE](NOTICE).

