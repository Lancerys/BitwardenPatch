# HyperOS unlock-first experiment (A)

**Branch:** `experiment/hyperos-unlock-first` — independent of the B warm-cache branch and main.

## Goal

On Xiaomi HyperOS, when the Bitwarden vault is locked, show a selectable
**Unlock Bitwarden** entry instead of falling through to "No passkeys available".
After tapping it, unlock with Bitwarden biometrics/PIN/master password, then
select a matching vault login to finish the *original* WebAuthn assertion.

There is **no in-memory passkey cache** and no persistent metadata index.

## How this differs from the warm-cache B version

- B: after one unlocked use, stores metadata in process memory for 10 minutes.
- A: while locked, offers one generic, non-autoselectable public-key entry per
  incoming passkey option. It does not claim to know which passkeys exist.
- Once chosen, it routes through the existing Bitwarden
  `CredentialProviderActivity -> MainActivity -> vault unlock -> Fido2Assertion`
  pipeline, using a sentinel to indicate that the real cipher is still to be
  selected.
- After unlock, the built-in vault login list is filtered by relying-party ID
  and the request's optional allowCredentials values.
- The user taps the real matching vault item; existing origin validation and
  user verification still run before SDK assertion signing.
- Password-only requests, normal unlocked requests, and other Android devices
  retain upstream behavior.

## Important limitations / security

- **Experimental only.** The single generic "Unlock Bitwarden" entry is
  *not* a real passkey. It may appear on sites where the vault has no passkeys.
  In that case an error will be shown after unlocking.
- Do not accept an automatic signature from the generic entry: it is explicitly
  non-autoselectable and the user must choose an actual vault item after unlock.
- No vault data or passkey private keys are cached by this patch. Bitwarden's
  existing origin validation and encrypted SDK storage remain unchanged.
- Passkeys stored with the same relying-party ID inside a single cipher may
  need additional selection UX; this prototype selects the matching vault item
  and lets Bitwarden's SDK perform final allowed-credential validation.
- The Xiaomi picker and end-to-end behavior remain **unverified** until tested
  on an actual device; compilation alone is insufficient.
- This APK uses the same application ID as the installed Bitwarden build;
  it is not side-by-side installable. Ensure vault sync/backup and matching
  signing certificates. Do not uninstall your working B APK to work around
  a version/signature error.

## Test sequence

1. Download the `bitwarden-hyperos-unlock-first-a` Actions artifact. Unzip
   the APK and check its versionCode exceeds the installed 11042 B build.
2. Install by normal update (not uninstall). Confirm Bitwarden remains the
   selected password/passkey provider in HyperOS settings.
3. With a disposable `webauthn.io` test passkey already stored, manually
   lock Bitwarden. **Do not open and unlock Bitwarden first.**
4. In the browser, press Authenticate. The picker should present
   "Unlock Bitwarden". Select it and use your normal unlock method.
5. The Bitwarden login list should contain only matching test-site entries.
   Tap the appropriate one and confirm authentication completes.
6. Repeat after force-stopping Bitwarden from app settings and after restarting
   the phone. The A patch has no warm-cache requirement.
7. Negative test: try a site without a stored passkey. The generic unlock entry
   may appear, but no unrelated credential should be offered or signed.
8. Only after the test-site flow works, optionally test a second WebAuthn site
   or a real app that already has a passkey. Avoid testing with valuable accounts
   until the behavior is predictable.

Record exactly whether the failure is at (a) generic entry visibility,
(b) opening Bitwarden, (c) unlocking, (d) list filtering/selection,
or (e) completion of the original site/app login.

## Restore and versioning

- Original working B artifact used versionCode **11042** (verified from
  successful Actions run 39726485457).
- A workflow chooses a versionCode greater than 11042 and above the current
  stable main build.
- Reinstalling an older B APK over newer A may be rejected as a downgrade.
  Do not uninstall to bypass that; build a subsequent B-compatible update
  with a higher versionCode if reverting is necessary.
- CI uploads an artifact only; it does **not** publish a GitHub Release.
