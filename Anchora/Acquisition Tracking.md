
Four valid entry paths this feature supports, with what each produces:

**1. Tagged campaign link → marketing site → signup**

```
https://anchora.com.ng/?utm_source=google&utm_medium=cpc&utm_campaign=spring-launch&utm_content=ad-variant-a
```

`AcquisitionTracker` captures this into the `anchora_acquisition` cookie on first load. Clicking "Get started free" builds:

```
https://app.anchora.com.ng/signup?acq_source=google&acq_medium=cpc&acq_campaign=spring-launch&acq_content=ad-variant-a&acq_landing_page=%2F%3Futm_source%3Dgoogle...&acq_first_touch_at=2026-08-18T09%3A12%3A00.000Z&ph_distinct_id=...
```

`/signup` reads those `acq_*` params, and the register payload carries `acquisitionSource: "google"`, `acquisitionMedium: "cpc"`, `acquisitionCampaign: "spring-launch"`, `acquisitionContent: "ad-variant-a"`.

**2. Untagged link → marketing site → signup (referrer fallback)** Someone clicks a plain link from another site, e.g. `https://someblog.com/anchora-review` → `https://anchora.com.ng/`. No UTM params, so `captureAcquisition()` falls back to `document.referrer`:

```
acquisitionSource: "https://someblog.com/anchora-review"
acquisitionMedium: null
acquisitionCampaign: null
```

**3. Paid link straight to the app (direct-entry capture)** A campaign that skips the marketing site entirely:

```
https://app.anchora.com.ng/signup?utm_source=facebook&utm_medium=paid-social&utm_campaign=retarget-q3
```

No `acq_*` handoff params present, so `/signup`'s own fallback capture reads the `utm_*` params directly off its own URL — same result as path 1, without ever touching `anchora-website`.

**4. Direct, untagged visit to `/signup`** Someone types the URL or uses a bookmark, no params, no referrer. All six fields end up `NULL` on the `User` row — no error, matches the acceptance criteria we verified earlier.

In all four cases, first-touch wins: once the cookie is set (on either domain), revisiting via a different link later doesn't overwrite it — `captureAcquisition()` returns the existing cookie value unchanged.