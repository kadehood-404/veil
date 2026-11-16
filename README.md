# Veil

**Redact secrets before you paste. Restore them after.**

Veil is a reversible, 100% client-side redactor for logs, configs, stack traces and code. It swaps API keys, tokens, passwords, emails, IPs and other sensitive values for stable placeholders, so you can debug with ChatGPT, Claude, a forum or a GitHub issue without leaking anything. Paste the reply back and Veil puts your real values back in.

![license](https://img.shields.io/badge/license-MIT-9d6bff)
![dependencies](https://img.shields.io/badge/dependencies-none-39ff9c)
![runs](https://img.shields.io/badge/runs-100%25%20in%20your%20browser-39ff9c)

**Live:** https://veil.sigil-framework.space

---

## Why

Everyone pastes logs into chatbots. Logs are full of things that should never leave your machine: cloud keys, database passwords, session tokens, customer emails, internal hostnames. Deleting them by hand is slow and error-prone, and it breaks the context the AI needs to help you.

Veil keeps the *structure* and removes the *secrets*. The same value always maps to the same placeholder, so `{{IP_1}}` is still recognizably the same host everywhere it appears, and the answer you get back still makes sense.

## How it works

1. **Paste** your text into the left box.
2. **Copy** the redacted version from the right box and send it wherever you like.
3. **Paste the reply** into box 3. Veil restores your real values in box 4.

### Before

```text
2026-11-16 03:12:44 ERROR deploy failed for kadehood.404@example.com from 203.0.113.42 (mac 00:1A:2B:3C:4D:5E)
Authorization: Bearer abcDEF1234567890abcDEF1234567890
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
db_password = "hunter2hunter2"
connecting to postgres://admin:s3cr3tpw@db.internal:5432/app
GITHUB_TOKEN: ghp_abcdefghijklmnopqrstuvwxyz0123456789
config at /home/kadehood/projects/secret-app/config.yml
```

### After

```text
2026-11-16 03:12:44 ERROR deploy failed for {{EMAIL_1}} from {{IP_1}} (mac {{MAC_1}})
Authorization: Bearer {{BEARER_TOKEN_1}}
AWS_ACCESS_KEY_ID={{AWS_KEY_1}}
db_password = "{{SECRET_1}}"
connecting to postgres://admin:{{URL_PASSWORD_1}}@{{HOST_1}}:5432/app
GITHUB_TOKEN: {{GITHUB_TOKEN_1}}
config at /home/{{USERNAME_1}}/projects/{{CUSTOM_1}}/config.yml
```

(All values above are fake examples.)

## Features

- **Reversible.** Restores placeholders in any text, even when an AI reformats them as `<EMAIL_1>`, `[IP_1]` or a bare `EMAIL_1`.
- **Consistent.** One value, one placeholder, across multiple pastes in a session.
- **Controllable.** Toggle each detector, add custom terms (names, codenames, hostnames), and uncheck false positives in the mapping table.
- **Portable.** Export and import the mapping as JSON, or keep it across a refresh for the current tab only.
- **Private by design.** No server, no uploads, no network requests, no dependencies. One HTML file.

## What it detects

| Category | Examples |
| --- | --- |
| Private keys | PEM `-----BEGIN … PRIVATE KEY-----` blocks |
| Tokens and API keys | JWTs, AWS, GitHub, Slack, Stripe, Google, OpenAI/Anthropic-style (`sk-…`), bearer tokens |
| Assignments | `password=`, `secret:`, `token=`, `api_key=`, `client_secret=` and similar |
| URL credentials | `scheme://user:password@host` |
| Identity | Emails, usernames in `/home/…`, `/Users/…`, `C:\Users\…` paths |
| Network | IPv4, IPv6, MAC addresses, internal hostnames (`.local`, `.internal`, `.lan`, `.corp`) |
| Financial / PII | Credit cards (Luhn-checked), US SSNs |
| Optional (off by default) | All domains, US phone numbers, UUIDs, high-entropy strings |
| Custom | Any terms you list, one per line |

## Privacy and limitations

**Everything happens in your browser.** Detection, redaction and restoration are plain JavaScript running locally. The mapping between placeholders and real values lives in the page's memory only, unless you explicitly keep it for the tab (`sessionStorage`) or export it. An exported map contains your real secrets, so treat it as sensitive.

**Veil is best-effort, not a guarantee.** It is pattern-based, so it cannot know that "Acme" is your client or that a sentence mentions your colleague by name. Always skim the output before you share it, and add anything sensitive to **Custom terms**. A leaked secret that Veil missed is still leaked.

Known gaps:
- Names, addresses and other free-text PII are not detected unless you add them as custom terms.
- Secrets with no recognizable prefix or label may slip through (the optional high-entropy detector helps, but is noisy).
- Placeholders are plain text. If an AI rewrites one beyond recognition, it can't be restored automatically.

## Run it

### Use it online

Open **https://veil.sigil-framework.space**. Nothing to install, and your text never leaves the page.

### Run it locally

No build step, no install. Handy if you want to work offline or audit exactly what you're running.

```bash
git clone https://github.com/<you>/veil.git
cd veil
open index.html        # or just double-click it
```

Or serve the folder with anything static:

```bash
python3 -m http.server 8080
```

## Deploy

Veil is a static site. On Cloudflare Pages, connect the repo with **framework preset: None**, **build command: empty**, **output directory: `/`**, then add your custom domain. It works the same on GitHub Pages, Netlify or any static host.

`robots.txt` and `sitemap.xml` are included for SEO. If you host it elsewhere, update the URLs in both files and the canonical and Open Graph tags in `index.html`.

## Add your own detector

Rules live in the `RULES` array in `index.html`. Each rule is a regex plus metadata:

```js
{
  id: 'myapp',                 // unique id (used by the toggle UI)
  cat: 'MYAPP_KEY',            // placeholder prefix → {{MYAPP_KEY_1}}
  label: 'MyApp API keys',     // shown in the UI
  on: 1,                       // enabled by default
  p: 1,                        // priority on overlaps (lower wins)
  re: /\bmyapp_[A-Za-z0-9]{32}\b/g,
  // g: 1,                     // optional: redact only capture group 1
  // ok: t => /* validator */  // optional: reject false positives
}
```

Matches are collected from every enabled rule, then overlaps are resolved by position, then priority, then length.

## Project layout

```text
index.html     the whole app (UI + detection engine), no dependencies
robots.txt     crawler rules
sitemap.xml    sitemap for search engines
```

The detection engine (`scan`, `redact`, `restore`) lives in its own `<script id="core">` block and has no DOM dependencies, so it is easy to lift out and test in Node.

## Roadmap

- Paste-from-clipboard and copy-as-code-block buttons
- File drag-and-drop
- Per-rule sensitivity settings
- Browser extension for in-page redaction before submitting to AI chat sites

## Contributing

Issues and pull requests are welcome, especially new detectors for token formats and false-positive reports. When adding a rule, include a fake example of the format and make sure it doesn't match ordinary text.

**Never put real secrets in an issue or PR.**

## License

MIT. See `LICENSE`.
