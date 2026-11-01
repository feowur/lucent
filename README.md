# lucent updates (feowur)

Sparkle 2.10.1 handles checking, notes, download progress, cancellation, Ed25519 verification, installation and relaunch. `lucent/UpdateChecker.swift` owns the single controller. Settings uses the existing button; Sparkle supplies its supported native update dialogs. The restart overlay is visual only and never postpones installation. Relaunch uses lucent's normal window animation and version label.

## Signing

Development builds are ad-hoc signed (`Scripts/build.sh Release`). Ad-hoc signing does not certify a publisher identity. The publisher is feowur. Sparkle separately verifies archives against `SUPublicEDKey` in `lucent/Info.plist`. The corresponding private key is in the login Keychain, service `https://sparkle-project.org`, account `lucent-updates`. Never commit/export it into a repository or app. Back it up securely outside Git before distributing the first build; loss prevents signing future updates for existing clients.

## Private testing

Both `feowur/lucent` (source) and `feowur/lucent-updates` (distribution) are private. The production feed URL is `https://github.com/feowur/lucent-updates/releases/latest/download/appcast.xml`. It will fail for unauthenticated clients while private. No token is embedded, and a 404 is an error, not an up-to-date result.

`Scripts/test-updater.py` makes isolated copies of the Release app with a distinct bundle ID, serves signed archives on loopback, and opens the older copy. Use its existing Check for Updates button and follow Sparkle's Download / Install and Relaunch. The server prints instructions; Ctrl-C removes only the local test server. Test artifacts stay ignored under `build/UpdaterTest`. This exercises the production updater with no private GitHub credentials.

## First distribution

1. Keep the private source repository private. Make update assets anonymously accessible when you are ready (e.g. make only `lucent-updates` public). No visibility change is performed by these scripts.
2. Increase both `CFBundleVersion` (monotonically) and `CFBundleShortVersionString` in `lucent/Info.plist`; build Release. Keep the bundle identifier, Ed25519 public key and app name stable.
3. Run `Scripts/prepare-update.sh /absolute/path/lucent.app v0.2 Updates/notes.html`. It verifies the app signature, packages with `ditto`, signs with the Keychain key, generates an appcast and creates a **draft** release in the distribution repository. Source and credentials are never included.
4. Review the draft's appcast, archive, and notes. Publish it manually only after testing. The stable `latest/download/appcast.xml` URL resolves to the published release. Until downloads are public, use local testing. Keep prior appcast entries when preparing subsequent releases (download the previous appcast into the output directory first).

The script intentionally does not install an app or publish a release automatically. No automatic downloads/checks are enabled by default. For broad installation on other Macs, Apple Developer ID/notarization remains a separate Gatekeeper requirement; ad-hoc signing plus Sparkle archive signing does not bypass it. Updates can change TCC trust for ad-hoc builds; lucent does not reset permissions.
