# Open SEO by Transformaiton.com

Open SEO is a self-hosted search tool for a small business. You use it to see which phrases people search, how a competitor shows up in Google, who links to a site, and what is wrong on your own website.

You run it on a computer you control, or on your own Cloudflare account. The software costs nothing. Search volumes, rankings, and backlink data come from [DataForSEO](https://dataforseo.com/), and you pay DataForSEO for those lookups.

The screens still say OpenSEO. That is the name of the application. This page is Transformaiton.com's guide to running it.

![Keyword ideas for a search, with monthly volume, cost per click, and intent](web/public/library/keyword-research/keyword-difficulty-volume-openseo.png)

The pictures on this page are already in this repository, under `web/public/library/`. They show the OpenSEO screens. They were not taken on a Transformaiton.com server.

## What you can do with it

- Keyword research. Type a phrase a customer might search, such as "emergency plumber". The app lists related phrases with monthly search volume, what advertisers pay per click, a competition score, and whether the search is someone trying to buy, to learn, or to find a specific site.
- Competitor check. Open Domain Overview and enter another company's domain. You get an estimate of their organic traffic and the keywords they rank for, with the page that ranks.
- Backlink check. Open Backlinks and enter your domain or a competitor's. You see how many sites link to it, how many different domains those links come from, and whether that number grew or shrank over the last year.
- Site audit of your own site. The crawler reads your pages and lists problems such as broken links, missing titles, and pages Google is told not to index. A normal crawl stays on your site.

Rank tracking, Google Search Console, and an optional in-app assistant are there too. [Usage](docs/USAGE.md) walks through the four jobs above.

![Domain overview for a competitor, with estimated organic traffic and the keywords that site ranks for](web/public/library/competitive-analysis/competitor-domain-overview-openseo.png)

![Backlink overview, with referring domains and a year of new and lost links](web/public/library/link-building/backlink-overview-openseo.png)

![Site audit issues grouped as critical, warning, and info](web/public/library/site-audit/audit-report-by-severity-openseo.png)

## What you need, and what it costs

| Requirement | Who you pay | Cost |
| --- | --- | --- |
| This software | Nobody | $0. The license is MIT. See [LICENSE](LICENSE). |
| Search data | [DataForSEO](https://dataforseo.com/) | Pay per lookup. A new account includes $1 of credit. The smallest top-up is $50. A default keyword search is about $0.04. A backlink overview of one domain is about $0.06. |
| Docker on your own machine | You, if you rent a server | The stack usually uses 1 to 2 GB of memory. Give Docker at least 4 GB. |
| Cloudflare, if you use that path | Cloudflare | The free Workers plan can run it. Site audits on that plan stop at about 40 pages. Turning on R2 asks for a card, including on the free tier. |
| In-app assistant | [OpenRouter](https://openrouter.ai/), optional | Only if you add an OpenRouter key. Keyword research, backlinks, domain checks, and audits work without it. |
| Search Console and Analytics | Google | Free. A self-hosted install needs your own Google Cloud OAuth client. |
| JavaScript rendering in an audit | Context.dev on Docker, or Cloudflare on a paid Workers plan | Optional. On Docker, Context.dev's Developer-plan list price recorded in the app is $0.0025 per page. On Cloudflare, Browser Run is $0.09 per browser hour after the hours included with Workers Paid. A plain crawl does not call DataForSEO. |
| Map on the Local SEO screen | Google | Optional. Google bills the Maps key if you add one. |

Those DataForSEO figures are the ones recorded in this project's pricing model, checked against DataForSEO in July 2026. The default keyword-search price was measured in September 2026. DataForSEO can change its prices.

You pay DataForSEO's price on this install. The original hosted service at openseo.so adds 28% on top of that price. The rank-tracking form still prints the marked-up number, about $0.0150 per keyword per check at the default depth on one device. DataForSEO's price for that same check is about $0.0117 per keyword. One hundred keywords, once a week, is up to about $4.70 a month at DataForSEO's price. Daily is 30 checks a month, and weekly is 4. Desktop and mobile together cost about twice one device. A plain site audit does not show a dollar amount on this install, because the crawl itself is not a DataForSEO lookup. Lighthouse, if you turn it on, is $0.005 per result in the app's price table.

## Quick start

This is the Docker path, which fits one business website on a computer you can sit at. The full steps, and the Cloudflare path, are in [Install](docs/INSTALL.md).

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine with Compose). In Docker's settings, set memory to at least 4 GB.
2. Create a DataForSEO account and copy the credential labeled **Base64** on [API Access](https://app.dataforseo.com/api-access).
3. Clone this repository and start the app:

```sh
git clone https://github.com/Transformaiton/open-seo.git
cd open-seo
export COMPOSE_FILE=deploy/standalone/compose.yaml
cp deploy/standalone/.env.example deploy/standalone/.env
```

4. In `deploy/standalone/.env`, set `DATAFORSEO_API_KEY` to that Base64 credential. Set `STORAGE_SECRET_KEY` to a long random value (`openssl rand -hex 32`). Leave `BETTER_AUTH_URL=http://localhost:3001` if you are using it on this computer.
5. Start it and create the admin account:

```sh
docker compose pull
docker compose up -d
docker compose logs -f init
```

Wait until the log says `OpenSEO initialized and ready.` Open <http://localhost:3001> and create the administrator account before anyone else can open that address. Registration closes after the first account exists.

The Compose file pulls `ghcr.io/every-app/open-seo-standalone`, the image published by the original project. That address is what this install runs.

## Install and day-to-day use

- [Install](docs/INSTALL.md) covers Docker, Cloudflare, backups, updates, and the problems that stop a first start.
- [Usage](docs/USAGE.md) covers keyword research, a competitor check, a backlink check, and an audit of your own site.
- Technical references, if you need them: [Docker](docs/SELF_HOSTING_DOCKER.md), [Cloudflare](docs/SELF_HOSTING_CLOUDFLARE.md), [DataForSEO key](docs/DATAFORSEO_API_KEY.md), [Google Search Console](docs/SELF_HOSTING_GOOGLE_SEARCH_CONSOLE.md), [Google Analytics](docs/SELF_HOSTING_GOOGLE_ANALYTICS.md).

## Troubleshooting

- The site does not load. Run `docker compose ps` and `docker compose logs --tail 100 init`. If `init` failed, the web server does not start. `curl --fail http://localhost:3001/api/health` succeeds only when sign-in is configured and the database answers.
- Lookups fail or the key is rejected. `DATAFORSEO_API_KEY` must be the Base64 credential from DataForSEO, not the account password on its own. After you edit `.env`, run `docker compose up -d --force-recreate init web crawl-gateway`.
- You cannot create a second account from the sign-up page. That is expected. Invite people from Team settings. The app copies a link, and you send it yourself. There is no invitation email.
- A site audit on Cloudflare stops around 40 pages. That is the Workers Free plan limit. Docker, or Workers Paid, is the path for a larger audit.
- `docker compose down -v` deletes the installation, including the database and the generated secrets.

More cases are in [Install](docs/INSTALL.md#troubleshooting).

## FAQ

### Do I need a Semrush or Ahrefs subscription?

No. This app does not log in to those products. Keyword, ranking, and backlink numbers come from your DataForSEO account. Their databases are not the same as Semrush or Ahrefs, which run their own crawlers.

### Is the software free?

The software is free to run. DataForSEO charges for the lookups. A new DataForSEO account includes $1 of credit, which is enough to try a few searches. The smallest amount you can add after that is $50.

### Can I check my own site only?

Yes. A site audit crawls the site you name and stays on that host. Domain Overview and Backlinks also accept your own domain, if you want the traffic estimate and the links pointing at you. Keyword research does not need a domain. You can type the phrases you care about.

### Where does my data sit?

On a Docker install, the database and files stay in Docker volumes on that machine. On a Cloudflare install, they sit in your Cloudflare account (D1, KV, and R2). Transformaiton.com does not receive your keywords, sites, or DataForSEO key.

The current Docker install turns usage telemetry off. A Cloudflare install can send anonymous counts (a random install id, and totals such as number of projects). It does not send URLs, keywords, or email addresses. To turn that off, set `OPENSEO_TELEMETRY_DISABLED=1` in `.env.selfhost` and deploy again. Details are in the [Cloudflare operations guide](docs/SELF_HOSTING_CLOUDFLARE_OPERATIONS.md#telemetry).

### Can I use ChatGPT or Claude with it?

Yes, after the app is running. An assistant connects to `/mcp` on your install and signs in with the account you created. The steps are in [Usage](docs/USAGE.md#connect-an-ai-assistant). The plugin pages that point at `https://app.openseo.so/mcp` are for the original hosted service, not for this copy.

## Contact

Transformaiton.com is an AI consultancy for small businesses.

- Website: <https://transformaiton.com/>
- Email: <info@transformaiton.com>
- GitHub: <https://github.com/Transformaiton>

Email info@transformaiton.com if you are stuck on this copy. Include what you tried and the error text. Do not include passwords, API keys, or the Base64 credential.

Transformaiton.com cannot sign you in to openseo.so, change a bill there, or speak for the original authors.

## Credits

Open SEO is a copy of [OpenSEO](https://github.com/every-app/open-seo), published under the MIT license. Copyright (c) 2026 Ben Senescu. The copyright notice and the permission notice are in [LICENSE](LICENSE), unchanged.

Transformaiton.com is not affiliated with, and is not endorsed by, the original authors.

### Modifications by Transformaiton.com

This copy changes the customer documentation: this README, [docs/INSTALL.md](docs/INSTALL.md), [docs/USAGE.md](docs/USAGE.md), and short notes in the self-hosting guides so they point here instead of at the original hosted plan. The application code is the upstream project. Docker still runs the image the upstream project publishes, `ghcr.io/every-app/open-seo-standalone`.

People who want to change the code can follow [local development](docs/LOCAL_DEVELOPMENT.md). Issues for the original project belong on [every-app/open-seo](https://github.com/every-app/open-seo). Issues for this guide belong on [Transformaiton/open-seo](https://github.com/Transformaiton/open-seo).
