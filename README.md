# ConsentBit CMP — Google Tag Manager Template

A Google Tag Manager custom tag template that loads the **ConsentBit** consent
banner and sets the **Google Consent Mode v2** default (all denied) before any
measurement tag in the container fires.

The only input is your **ConsentBit Script ID**. Everything else — regulation
(GDPR / CCPA), geo detection, banner styling, translations, and cookie/script
blocking rules — is configured in your ConsentBit dashboard and loaded
automatically.

> Requires a [ConsentBit](https://consentbit.com) account. Create a site in the
> dashboard to get your Script ID and register your domain.

---

## What it does

When the tag fires it performs three steps, in order:

1. **Sets the Consent Mode v2 default to denied.** Every storage type
   (`ad_storage`, `ad_user_data`, `ad_personalization`, `analytics_storage`,
   `functionality_storage`, `personalization_storage`) is set to `denied`;
   `security_storage` stays `granted`, with `wait_for_update: 500`. Because the
   tag runs on the **Consent Initialization** trigger, this default is in place
   before any other Google tag in the container fires.
2. **Signals the banner** that the default is already set, so the banner does not
   push a duplicate consent default.
3. **Loads the ConsentBit banner** from your Script ID.

Once loaded, the banner handles the rest: showing the consent UI, blocking
non-consented scripts, and — when the visitor makes a choice — pushing the
updated consent state back to Google Consent Mode via `gtag('consent','update')`
and a `consentbit_consent_update` event on the data layer.

## Setup

1. **Add the tag.** In GTM: **Tags → New → Tag Configuration →** *ConsentBit CMP*.
2. **Paste your Script ID.** From your ConsentBit dashboard under **Install** —
   it's the ID in the middle of your install URL:
   `https://manager.consentbit.com/consentbit/{SCRIPT ID}/script.js`. Paste the
   ID only, not the whole URL.
3. **Set the trigger** to **Consent Initialization - All Pages**.
4. **Save**, then **Submit / Publish** the container.

That's the whole setup. Do not also paste the ConsentBit `<script>` snippet in
your site's `<head>` — the GTM tag replaces it.

### Firing your own tags on consent changes (optional)

When a visitor makes or changes a choice, the banner pushes a
`consentbit_consent_update` event to the data layer. To fire your own GTM tags
from it:

- **Trigger:** Custom Event → event name `consentbit_consent_update`.
- **Data Layer Variables** you can read from that event:
  - `consentbit_analytics` (boolean)
  - `consentbit_marketing` (boolean)
  - `consentbit_preferences` (boolean)
  - `consentbit_regulation` (`gdpr` or `ccpa`)
  - `consentbit_source` (where the choice came from)

## Permissions

This template requests four permissions:

| Permission | Purpose |
| --- | --- |
| Logs to console | Debug messages in GTM Preview only |
| Injects scripts (`https://manager.consentbit.com/consentbit/*`) | Loads the banner |
| Accesses consent state | Sets the Consent Mode v2 default |
| Accesses global variables (`__cbConsentDefaultSet`) | Coordinates with the banner to avoid a duplicate default |

## Notes & limitations

- **Registered domain required.** The banner only loads on the domain registered
  for your Script ID in the ConsentBit dashboard (plus Webflow staging domains).
  On any other origin the request is rejected — this is a security feature, not a
  bug. When testing in GTM Preview, preview the site whose domain matches the
  Script ID.
- **Custom data layer name.** The banner reads and writes the standard
  `dataLayer`. If your container uses a renamed data layer, the Consent Mode
  default still works, but the banner's consent-update events will be on
  `dataLayer` rather than your custom name.

## Support

- Product & dashboard: https://consentbit.com
- Documentation and help: your ConsentBit dashboard

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE).
