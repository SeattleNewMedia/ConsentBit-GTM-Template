# ConsentBit — Google Tag Manager Custom Template

A GTM tag template that loads the ConsentBit consent banner and sets the Google
Consent Mode v2 default (all denied) before any measurement tag in the container
fires.

The **only** input is the **Script ID** — the `cdnScriptId` from the site's
install snippet. Everything else (regulation, geo resolution, styling,
translations, script-blocking rules) is resolved server-side from that ID by
[`src/handlers/cdnNm.js`](../src/handlers/cdnNm.js), exactly as it is for the
hand-pasted `<script>` install.

---

## What the template does

When the tag fires it does three things, in order:

| Step | Why |
| --- | --- |
| `setDefaultConsentState(…all denied…)` | Runs on the **Consent Initialization** trigger, which GTM guarantees fires before every other tag. Denies every storage type (except `security_storage`) with `wait_for_update: 500`. The banner's own default (`cdnNm.js:3130`) can only run once the script has downloaded — too late for tags already queued in the same container. |
| `setInWindow('__cbConsentDefaultSet', true)` | Suppresses the banner's duplicate default push. The banner checks this flag at `cdnNm.js:3128`. |
| `injectScript(…/consentbit/{id}/script.js)` | Loads the banner. From here the banner does everything else. |

The consent default is **hardcoded** — there are no configuration fields for it.
The template is deliberately a single-input tag.

### What the template intentionally does NOT do

- **Consent updates.** The banner pushes these itself when the visitor chooses:
  - `gtag('consent','update', …)` — [`cdnNm.js:2117`](../src/handlers/cdnNm.js#L2117)
  - a named `consentbit_consent_update` dataLayer event — [`cdnNm.js:1469`](../src/handlers/cdnNm.js#L1469)

  Both go through `window.dataLayer`, so GTM picks them up with no template
  involvement.
- **`ads_data_redaction` / `url_passthrough`.** The banner sets these itself in
  `setConsentModeFlags()` ([`cdnNm.js:1451`](../src/handlers/cdnNm.js#L1451)).
  They were previously set in the template too, but that required a
  `write_data_layer` permission and added no real benefit, so they were dropped.

## Permissions

Four, all required:

| Permission | Used by |
| --- | --- |
| `logging` | debug logging |
| `inject_script` (`https://manager.consentbit.com/consentbit/*`) | loading the banner |
| `access_consent` (all consent types) | `setDefaultConsentState` |
| `access_globals` (`__cbConsentDefaultSet`) | `setInWindow` |

## Container setup (what the customer does)

1. **Tag** → new tag → *ConsentBit CMP* → paste the Script ID.
2. **Trigger** → `Consent Initialization - All Pages`. Nothing else.
3. For tags that should fire on a *change* of consent, add a **Custom Event**
   trigger on `consentbit_consent_update` and Data Layer Variables for
   `consentbit_analytics`, `consentbit_marketing`, `consentbit_preferences`,
   `consentbit_regulation`, `consentbit_source`.

## Two things to verify before submitting

**1. The domain guard must not reject GTM-injected loads.**
`cdnNm.js` blocks the request when `Origin`/`Referer` does not match the site's
registered domain (`cdnNm.js:56–104`). A GTM `injectScript` produces a normal
`<script src>` from the page, so the browser sends `Referer: https://<the page>`
and the check passes. Confirm on a real container before publishing. Note that
Google's reviewers test on their **own** page, where `script.js` will return
**403** and the banner will not render — make the README and description clear
that a ConsentBit account and a registered domain are required, so this reads as
"needs setup," not "broken."

**2. A renamed dataLayer breaks consent updates.**
The banner pushes to `window.dataLayer` literally. A container installed with a
custom dataLayer name will receive the template's default (GTM handles that) but
**not** the banner's updates. Either document this as unsupported, or read the
container's dataLayer name in the template and pass it to the banner as a query
parameter that `cdnNm.js` bakes into the loader.

## Publishing to the Community Template Gallery

The gallery requires a **dedicated public GitHub repo** with these files at the
repository root — this subfolder will not work as-is:

```
template.tpl        ← from this folder (see the export note below)
metadata.yaml       ← from this folder, with a real commit SHA
LICENSE             ← Apache 2.0
README.md
```

Steps:

1. Create a public repo, e.g. `consentbit/gtm-consentbit-cmp`.
2. In GTM: **Templates → New → ⋮ → Import** the `.tpl`, add the brand icon on the
   **Info** tab, run the built-in tests, then **⋮ → Export**. Commit the
   **exported** file — GTM normalises formatting and rewrites the placeholder
   `id`. Do not commit the hand-written `.tpl` for submission.
3. Push, then fill `metadata.yaml` `sha:` with the full 40-char SHA of that
   commit and push again.
4. GTM → Templates → your template → **Submit to Gallery**, and authorise the
   GitHub account. Google requires the account to have a verified email and 2FA,
   and the repo to be public.
5. Review typically takes a few business days.

### Before submitting

- Replace `"id": "cvt_temp_public_id"` in `___INFO___` — the GTM **Export** in
  step 2 does this automatically.
- Add a brand icon on the **Info** tab (square, ideally 128×128). Re-importing
  the `.tpl` clears it, so add it *after* the final import and before export.
- `"categories"` are `UTILITY` and `PERSONALIZATION` — both valid. An
  unrecognised value fails validation on import.
- To ship an update later, **append** a new `- sha: … / changeNotes: …` block to
  `metadata.yaml`; never edit an existing entry. The gallery treats the list as
  append-only version history.

## Local testing

Import `template.tpl` in GTM's template editor and press **Run Tests** — the
`___TESTS___` block covers URL construction, the all-denied consent default, and
the empty-Script-ID path.
