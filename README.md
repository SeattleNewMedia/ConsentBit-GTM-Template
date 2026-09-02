# ConsentBit CMP — Google Tag Manager Template

A Google Tag Manager custom tag template that loads the **ConsentBit** consent
banner and publishes the **Google Consent Mode v2** default — per region, or all
denied worldwide — before any measurement tag in the container fires.

The only required input is your **ConsentBit Script ID**. Everything else —
regulation (GDPR / CCPA), geo detection, banner styling, translations, and
cookie/script blocking rules — is configured in your ConsentBit dashboard and
loaded automatically.

> Requires a [ConsentBit](https://consentbit.com) account. Create a site in the
> dashboard to get your Script ID and register your domain.

---

## What it does

When the tag fires it performs four steps, in order:

1. **Sets the ConsentBit developer ID and the two consent-mode settings** in one
   `gtagSet` call — `developer_id.dN2Q3Yj` (the ID Google issued to ConsentBit, so
   it can identify the CMP behind these signals), plus `ads_data_redaction` and
   `url_passthrough`. The ConsentBit script sets those two itself on a direct
   install; this keeps the GTM install identical.
2. **Sets the Consent Mode v2 defaults**, one `gtag('consent','default')` command
   per row of the region table (see below). With no rows configured that is a
   single worldwide command denying every storage type except `security_storage`,
   with `wait_for_update: 500`. Because the tag runs on the **Consent
   Initialization** trigger, these land before any other Google tag in the
   container fires.
3. **Signals the banner** that a default is already published, so it does not
   overwrite the region-scoped state with an unscoped one of its own.
4. **Loads the ConsentBit banner** from your Script ID.

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
3. **Review the default consent state** (optional — see below).
4. **Set the trigger** to **Consent Initialization - All Pages**.
5. **Save**, then **Submit / Publish** the container.

That's the whole setup. Do not also paste the ConsentBit `<script>` snippet in
your site's `<head>` — the GTM tag replaces it.

### Consent Mode v2 defaults by region

The default is the consent state that applies **before** the visitor answers the
banner. Each row of the table becomes one `gtag('consent','default')` command:

| Regions | Analytics | Advertising | … | Strictly necessary |
| --- | --- | --- | --- | --- |
| `All` | Denied | Denied | Denied | Granted |
| `US-CA,US-VA` | Granted | Granted | Granted | Granted |

- **Regions** takes comma-separated [ISO 3166-2](https://en.wikipedia.org/wiki/ISO_3166-2)
  codes — countries (`GB`, `DE`) or subdivisions (`US-CA`). Enter `All` for the
  row that covers every visitor no other row matched.
- The **most specific** matching region wins: a `US-CA` row overrides a `US` row
  for visitors in California.
- **Leave the table empty** and everything except *Strictly necessary* is denied
  worldwide, which is what GDPR requires. That is also what an existing tag does
  until you edit and save it.
- For an opt-out regime such as the US state privacy laws, add a row naming those
  regions and set its categories to **Granted** — the banner still applies the
  visitor's opt-out when they make one.

The table controls the *default* only. Which banner a visitor sees, and which law
applies to them, is decided by your ConsentBit dashboard, and the visitor's real
choice is applied afterwards by the banner.

### Other settings

- **Wait for update** — how long Google's tags wait for the visitor's choice
  before running with the default. Defaults to 500ms, matching the banner.
- **Redact ads data while consent is denied** (`ads_data_redaction`) — on by
  default.
- **Pass ad click and session information in URLs** (`url_passthrough`) — on by
  default. Both match what the ConsentBit script does on a direct install.

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

This template requests five permissions:

| Permission | Purpose |
| --- | --- |
| Logs to console | Debug messages in GTM Preview only |
| Injects scripts (`https://manager.consentbit.com/consentbit/*`) | Loads the banner |
| Accesses consent state | Sets the Consent Mode v2 defaults |
| Accesses global variables (`__cbConsentDefaultSet`, `__cbGtmInstall`) | Tells the banner a default is already published, and that this is a GTM install |
| Writes data layer (`developer_id.…`, `ads_data_redaction`, `url_passthrough`) | Sets the developer ID and the two consent-mode settings |

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
