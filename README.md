# GoTrust CMP — Google Tag Manager Template

Install the [GoTrust](https://gotrust.tech) consent banner through Google Tag Manager with
Google Consent Mode v2 built in.

**Full documentation:** <https://gotrust.tech/docs/gtm-template>

## What it does

- Sets the Consent Mode **default** before any other tag runs. Visitors in the EEA, UK and
  Switzerland always start denied; everywhere else uses the defaults you choose in the tag
  (granted unless you change them), with optional per-region overrides.
- **Restores returning visitors' choices** immediately, before the banner loads.
- Loads the GoTrust banner and passes every consent choice to your Google tags with
  `updateConsentState`.
- Sets `ads_data_redaction` and `url_passthrough` (both configurable), and GoTrust's
  Google developer ID.

## Quick install

1. In GTM, go to **Templates → Tag Templates → Search Gallery**, find **GoTrust – Consent
   Mode & CMP Loader** and add it to your workspace.
2. Create a tag from the template and fill in **GoTrust Domain ID**, **Registered Site URL**
   and **GoTrust Platform Base URL**, all shown in your GoTrust dashboard under
   **Cookie Consent Management → your domain → Consent Code**.
3. Set the trigger to **Consent Initialization - All Pages**.
4. Preview and check that the banner appears and the Consent tab shows the default and then
   an update when you make a choice.
5. Publish the container.

Use this template **or** the HTML embed code from your dashboard, not both.

## Support

Email [support@gotrust.tech](mailto:support@gotrust.tech) or open an issue in this
repository.

## License

[Apache 2.0](LICENSE)
