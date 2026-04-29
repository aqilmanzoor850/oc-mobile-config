# Universal Links / App Links — Singpass FAPI 2.0 redirect

These two files **must** be served from the **root of the domain you list in
the app's `associatedDomains` (iOS) and `intentFilters` (Android) configs**.

For this app, that domain is **`app.one-constellation.com`**.

## Files

| File | Required URL | Notes |
|------|--------------|-------|
| `apple-app-site-association` | `https://app.one-constellation.com/.well-known/apple-app-site-association` | **No file extension.** Must be served as `application/json` over HTTPS, no redirects. |
| `assetlinks.json` | `https://app.one-constellation.com/.well-known/assetlinks.json` | Served as `application/json` over HTTPS. |

## Before you go live

### 1. AASA — replace `REPLACE_WITH_TEAM_ID`

Find your Apple Team ID at https://developer.apple.com/account → Membership.
It's a 10-character string like `ABCDE12345`.

Replace **both** occurrences of `REPLACE_WITH_TEAM_ID` in
`apple-app-site-association` with that value, e.g.
`ABCDE12345.com.anonymous.scbb`.

### 2. assetlinks.json — verify SHA-256

The current value is the SHA-256 of the EAS-managed Android keystore
(SHA1: `E2:E8:6B:A9:8A:64:5F:02:75:8E:F3:C0:C2:73:9E:63:06:57:A1:E8`).

If you ever rotate the keystore, regenerate the SHA-256 with:

```bash
keytool -list -v -keystore android/app/release.jks -storepass <password>
```

…and paste the new SHA-256 value into `sha256_cert_fingerprints`.

For Play Protect–enrolled apps, you may also need to add Google Play's
**App Signing** SHA-256 (Play Console → App integrity → App signing).

### 3. Hosting

These files must be served from `app.one-constellation.com` — that's the
domain you registered with Singpass and listed as `applinks:` /
`intentFilters` in `app.json`.

Whoever runs DNS for `one-constellation.com` should:

1. Point `app.one-constellation.com` at a static-file host (S3, CloudFront,
   GitHub Pages with custom domain, Vercel, Netlify — anything that serves
   HTTPS without redirects).
2. Upload these two files preserving the `.well-known/` path.
3. Verify:
   ```bash
   curl -I https://app.one-constellation.com/.well-known/apple-app-site-association
   # Should return: HTTP/2 200, content-type: application/json
   curl -I https://app.one-constellation.com/.well-known/assetlinks.json
   # Same.
   ```

### 4. Validate from Apple / Google's side

- **iOS AASA validator**: https://branch.io/resources/aasa-validator/
  → enter `app.one-constellation.com` → all checks should be green.
- **Android App Links assistant**: paste this URL in a browser:
  ```
  https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https%3A%2F%2Fapp.one-constellation.com&relation=delegate_permission%2Fcommon.handle_all_urls
  ```
  Should return your `assetlinks.json` content.

### 5. Register with Singpass

In your Singpass app config (developer portal):

- **Redirect URL** = `https://app.one-constellation.com/singpass/callback`
- **Redirect URL** = `https://app.one-constellation.com/corppass/callback`
  (if you have a Corppass app too)

### 6. Rebuild the app

After the AASA + assetlinks.json are reachable AND Singpass has the new
redirect URLs registered, rebuild the app — Universal Links / App Links
are evaluated at install time, so a fresh install picks up the
associatedDomains entitlement and intent-filter autoVerify.

```bash
npx expo prebuild --clean
eas build -p ios --profile production
eas build -p android --profile production
```
