# GoTrust CMP — Google Tag Manager Installation Guide

This guide walks you through installing the **GoTrust – Consent Mode & CMP Loader**
template in your own Google Tag Manager (GTM) container. It covers the same ground as the
direct HTML embed instructions in your GoTrust dashboard (**Cookie Consent Management →
your domain → Consent Code** step), adapted for a GTM-managed install. Use this guide
instead of the HTML snippet if your site's tags are managed through GTM.

---

## What this installs

Adding this tag to your container does two things automatically, in order, every time a
page loads:

1. Sets Google Consent Mode's default-denied baseline (`ad_storage`, `analytics_storage`,
   `functionality_storage`, `personalization_storage`, `ad_user_data`,
   `ad_personalization` all denied; `security_storage` always granted) before any other
   tag in your container evaluates consent.
2. Loads the GoTrust cookie banner and mounts it on the page. From there, the banner
   behaves exactly as it does everywhere else — it shows the consent UI, remembers
   returning visitors' choices, and pushes consent updates that GTM and your other tags
   observe automatically.

You do not need to also add the HTML `<script>` tags from the Consent Code step if you
install via this template — use one method or the other, not both.

---

## Prerequisites

Before you start, have these on hand:

- **Edit access to your GTM container.**
- **Your GoTrust Domain ID.** In the GoTrust dashboard, go to **Cookie Consent
  Management → [your domain] → Consent Code** step — your ID is shown under
  "Configuration Details" as **Config ID**. (The same value the HTML embed snippet uses
  as `domain_id`.)
- **Your registered site URL and GoTrust platform base URL**, also shown on that same
  Consent Code step under "Configuration Details" as **Domain** and **Environment**.
- Ability to publish a new version of your GTM container (or a teammate who can).

---

## Installation Steps

### Step 1 — Add the template to your container

If the template is not yet in your container:

1. In GTM, go to **Templates → Tag Templates → Search Gallery**.
2. Search for "GoTrust" and select the **GoTrust – Consent Mode & CMP Loader** template.
3. Click **Add to workspace** → **Add**.

(If your GoTrust contact gave you the template directly instead of pointing you to the
gallery, use **Templates → New → Import** and select the provided `.tpl` file instead.)

### Step 2 — Create a new tag from the template

1. Go to **Tags → New**.
2. Name it something identifiable, e.g. `GoTrust – Consent Mode & CMP Loader`.
3. Under **Tag Configuration**, choose the GoTrust template you just added.

### Step 3 — Fill in the configuration fields

| Field                     | Required              | What to enter                                                              |
| ------------------------- | --------------------- | -------------------------------------------------------------------------- |
| GoTrust Domain ID         | Yes                   | The **Config ID** from your GoTrust dashboard's Consent Code step          |
| Registered Site URL       | Yes                   | The **Domain** value from the same page (e.g. `https://www.example.com`)   |
| GoTrust Platform Base URL | Yes                   | The **Environment** value from the same page                               |
| Consent Mode Type         | No — default Advanced | Leave as **Advanced** unless GoTrust support tells you otherwise           |
| Debug Mode                | No — default off      | Turn on temporarily while testing (see Step 6); turn off before publishing |

Everything else is under **Advanced Settings** (collapsed by default) and only needs
attention in specific cases:

| Advanced field                           | When to change it                                                                                                                                                              |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Wait For Update (ms)                     | Leave at `500` unless GoTrust support advises otherwise                                                                                                                        |
| GoTrust CDN Bundle Base URL              | Only if you're on a self-hosted/on-prem GoTrust deployment that serves the script bundle from your own infrastructure — leave at the default otherwise                         |
| Banner Mount Selector                    | Leave at the default `#gotrust-cookie-banner` — if that element isn't present on your page, the banner mounts itself automatically, so you don't need to add any HTML for this |
| Restrict Default-Denied State To Regions | **Leave empty** unless you have a specific reason to restrict Consent Mode defaults to certain countries — empty means it correctly applies to every visitor everywhere        |

### Step 4 — Attach the Consent Initialization trigger

This is the step most likely to be missed, and the tag will not work correctly without
it:

1. Under **Triggering**, click the trigger selector.
2. Choose **Consent Initialization - All Pages**. If it doesn't exist yet in your
   container, create it: **Triggers → New → Consent Initialization → All Pages**.
