# oc-mobile-config

Public configuration files for the OC mobile apps. Hosted as GitHub raw URLs
so the apps can fetch the latest values without shipping a new build.

## `version-policy.json` — minimum-version gate

The mobile app fetches this file on every cold start and after each
foreground transition. If the user's installed version is below
`minimumVersion`, the **forced-update modal** is shown until they update.

### Schema

```json
{
  "minimumVersion": "1.1.25",
  "message": "Please update OC Onboarding to continue.",
  "iosStoreUrl": "https://apps.apple.com/app/oc-kyc/id6751334930",
  "androidStoreUrl": "https://play.google.com/store/apps/details?id=com.anonymous.scbb"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `minimumVersion` | ✅ | Lowest version that's allowed to keep using the app. Bump after each release. |
| `message` | ❌ | Body text in the forced-update modal. |
| `iosStoreUrl` | ❌ | App Store deep link for the "Open App Store" button. |
| `androidStoreUrl` | ❌ | Play Store deep link for the "Open Play Store" button. |

### How to release a new version

1. Push the new build to App Store + Play Store as usual.
2. Edit `version-policy.json` here and bump `minimumVersion`.
3. Commit + push. Within ~5 minutes (GitHub raw cache TTL — the app
   cache-busts so most users see it within seconds of a cold start),
   every user on the previous version sees the forced-update modal.

### Public URL the app uses

```
https://raw.githubusercontent.com/<your-github-username>/oc-mobile-config/main/version-policy.json
```
