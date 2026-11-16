# DataForSEO API Key Setup

> Customer install steps for this copy are in [INSTALL.md](./INSTALL.md). Transformaiton.com is not affiliated with DataForSEO or with the original OpenSEO authors.

OpenSEO uses [DataForSEO](https://dataforseo.com/) to fetch SEO data. DataForSEO is a separate company. You pay DataForSEO for the lookups. You need an API key to connect OpenSEO to it.

New DataForSEO accounts include $1 of free credit to test with, and the minimum top-up is $50.

## Get your API key

1. Go to [DataForSEO API Access](https://app.dataforseo.com/api-access) (create an account if you don't have one).
2. Copy the API credential labelled "Base64" from the API Access page. This is the base64 encoded value of your DataForSEO email and API password in the format `email:password`.

## Where to set it

Set the value as `DATAFORSEO_API_KEY`:

- **Docker self-hosting:** in `deploy/standalone/.env` (see [`SELF_HOSTING_DOCKER.md`](./SELF_HOSTING_DOCKER.md)), or `.env` for the [older single-container image](./SELF_HOSTING_DOCKER_LEGACY.md).
- **Cloudflare self-hosting:** in `.env.selfhost` (see [`SELF_HOSTING_CLOUDFLARE.md`](./SELF_HOSTING_CLOUDFLARE.md)). Legacy button/Wrangler deployments: as a Worker secret in the dashboard under `Settings` -> `Variables & Secrets`.
- **Local development:** in `.env.local` (see [`LOCAL_DEVELOPMENT.md`](./LOCAL_DEVELOPMENT.md)).