3. Save the tag.
4. In **Tag sequencing / firing priority** (or the container's tag settings), make sure
   this tag has the **highest priority** among your Consent Initialization tags, so it
   runs before any other tag that reads consent state.

### Step 5 — Save, Preview, and test

1. Click **Save**, then **Preview**.
2. Open your site in the Preview/Debug window.
3. Confirm:
   - The GoTrust banner appears.
   - In the GTM debug panel, your tag fired on the **Consent Initialization** step, before
     any other tags.
   - In the browser console (if Debug Mode is on), you see `[GoTrust CMP]` log lines with
     no errors.
   - Accepting/rejecting in the banner updates consent state as expected for your other
     tags (e.g. GA4, Google Ads).
4. If something fails, check the `dataLayer` in Preview mode for a `gotrust_template_error`
   event — it includes `gotrust_error_stage` and `gotrust_error_message` fields that say
   exactly what went wrong (see Troubleshooting below).
5. Turn **Debug Mode** back off once you've confirmed everything works.

### Step 6 — Publish

Once Preview mode confirms everything works, **Submit** and publish your container
version as usual.

---

## Manual Cookie Preferences Trigger (Optional)

If you don't want to rely on the banner's own floating settings button, you can open
cookie preferences from any button or link on your site — this works identically whether
the banner was installed via this GTM template or the direct HTML embed, since it's the
same underlying banner:

```html
<script>
  function openCookiePreferences() {
    if (window.showGoTrustCookiePreferences) {
      window.showGoTrustCookiePreferences(true);
    } else {
      console.warn('GoTrust Cookie Banner not loaded yet.');
    }
  }
</script>

<button onclick="openCookiePreferences()" type="button">Cookie Preferences</button>
```

This can be added directly to your site's HTML, or as its own GTM Custom HTML tag if you'd
rather manage it through GTM as well.

---

## Important Notes

- **Test on staging before production**, same as with any tag change.
- **The banner respects user preferences automatically** — you don't need to write any
  logic to persist or re-apply consent choices; that's handled for you.
- **Leave "Restrict Default-Denied State To Regions" empty** unless you have a specific
  reason not to — this ensures Consent Mode defaults correctly apply worldwide, not just
  in the EEA.
- **The Consent Initialization trigger is required.** Attaching this tag to any other
  trigger (e.g. "All Pages" / Page View) means Consent Mode defaults are set too late,
  after other tags may have already fired.
- **No HTML changes are required** for a standard install — the banner mounts itself even
  if the page doesn't have a `#gotrust-cookie-banner` element.
- **Errors are always reported** to `dataLayer` as a `gotrust_template_error` event,
  whether or not Debug Mode is on — check there first if something isn't working.
- Appearance, categories, and languages are all customized from your GoTrust dashboard,
  the same as with the HTML embed method — this template doesn't change how that's
  configured.

---

## Troubleshooting

| Symptom                                                                        | Likely cause                                                                                            | Fix                                                                                                                                    |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Banner never appears                                                           | Tag isn't firing, or fired on the wrong trigger                                                         | Confirm the tag is attached to **Consent Initialization - All Pages** and check GTM's Preview debug panel to confirm it actually fired |
| Banner appears but Consent Mode signals look wrong in GA4/Ads                  | Tag firing priority                                                                                     | Make sure this tag has the highest priority among Consent Initialization tags, so it runs before other tags read consent state         |
| Nothing happens, no console errors                                             | Debug Mode is off                                                                                       | Temporarily enable **Debug Mode** on the tag and re-run Preview mode to see `[GoTrust CMP]` diagnostic logs                            |
| `gotrust_template_error` with stage `config`                                   | A required field is missing or malformed (Domain ID, Registered Site URL, or GoTrust Platform Base URL) | Re-check Step 3's required fields against your dashboard's Consent Code page                                                           |
| `gotrust_template_error` with stage `permission_denied`                        | GTM's Permissions tab wasn't configured/approved for this container                                     | Have whoever manages the container's template permissions check the Permissions tab on the tag template                                |
| `gotrust_template_error` with stage `shim_load_failed` or `bundle_load_failed` | Network/CDN issue, or an incorrect **GoTrust CDN Bundle Base URL** override in Advanced Settings        | Confirm you haven't changed the Advanced "GoTrust CDN Bundle Base URL" unless your deployment specifically requires it                 |
| `gotrust_template_error` with stage `sdk_api_missing`                          | The bundle loaded but its public API didn't match what the template expects                             | Contact GoTrust support — this usually means a version mismatch and needs to be resolved on the GoTrust side                           |

---

## Need Help?

If you run into any issues during implementation, contact
[support@gotrust.tech](mailto:support@gotrust.tech).
