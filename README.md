# DMARC Report Viewer

A small, single-file web app for reading DMARC **aggregate (RUA) reports**.
Drop in the XML files your mail providers send you and see who is sending email
as your domain, and whether it passed or failed DMARC.

- **No server, no build step, no tracking.** One `index.html`.
- **Private by design.** Reports are parsed locally in your browser and are never uploaded.
- **Works offline.** The only dependency ([fflate](https://github.com/101arrowz/fflate)) is bundled into the file.

## Features

- Accepts `.xml`, `.gz` and `.zip` files, many at once (duplicate reports are skipped)
- Summary: total emails, DMARC pass rate, failures, quarantined, rejected, unique IPs, date range
- Reports table: reporter (Google, Microsoft, Yahoo...), domain, published policy, period, volume, pass rate
- Sending sources table: source IP, count, DMARC result, disposition, SPF/DKIM results,
  header-from, envelope-from, SPF/DKIM authenticated domains, policy override reason
- Sort any column, text filter, "only failing" toggle, disposition filter
- Export the filtered rows to CSV
- Light and dark mode, mobile friendly

## Usage

### Locally
Open `index.html` in any modern browser and drag your report files onto the page.

### Deploy on Cloudflare Pages (from GitHub)

1. Push this repo to GitHub (private is fine).
2. Cloudflare dashboard -> **Workers & Pages** -> create a project -> connect the repo.
3. Framework preset: **None**. Build command: *(empty)*. Output directory: `/`.
4. Deploy. Every push to the repo redeploys automatically.

The included `_headers` file adds a strict Content-Security-Policy and other hardening
headers (optional, safe to delete). The page is search-engine friendly: `robots.txt` allows
crawling and `index.html` has description, Open Graph and author metadata for Manish Kafle.

### Optional: restrict access

A deployed Pages site is public by default, which is what you want if you want it indexed.
Cloudflare Access is optional and **will prevent search engines from indexing the site**.
To restrict it anyway:

1. Cloudflare **Zero Trust** -> **Access** -> **Applications** -> add a self-hosted application.
2. Cover your custom domain **and** the `*.pages.dev` address.
3. Add an Allow policy for your email address(es), using one-time PIN login.

The tool never sends report data anywhere, so Access only controls who can open the page.

## How results are interpreted

A record counts as a **DMARC pass** when SPF *or* DKIM passes **and** aligns with the
header-From domain, as reported in the receiver's `policy_evaluated` block. Anything else is a fail.

| Disposition | Meaning |
|---|---|
| `none` | Delivered normally (either passed, or policy is `p=none`) |
| `quarantine` | Receiver treated it as suspicious (e.g. spam folder) |
| `reject` | Receiver refused the message |

Failing rows from IPs you recognise (your CRM, practice management software, website
mailer, newsletter tool) are legitimate senders that need SPF/DKIM alignment fixed
before you move to `p=reject`. Failing rows from IPs you don't recognise are most likely spoofing.

## Limitations

- Aggregate (RUA) reports only. Forensic/failure (RUF) reports are not supported.
- No reverse DNS or IP ownership lookup (that needs a server-side lookup).
- Data lives in memory only. Reloading the page clears it.

## Security notes

- All values from the XML are HTML-escaped before display.
- Do **not** commit real report files to the repository. The `.gitignore` excludes `*.xml`, `*.gz` and `*.zip`.

## Project layout

```
index.html    the whole app (HTML + CSS + JS + bundled fflate)
_headers      Cloudflare Pages security headers (optional)
robots.txt    allows crawlers
LICENSE       MIT
README.md     this file
.gitignore    keeps report files out of git
```

## Author

**Manish Kafle**
GitHub: [@K4fledmanish](https://github.com/K4fledmanish) | Profile: [k4fledmanish.github.io](https://k4fledmanish.github.io/)

## License

[MIT](LICENSE) (c) 2026 Manish Kafle. Bundled fflate is also MIT licensed (see `LICENSE`).
