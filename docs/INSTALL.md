# Install Open SEO

This is the install guide for [Open SEO by Transformaiton.com](../README.md). It is written for a small-business owner who wants the app on a computer they control, or on their own Cloudflare account.

The application code is the upstream project, [every-app/open-seo](https://github.com/every-app/open-seo). Transformaiton.com is not affiliated with, and is not endorsed by, the original authors. The license is unchanged. See [Credits](../README.md#credits).

Two ways to run it:

- **Docker.** A prebuilt image on your computer or a server. This is the path to use for one website, and for site audits larger than a few dozen pages. The image is `ghcr.io/every-app/open-seo-standalone`, published by the original project. The commands below pull that image. They do not build a Transformaiton.com image.
- **Cloudflare.** No server to keep running. One deploy command creates the database and the workers in your Cloudflare account. On Cloudflare's free Workers plan, a site audit stops at about 40 pages.

Both ways use email and password accounts. The first person to sign up becomes the administrator. After that, the public sign-up page closes. Teammates join from a link you copy out of Team settings. The app does not email that link.

## Before you start

1. A [DataForSEO](https://dataforseo.com/) account. Open [API Access](https://app.dataforseo.com/api-access) and copy the credential labeled **Base64**. That value is your email and API password, already encoded. Paste it as `DATAFORSEO_API_KEY`. Do not paste the password by itself.
2. New DataForSEO accounts include $1 of credit. The smallest top-up after that is $50. You pay DataForSEO, not Transformaiton.com. What a lookup costs is in the [README](../README.md#what-you-need-and-what-it-costs).
3. A computer for Docker, with Docker Desktop or Docker Engine plus Compose, and at least 4 GB of memory given to Docker. Or a Cloudflare account, if you use that path.

Optional, and safe to skip on day one:

- `OPENROUTER_API_KEY`, if you want the in-app assistant. Research screens work without it.
- Google OAuth client id and secret, if you want Search Console or Analytics. Guides: [Search Console](./SELF_HOSTING_GOOGLE_SEARCH_CONSOLE.md), [Analytics](./SELF_HOSTING_GOOGLE_ANALYTICS.md).
- `CONTEXT_API_KEY`, if a Docker audit must run JavaScript. A normal HTML crawl does not need it.
- `GOOGLE_MAPS_API_KEY`, only for the map on the Local SEO screen. See [Google Maps setup](./GOOGLE_MAPS_SETUP.md).

## Docker

Run every command from the repository folder after the clone.

### 1. Install Docker

Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) on a Mac or Windows PC, or Docker Engine and the Compose plugin on Linux. Open Docker's settings and set memory to at least 4 GB. The stack usually sits at 1 to 2 GB once it is up.

### 2. Get this repository

```sh
git clone https://github.com/Transformaiton/open-seo.git
cd open-seo
export COMPOSE_FILE=deploy/standalone/compose.yaml
cp deploy/standalone/.env.example deploy/standalone/.env
```

`export COMPOSE_FILE=...` applies to that terminal session. Put the same line in your shell profile if you will come back in a new window, or pass `--file deploy/standalone/compose.yaml` on each `docker compose` command.

### 3. Set the two required values

Open `deploy/standalone/.env`.

- `STORAGE_SECRET_KEY`: replace the placeholder with output from `openssl rand -hex 32`. Pick this once, before the first start, and keep it. It is part of how stored data is protected.
- `DATAFORSEO_API_KEY`: the Base64 credential from DataForSEO.
- `BETTER_AUTH_URL`: leave `http://localhost:3001` if you will open the app on this computer. If people will open it at a real address, set this to that `https://` URL before the first start.
- `AUTH_MODE`: leave `password`.

Do not commit `.env`. It is a local file.

### 4. Start

```sh
docker compose pull
docker compose up -d
docker compose logs -f init
```

Wait until a line says `OpenSEO initialized and ready.` The first start downloads the image. Later starts are quicker. `init` exits on purpose after each start. That is not a crash.

### 5. Create the admin account

Open <http://localhost:3001>. Create the administrator account immediately. Until that account exists, anyone who can open the URL can claim the installation.

The app then asks "What's your website?" Enter your domain, such as `example.com`. Saving it starts a 50-page audit in the background. Lighthouse is off for that first audit. JavaScript rendering runs only when a renderer is configured, so a Docker install without `CONTEXT_API_KEY` crawls the HTML. You can choose **Skip for now** and add the site later.

The country defaults to the United States. Open "Choose default country & language" on that screen if you sell somewhere else.

### Put it on the internet

Keep the app on localhost until the admin account exists. Then set `BETTER_AUTH_URL` to the public `https://` address and run:

```sh
docker compose up -d --force-recreate init web crawl-gateway
```

Put an HTTPS reverse proxy in front of `127.0.0.1:3001`. Forward WebSocket upgrades if you use the in-app assistant. Plain HTTP is accepted only for localhost. The published port is Nginx, on the machine's loopback address. The database is not published to the network.

Coolify, Portainer, and Dokploy can run `deploy/standalone/compose.yaml` on its own. Point the platform's domain at the `web` service on port 80, and set `BETTER_AUTH_URL` to that `https://` address. The longer notes are in [Docker self-hosting](./SELF_HOSTING_DOCKER.md).

### Accounts, updates, backups

Invite a teammate from Team settings. Copy the link and send it yourself.

Password resets do not send email. From the repository, with the stack running:

```sh
read -rs -p 'New password: ' password; echo
printf '%s\n' "$password" | pnpm standalone:admin reset-password you@example.com
unset password
```

That needs [pnpm](https://pnpm.io/). Without it:

```sh
docker compose exec -T runtime node /opt/standalone/admin.mjs reset-password you@example.com
```

Pass the password on standard input the same way. A reset signs that person out everywhere.

To update, back up first, then pull:

```sh
node scripts/celld/backup.mjs backup ./backups/before-update
docker compose pull
docker compose stop web crawl-gateway runtime
docker compose up -d
docker compose logs init
```

The backup command needs Node.js on the host. Backups contain secrets and your data. Keep the folder private. Restoring into a stack that already has data is refused. The procedure is in [Docker self-hosting](./SELF_HOSTING_DOCKER.md#backup-and-restore).

Going back to an older image does not undo database migrations. Restore the matching backup instead.

An older single-container image, `ghcr.io/every-app/open-seo`, still receives updates. New installs should use the steps above. Moving an old volume across is documented in [Move from the old Docker image](./SELF_HOSTING_DOCKER.md#move-from-the-old-docker-image).

## Cloudflare

Use this when you do not want a server. You need:

- Node.js 22.6 or newer, and pnpm. This repository pins `pnpm@10.30.1`. `corepack enable` selects it.
- A Cloudflare account with R2 enabled. The first time you open R2, Cloudflare asks for a payment method, including on the free tier.
- A workers.dev subdomain. A new account has none until you open Workers & Pages in the dashboard once.

```sh
git clone https://github.com/Transformaiton/open-seo.git
cd open-seo
corepack enable
pnpm install
pnpm alchemy login deploy/alchemy/alchemy.run.ts
pnpm alchemy cloudflare bootstrap
cp deploy/.env.selfhost.example .env.selfhost
```

Set `DATAFORSEO_API_KEY` in `.env.selfhost`, then:

```sh
pnpm deploy:selfhost --yes
```

Open the worker URL printed at the end and create the administrator account before you share the URL. Add your website on the next screen.

To update later: `git pull`, `pnpm install`, `pnpm deploy:selfhost --yes`.

A custom domain, Cloudflare Access, password resets, and tearing the deploy down are in [Cloudflare self-hosting](./SELF_HOSTING_CLOUDFLARE.md). Read the Workers Free plan limit there before you rely on it for a large audit.

Cloudflare installs can send anonymous usage counts. Set `OPENSEO_TELEMETRY_DISABLED=1` in `.env.selfhost` before you deploy if you do not want that. See [Telemetry](./SELF_HOSTING_CLOUDFLARE_OPERATIONS.md#telemetry). The current Docker install forces telemetry off.

## Troubleshooting

**The Docker site never opens.** `docker compose logs --tail 100 init runtime storage crawl-gateway web` and `docker compose ps`. If `init` failed, Nginx does not start, and the init log says why. `curl --fail http://localhost:3001/api/health` is the check that sign-in and the database are up.

**"API key" or empty research results.** Confirm `DATAFORSEO_API_KEY` is the Base64 credential. Confirm the DataForSEO account still has credit. After any `.env` change:

```sh
docker compose up -d --force-recreate init web crawl-gateway
```

**I changed the public URL and everyone was signed out.** `BETTER_AUTH_URL` is the address the session expects. Set it to the URL people actually type, including `https://`, and recreate `init` and `web` as above.

**A Raspberry Pi 5 exits immediately.** The default Pi kernel uses 16 KB memory pages. This runtime needs 4 KB pages. Add `kernel=kernel8.img` to `/boot/firmware/config.txt` and reboot. `getconf PAGESIZE` should print `4096`.

**Cloudflare deploy says you need a workers.dev subdomain.** Open Workers & Pages in the Cloudflare dashboard once, then run the deploy again.

**Cloudflare deploy fails on R2.** Open R2 in the dashboard and add the payment method Cloudflare asks for, then deploy again.

**An audit stops early on Cloudflare.** On the Workers Free plan a workflow can make 50 outgoing requests, so an audit stops at about 40 pages. Use Docker, or a paid Workers plan, for a full site.

**I forgot the admin password.** There is no reset email. Use the admin command in the Docker section above, or [Manage accounts](./SELF_HOSTING_CLOUDFLARE_OPERATIONS.md#manage-accounts) on Cloudflare.

**I ran `docker compose down -v`.** The `-v` flag deletes volumes. That removes the database and the generated secrets. There is nothing to restart. Restore a backup, or start over and create the admin account again.

**The screen links to app.openseo.so or asks about a hosted plan.** Those pages belong to the original hosted service. This install bills DataForSEO on your own key. You do not need an openseo.so subscription to use the copy you just started. Menu names inside the app still say OpenSEO.

## Help

Email <info@transformaiton.com> with the command you ran and the error text. Leave out passwords and API keys.

Website: <https://transformaiton.com/>. GitHub: <https://github.com/Transformaiton/open-seo>.
