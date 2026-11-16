# Use Open SEO

This guide assumes the app is already running and you are signed in as the administrator. Install steps are in [INSTALL.md](./INSTALL.md). Costs are in the [README](../README.md#what-you-need-and-what-it-costs).

The interface says OpenSEO. The sidebar is grouped. **Research** holds Keyword Research, Domain Overview, and Backlinks. **My Site** holds Site Audit and Rank Tracking.

Each lookup that asks DataForSEO for new numbers spends money on your DataForSEO account. Saving a keyword, reading an audit you already ran, and reading Google Search Console do not.

## Set the website

On the first sign-in the app asks "What's your website?" Type your domain, such as `example.com`, and continue. That names the project and starts a 50-page audit of the site.

To add another site later, use the project switcher in the sidebar and choose **New project**. The name is required. The domain can wait, but the research tools are more useful once it is set. The country and language on the project are what keyword and domain lookups use unless you change them on that search. New projects default to the United States.

## Keyword research

Use this when you want phrases to put on a page, or a check on whether anyone searches for the service you sell.

1. Open **Keyword Research**.
2. Type a phrase in "Enter a keyword, or a pattern like best * for dogs". Press Enter, or click **Search**. Shift+Enter adds another phrase in the same search.
3. Read the table. Volume is the estimated monthly searches. CPC is what advertisers pay per click. Comp. is a competition score. Intent is a short label: commercial, transactional, informational, or navigational.
4. Use **Filters** when the list is too long. **Export** sends the rows out of the app.

A default search is about $0.04 on DataForSEO, at the price recorded in September 2026. DataForSEO can change it.

What to do with the list: keep phrases that describe a service you actually offer, with enough volume to be worth a page, and an intent that matches the page. A phrase marked informational needs an explanation. A phrase marked transactional needs a page where someone can hire you or buy.

![Keyword ideas, with volume, cost per click, and intent](../web/public/library/keyword-research/keyword-difficulty-volume-openseo.png)

This picture is from `web/public/library/` in the repository. It shows the keyword table. It is not a screenshot of a Transformaiton.com server.

## Competitor check

Use this when you want to see which searches already send people to another company.

1. Open **Domain Overview**.
2. Enter the competitor's domain or a page URL in "Enter a domain or URL". Click **Search**.
3. Read estimated organic traffic and the organic keyword count.
4. Open **Top Keywords** for the phrases they rank for, the position, the search volume, and the URL that ranks. **Top Pages** lists the pages that pick up the most of that traffic.

You can run the same check on your own domain. The numbers are estimates from DataForSEO's index. They will not match Google Search Console, which is measured on your site. Search Console is the better source for your own clicks. Domain Overview is the tool that can see a site you do not own.

The keyword list can be long. Filter it before you treat every row as a page you should write. Brand names and close variants inflate the count.

![Domain overview for a competitor, with traffic and ranking keywords](../web/public/library/competitive-analysis/competitor-domain-overview-openseo.png)

## Backlink check

Use this to see who links to you, or who links to a competitor.

1. Open **Backlinks**.
2. Enter a domain or URL. Click **Search**.
3. Read the totals: backlinks, referring domains, referring pages, rank, and spam score. The charts show the last year, including links gained and links lost.

A link from a relevant business in your industry is worth more than a large count. The spam score is a sort aid from DataForSEO. It is not a verdict.

One domain overview of this kind is about $0.06 at the price recorded in July 2026 (a summary plus one year of daily history).

Run it on your domain first, then on two or three competitors. The gap that matters is referring domains from real companies in your market, not the raw backlink total.

![Backlink overview, with referring domains and a year of link growth](../web/public/library/link-building/backlink-overview-openseo.png)

## Site audit of your own site

Use this to find pages on your website that are broken, thin, or hard for Google to index. Point it at a site you own. The crawler stays on that host. `www.example.com` and `example.com` count as the same site. Other subdomains are skipped.

1. Open **Site Audit**.
2. Enter the URL to start from. `example.com` is enough.
3. Set **Max pages**. The first audit the app starts for you uses 50. A Docker install can go up to 10,000. A Cloudflare install on the free Workers plan stops at about 40.
4. Leave **Include Lighthouse** and **Render JavaScript** off for a first pass, unless you know you need them. Lighthouse measures speed on a sample of pages. Render JavaScript loads each page in a browser. On Docker the switch stays off until you set `CONTEXT_API_KEY`. On Cloudflare it needs a paid Workers plan. This install does not print a dollar estimate on the audit form. The hosted service does.
5. Click **Start Audit**.

When it finishes:

- **Issues** groups findings as Critical, Warning, and Info. Open one to read what it means, how to fix it, and up to 100 affected URLs.
- **Pages** lists status code, title, H1, and word count. Filter **Crawl result** to Failed or Blocked if the issue list looks empty. An empty issue list often means the crawler was blocked, not that the site is clean.
- **Export** can download CSV or JSON.

There is no re-run button. Start a new audit after you fix pages. Deleting a running audit stops it.

If the report says the site's bot protection blocked the crawler, the crawler identifies itself as `OpenSEO-Audit`. Allow that agent in your firewall, or on Shopify under Online Store, Preferences, Crawler access. Then connect the signature in Project settings, Integrations.

![Site audit issues grouped by severity](../web/public/library/site-audit/audit-report-by-severity-openseo.png)

A plain crawl does not call DataForSEO. Lighthouse does. The price recorded in the app is $0.005 per Lighthouse result. The audit tests up to 10 page templates, on mobile and desktop, so a full run is about $0.10 at that rate. JavaScript rendering on Docker is billed by Context.dev at $0.0025 per page on the Developer plan, which is the list price recorded in the app. On Cloudflare it is Browser Run, at $0.09 per browser hour after the hours included with the paid Workers plan. The dollar line the original hosted docs quote (about $0.14 for Lighthouse, about $1.28 per 1,000 rendered pages) is the hosted price, including a 28% fee. It is not what this install charges, and the self-hosted form does not show it.

## Rank tracking

Open **Rank Tracking** under My Site when you want a weekly position for phrases you already chose.

Add a domain, pick the country, and choose a device. Mobile only is the default. Desktop and mobile together cost about twice as much. The default depth is the top 40 results. A rank below that is reported as not found.

The schedule defaults to weekly. The form shows an "up to" price before you save. That figure includes the 28% hosted-service markup, about $0.0150 per keyword per check at the default depth on one device. DataForSEO charges about $0.0117 for the same check. One hundred keywords, weekly, is up to about $4.70 a month on the DataForSEO bill. A check usually takes about an hour, and can take up to 2 hours.

"Not checked" means Google returned no result for that keyword on that run. The last known position stays. It is not the same as dropping off the page.

## Google Search Console

Search Console shows the clicks and impressions Google measured for your site. It is free, and it is optional.

On this self-hosted copy you create your own Google Cloud OAuth client first. The steps take about ten minutes: [Search Console](./SELF_HOSTING_GOOGLE_SEARCH_CONSOLE.md) and [Analytics](./SELF_HOSTING_GOOGLE_ANALYTICS.md). Then, in the project, open Project settings, Integrations, and connect.

After it is connected, **GSC Insights** lists queries where you already rank between about 5th and 20th. Those are often the pages worth improving before you write something new.

## Connect an AI assistant

The app speaks MCP at `/mcp` on the same host as the site.

- Docker on this computer: `http://localhost:3001/mcp`
- A public install: `https://YOUR-HOST/mcp`

In the client, add that URL as an HTTP MCP server and sign in with the email and password you created here. For a client that cannot open a browser, create a key under Settings, API keys, and send it as `Authorization: Bearer oseo_...`. The key is shown once.

Do not paste `https://app.openseo.so/mcp` if you are using this copy. That address is the original hosted service. It will not see the projects on your install.

The assistant can run the same jobs as the screens: keyword research, a domain overview, backlinks, and a site audit. Ask it to show the cost before it starts a rank check or a rendered audit.

## A sensible first week

1. Finish the first 50-page audit and fix anything listed as Critical, starting with broken pages.
2. Run Keyword Research for the three services you most want to be hired for. Save the phrases that match a page you can actually publish.
3. Run Domain Overview and Backlinks on your domain and on two competitors.
4. Connect Search Console if you already have it, and compare its clicks with the keyword list. Trust Search Console for your own traffic.

## FAQ

### The audit says no issues. Is the site fine?

Open **Pages** and set **Crawl result** to Failed or Blocked. If many pages are there, the crawler did not read them, so it had nothing to judge.

### A competitor's traffic number looks huge.

It is an estimate from DataForSEO, and brand searches inflate it. Sort by phrases a customer would type, and ignore the company's own name, before you decide they are winning a market.

### I do not want to spend the $50 top-up yet.

The $1 of starting credit covers a handful of keyword searches and a backlink overview. Stop when DataForSEO refuses the call. The app does not have a separate Transformaiton.com bill.

### Who do I ask?

Email <info@transformaiton.com>. Do not send the DataForSEO credential or your password. Transformaiton.com is not the original author of OpenSEO, and is not endorsed by them. The upstream project is <https://github.com/every-app/open-seo>.
