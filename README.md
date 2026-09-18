# Cited

**Check whether AI assistants name your product when your buyer asks them what to buy.**

Ranking #1 on Google no longer means being in the answer. Buyers open ChatGPT, Claude, Perplexity or Gemini, ask *"best billing tool for a seed-stage SaaS on Stripe"*, and shortlist whatever the model names. Cited measures that — prompt by prompt, engine by engine.

One HTML file. No backend, no build step, no account. It runs with zero API keys.

---

## What it does

You give it your brand, your aliases, your domain and your competitors, then the questions your buyer would actually type. It builds a grid — one square per prompt per engine — and scores each answer out of 100.

| Signal | Weight | Why it counts |
|---|---|---|
| Named at all | 32 | Everything else is zero if the model never says your name |
| Position in the list | 34 | Decays steeply (`0.72^(rank-1)`); being fourth is worth a third of being first |
| How early you appear | 10 | A mention in the closing caveat is not a recommendation |
| Your domain in the answer | 14 | A named brand with no link is a dead end for the reader |
| How you're described | 10 | "Purpose-built for" and "overkill unless" are not the same result |

Alongside the grid you get:

- **Share of voice** — how often you're named against each competitor in the same answers
- **Cited sources** — every domain the answers leaned on, which is the third-party layer worth winning
- **A fix kit** — `llms.txt`, a `robots.txt` that lets the answer-engine crawlers in, FAQ schema, and an answer-first page skeleton, all generated from what you entered
- **Export** — JSON or CSV, so you can diff this week against last week

## Two ways to collect answers

**Paste mode — no keys.** Click a square, copy the prompt, open the real consumer app, paste the reply back. Slower, and better: you're scoring the logged-in, personalised, retrieval-backed answer your buyer actually sees, which is not what an API returns.

**Live mode — your keys.** Paste a key per provider and the whole grid fills itself. Set runs-per-prompt to 2 or 3, because model answers wobble between runs and one sample is not a measurement.

| Engine | Live mode | Default model |
|---|---|---|
| ChatGPT | yes | `gpt-4o-mini` |
| Claude | yes | `claude-sonnet-4-5` |
| Perplexity | yes* | `sonar` |
| Gemini | yes | `gemini-2.0-flash` |
| Google AI Mode | paste only | — |

\* Browser-to-provider calls depend on each API sending CORS headers. Anthropic supports it explicitly, Gemini and OpenAI generally work, Perplexity may refuse from a hosted origin. Any engine that fails falls back to paste mode with a clear message rather than a silent dead button.

## Running it

**Locally.** Download `index.html` and open it. That's the whole install. If a provider rejects the `file://` origin, serve it instead:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

**Deploying it.** Drag the folder onto [Netlify Drop](https://app.netlify.com/drop), or point any static host at it. The `_headers` file ships the content security policy as a real response header rather than only a `<meta>` tag.

## About the API key question

The honest version: you shouldn't paste a key into a page you don't know on trust alone. So the design doesn't ask you to. Six answers, in order of how paranoid you want to be.

1. **Use no key.** Paste mode produces every number on the page.
2. **Take it off the internet.** Save the file, open it from your own disk, and the site you don't know is now a file you own.
3. **Read it.** Search for `fetch(` — there are four call sites. No bundler, no minified blob.
4. **Let the browser enforce it.** The file ships a CSP whose `connect-src` allows exactly four hosts: `api.openai.com`, `api.anthropic.com`, `api.perplexity.ai`, `generativelanguage.googleapis.com`. If this page tried to post your key anywhere else, the browser would refuse the request rather than ask permission.
5. **Watch it work.** Every outbound call is logged in the page with host, status, bytes and duration. Calls to anything off the allowlist are blocked in code before the network is touched, and logged in red.
6. **Make the key worthless.** Cut a fresh key with a $1 cap, run the check, revoke it.

Keys are held in memory by default and disappear on refresh. Session and device storage are opt-in, per the dropdown, and "Wipe keys now" clears both.

If your policy is that keys never touch a browser at all, the page ships a copyable Cloudflare Worker: deploy it on your own domain, keep the key in an environment variable, point the endpoint field at your worker.

### If you fork and host this

Hosting it changes who the trust question is about — visitors are now trusting you not to swap the file tomorrow. Three things keep that claim honest:

- Deploy from the repo, so what's served is traceable to a commit anyone can read
- Keep the CSP intact
- Don't add analytics, chat widgets or marketing pixels. The CSP will block them, and the only way to make them work is to widen the allowlist, which quietly destroys the guarantee. Server-side host analytics doesn't touch the page

## Configuration

Everything lives in `index.html`.

- **Engines and default models** — the `ENGINES` array near the top of the script
- **Network allowlist** — the `ALLOW` array, which must stay in sync with `connect-src` in the CSP meta tag. Adding a proxy endpoint means editing both
- **Scoring weights** — the `analyse()` function, deliberately written as five readable lines you can argue with

## Limitations, stated plainly

This is a diagnostic, not an oracle.

- Model answers vary run to run, by account, by region and by model version. A single square is a sample.
- API answers and consumer-app answers are different products. Paste mode is closer to the truth; live mode is closer to scale.
- Tone detection is a small keyword lexicon, not sentiment analysis. It's directional.
- Rank parsing reads ordered lists, bullets, bold headings and headers. Prose answers fall back to order of first mention.
- The value is the trend. Run the same prompts weekly and watch the grid change; don't screenshot one run and call it a benchmark.

## Roadmap

- Snapshot history with week-over-week diffing
- Prompt packs by category
- Optional headless collection for the engines without a public API
- Competitor grids, so you can run the check on someone else's brand

## Contributing

Issues and pull requests welcome. Two rules: it stays a single file with no build step, and nothing gets added that widens the network allowlist without a very good reason spelled out in the PR.

## License

MIT.
