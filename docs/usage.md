# Usage

## Browser UI

```bash
cp .env.example .env
npm install
npm run dev
```

Open `http://localhost:5173`.

1. Log in with a Solid identity provider.
2. Enter a direct `catalog/cat.ttl` URL or infer one from a WebID.
3. Preview the DCAT summary and warnings.
4. Set the central catalogue URL.
5. Publish.

## Local File Demo

Local file upload lets you preview RDF without Solid login. Publishing may still fail if the central catalogue requires a real Solid-OIDC token.

## CLI

```bash
python tools/publish-local-catalog/publish_local_catalog.py \
  --catalog-file data/examples/catalog-valid.ttl \
  --central-url http://localhost:8000/catalog \
  --webid https://example.org/profile/card#me \
  --token dev-token
```

