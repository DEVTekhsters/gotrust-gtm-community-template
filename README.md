# GoTrust CMP — GTM Community Template

> **Looking for install instructions to give a customer?** See
> [INSTALLATION-GUIDE.md](./INSTALLATION-GUIDE.md) — the complete, client-facing,
> step-by-step walkthrough (mirrors the dashboard's Consent Code step, adapted for GTM).
> This README is the engineering-facing doc: what the template is, why it's built this
> way, and how to build/test/submit it to the gallery.

A Google Tag Manager (GTM) Tag template that (1) sets the Google Consent Mode
default-denied baseline via GTM's dedicated `setDefaultConsentState` template API on the
**Consent Initialization - All Pages** trigger, then (2) loads the existing GoTrust CMP
bundle and mounts the consent banner. This exists specifically to satisfy the Google CMP
Partner Program application requirement: _"If your GTM template is not published on the
community template gallery, we can NOT proceed forward with your application."_

## Why this design

Rather than reimplementing consent logic inside the GTM sandbox, the template is a thin
loader: it sets a safe default baseline immediately (defense-in-depth, in case script
injection is delayed), then defers to the already-built, already-certified-for-TCF GoTrust
bundle (`gotrust-consent-shim.js` + `cookie-banner.bundle.js`) for everything else —
returning-visitor consent restoration, the actual banner UI, TCF/AC-string generation, and
pushing `gtag('consent','update',...)` updates. GTM observes those updates natively because
they land on the same `dataLayer` GTM manages. This mirrors how other certified CMPs
(Cookiebot, OneTrust, etc.) structure their own gallery templates.

## Multi-tenant / on-prem hosting

GoTrust runs both a shared SaaS deployment and per-client on-prem/self-hosted deployments.
That flexibility applies differently to the two base-URL fields, because of a real GTM
constraint discovered while testing this template: the `inject_script` permission requires
a concrete, declared host at template-authoring time — a fully-wildcarded host (e.g.
`https://*/file.js`) is rejected outright by the Template Editor ("must specify both a host
and path pattern"). It is not possible for a permission to say "any host, but only this
filename."

- **`environment`** (GoTrust platform base URL) has no fixed default and is a required
  top-level field, validated only as a well-formed `https://` URL. This field is only ever
  passed as a config string into `CookieBanner.init()` — this template's own sandboxed JS
  never fetches it directly — so it isn't gated by any GTM permission and stays fully
  flexible per on-prem deployment.
- **`bundleUrl`** stays configurable (Advanced group), defaulting to the shared SaaS CDN
  (`https://cdn.gotrust.tech/gotrust-client`). Unlike `environment`, this one **is** gated:
  the `inject_script` permission is scoped to
  `https://cdn.gotrust.tech/gotrust-client/*`. On-prem clients who self-host the JS bundle
  itself (not just the backend API) on a genuinely different domain cannot use this
  gallery-submitted template as-is — that would need either a separately maintained
  (non-gallery) template variant naming their specific domain, or serving the bundle
  through a GoTrust-controlled domain pattern that could be added here as an additional
  wildcarded entry (e.g. a dedicated subdomain-per-tenant convention under a domain GoTrust
  controls).

## Fields

| Field                                                                   | Required                | Default                                   | Notes                                                                                                                                                  |
| ----------------------------------------------------------------------- | ----------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| GoTrust Domain ID (`domainId`)                                          | Yes                     | —                                         | The tenant's `domain_id` / `domain_public_id` from the GoTrust dashboard                                                                               |
| Registered Site URL (`domainUrl`)                                       | Yes                     | —                                         | The site URL registered in GoTrust; validated as `http(s)://`                                                                                          |
| GoTrust Platform Base URL (`environment`)                               | Yes                     | —                                         | No fixed default (see Multi-tenant/on-prem above); validated as `https://`                                                                             |
| Consent Mode Type (`consentModeType`)                                   | No                      | `advanced`                                | Only controls `wait_for_update` timing — does not itself block any tag from firing (see field help text)                                               |
| Debug Mode (`debugMode`)                                                | No                      | off                                       | Verbose console logging. Errors are always reported to `dataLayer` regardless of this setting                                                          |
| Wait For Update (ms) (`waitForUpdateMs`) — _Advanced group_             | No (Advanced mode only) | `500`                                     | Falls back to `500` if blank/invalid                                                                                                                   |
| GoTrust CDN Bundle Base URL (`bundleUrl`) — _Advanced group_            | No                      | `https://cdn.gotrust.tech/gotrust-client` | Only change for self-hosted/on-prem bundle hosting; validated as `https://`                                                                            |
| Banner Mount Selector (`mountSelector`) — _Advanced group_              | No                      | `#gotrust-cookie-banner`                  | If not found on the page, the SDK now falls back to `document.body` automatically (see SDK change below) — no HTML edit required for a default install |
| Restrict Default-Denied State To Regions (`regions`) — _Advanced group_ | No                      | _(empty = all regions)_                   | Comma-separated 2-letter codes. Left empty by design so the template works for Google's international audit reviewers out of the box                   |
