# send0 for Cloudflare

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/send0-dev/send0-cloudflare)

send0 is an open-source email API where the inbox is the core object: one API call gives an agent or
an app an address that can send, receive, thread replies and push webhooks. This repo runs all of it
(API, dashboard, inbound mail, webhooks, real-time events) in one Worker on your own Cloudflare account.

> Generated from [send0-dev/send0-v2@eab68c4eb05b](https://github.com/send0-dev/send0-v2/commit/eab68c4eb05b09c07759e2d40d0321a65caacdb4) (send0 0.2.0).
> Don't edit here; pull requests are closed automatically. Contribute upstream at
> [send0-dev/send0-v2](https://github.com/send0-dev/send0-v2).

## What you need

- A Cloudflare account. The Workers free plan is enough to start.
- A Postgres connection string, such as a free [Neon](https://neon.tech) database. The Worker reaches it
  through Hyperdrive and creates its tables on first start.
- An AWS account with an SES identity (your mail domain) and an IAM user allowed `ses:SendRawEmail`.
  send0 sends all mail through SES, including the owner's verification email, so it won't start without it.
- A domain on Cloudflare (its nameservers point to Cloudflare) to receive mail on, like `agents.acme.com`.

## Deploy

Click the button above. Cloudflare copies this repo to your GitHub or GitLab account, creates the R2
bucket, the queue, the Hyperdrive connection and the Durable Object, and asks for:

| Setting | What to enter |
| --- | --- |
| Hyperdrive | Your Postgres connection string |
| `MAIL_DOMAINS` | The domain(s) you receive mail on, comma-separated. The first is the default for new inboxes |
| `SES_REGION` | The region of your SES identity |
| `ALLOW_SIGNUP` | Leave `false` |
| `SECRET_KEY` | 32+ random characters: `openssl rand -hex 32` |
| `OWNER_EMAIL` | Your email; the first account must use it |
| `SES_ACCESS_KEY_ID`, `SES_SECRET_ACCESS_KEY` | The IAM user's keys |

Optional settings, added later as variables under Workers & Pages > send0 > Settings:
`MAIL_FROM` (sender of account email; defaults to `noreply@` your first mail domain), `PUBLIC_URL`,
`SES_CONFIGURATION_SET`, `SES_EVENTS_TOKEN` with `SES_EVENTS_TOPIC_ARN` (see step 2), and
`OPERATOR_FORWARD_TO`: an address to forward `postmaster@` and `abuse@` on your mail domains to,
instead of refusing them. It must be a verified destination address under Email Routing > Destination
addresses.

`PUBLIC_URL`: set it if you add a custom domain, to that origin (like `https://mail.acme.com`).
send0 then redirects every other hostname, including workers.dev, to it, so sign-in, download
links and webhooks all use one URL. Unset, each request's own origin is used.

If a setting is wrong, every page shows which one and why (never its value) until you fix it.

## After deploying

1. **Email Routing.** In the Cloudflare dashboard, open each mail domain, go to Email > Email Routing
   and enable it (Cloudflare adds the MX and SPF records). Then under Routing rules, set the
   **catch-all** action to "Send to a Worker" and pick `send0` (or the name you gave the Worker).
   Mail to addresses that aren't inboxes is rejected during the SMTP session.
2. **SES.**
   - Verify the mail domain as an SES identity in `SES_REGION`, with Easy DKIM (add the three CNAME
     records it shows). A custom MAIL FROM domain and a DMARC record help deliverability.
   - Request production access. Until AWS grants it, SES only delivers to verified addresses, so
     verify your own email address in SES too, or the owner's verification email won't arrive.
   - Optional, for delivery, bounce and complaint events: create an SNS topic, set it as the event
     destination of a configuration set (and set `SES_CONFIGURATION_SET`), set `SES_EVENTS_TOKEN`
     (32+ random characters) and `SES_EVENTS_TOPIC_ARN`, then add an HTTPS subscription to
     `https://<your send0 URL>/internal/ses-events?token=<SES_EVENTS_TOKEN>` (your `PUBLIC_URL` if set). send0 confirms it.
3. **Sign up.** Open the Worker URL (`https://send0.<your subdomain>.workers.dev`, or a custom domain
   you attach), sign up as `OWNER_EMAIL` and follow the verification email. Then create inboxes and
   API keys from the dashboard, and invite your team.

## Upgrading

Each send0 release replaces this repo's contents with a new build (as one fresh commit). To upgrade
the copy the button made for you, copy the new release's files over it, keeping the resource ids the
button wrote into your `wrangler.jsonc`, then commit and push: Workers Builds redeploys it. Or deploy
by hand with `npm install && npx wrangler deploy`. Database migrations run on first start.

## Limits

- Mail is received only on domains whose DNS is on your Cloudflare account, through Email Routing
  (messages up to 25 MiB).
- Free-plan Workers allow 100,000 requests a day and 10 ms of CPU per request. Large webhook volumes,
  big attachments or busy inboxes may need Workers Paid ($5/month).
- Raw mail and attachments live in R2 (10 GB free). Message content is deleted after each inbox's
  `retention_days` (default 7, at most 30), and messages after 35 days.
- The hosted service's plan limits (reply-only free tier, daily send caps) are off: you own the SES
  account and its reputation.

## License

send0 is licensed under the [GNU AGPL v3](LICENSE). The corresponding source is described in
[SOURCE.md](SOURCE.md).
