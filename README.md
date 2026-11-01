# lucent update distribution

This repository holds public-key/configuration metadata, appcasts and release notes. Publishing commands below run from the **source checkout**, not this repository. ZIP archives belong in GitHub Release assets. Both repositories and the v0.1 draft remain private/unpublished at this handoff.

# Updating lucent — ad-hoc distribution

## What is configured

- Source: `feowur/lucent`; downloads: `feowur/lucent-updates`. Both are currently private. This runbook changes neither visibility nor release publication automatically.
- Sparkle **2.10.1**, pinned in Xcode's `Package.resolved`. One `SPUStandardUpdaterController` in `lucent/UpdateChecker.swift` owns the native checking, release notes, progress, cancellation, installation and relaunch flow.
- Feed: `https://github.com/feowur/lucent-updates/releases/latest/download/appcast.xml` (`SUFeedURL`). User-initiated checks only.
- `SUPublicEDKey` authenticates Ed25519-signed archives; `SUVerifyUpdateBeforeExtraction` is enabled. Sparkle's standard installer performs replacement/relaunch. The lucent restart overlay never delays installation.
- Ad-hoc code signing remains the chosen route. It does not give a verified developer identity or notarization. Users may need macOS Privacy & Security → Open Anyway, and updates may require input permissions to be approved again. No Apple Developer membership is part of these instructions.

## What belongs where

| Location | Include | Exclude |
| --- | --- | --- |
| Source repository | `lucent/`, `lucent.xcodeproj/`, pinned `Package.resolved`, `Scripts/`, `Tests/`, `Updates/`, `README.md`, MIT `LICENSE`, `.gitignore`, original `icon.png`, empty `UserData/.gitkeep` | Builds/apps/zips/dmgs/dSYMs, logs/Validation, Checkpoints, Research, imported user media, preferences, Obsidian, `.DS_Store`, Xcode user state, secrets/private keys/tokens/certificates |
| Distribution repository Git tree | Public key, appcast, notes, configuration metadata and publishing instructions | App source, user data, credentials, private signing key, intermediate builds |
| Distribution GitHub Release assets | `appcast.xml`, signed `lucent-VERSION.zip`, optional release notes file | Source checkout, `.git`, dSYM, debug builds, test app, keys/tokens |
| Inside `lucent.app` | Executable, normal resources/assets, embedded Sparkle framework, Sparkle license notices, public key/feed URL in Info.plist | GitHub tokens, Ed25519 private key, publishing scripts, test fixtures |

The public update signing key is intentionally safe to include in both repositories and the app. The private key remains in the login Keychain under account `lucent-updates`. GitHub authentication is only for the publisher's `gh` CLI. Do not add Authorization headers or a GitHub token to the app to make private assets work.

## Seven remaining steps before a general public release

1. **Resolve media portability.** The approved app currently embeds the development workspace path for background storage. Move/migrate media to the user's Application Support directory in a separately authorized app change, then build/test that candidate. This preparation pass documents the issue; it does not silently change runtime storage.
2. **Back up the existing signing key securely.** Use Sparkle's `generate_keys --account lucent-updates -x /secure/outside-repositories/private-key` only to an encrypted/private backup location, with access restricted to you. Never generate a replacement key for an app already distributed. Keep this backup and recovery information outside Git and the app.
3. **Decide repository visibility.** Anonymous clients need public update assets. When you authorize it, make only `lucent-updates` public (or provide another public GitHub-hosted distribution endpoint). Opening the source `lucent` repository is optional and separate. No visibility change has been performed.
4. **Prepare the first public candidate.** Current draft `v0.1` is the initial-install baseline, not an update for clients already on build 3. For the first actual update use, for example, short version `0.1.1` and build `4` in `lucent/Info.plist`. Increase the numeric build every release; keep app name, bundle ID and public key unchanged.
5. **Build and prepare a draft**, using the commands below. The script checks the app signature and matching Keychain public key, packages with `ditto`, generates the signed enclosure/appcast, and creates or refreshes a private/unpublished draft. It refuses to overwrite a published release.
6. **Test the candidate.** Run the loopback test and install the actual downloaded draft on a second Mac if available. Check startup, input access, a short controlled mouse/key run, backgrounds, update cancellation/relaunch and version change. Local update testing does not establish another Mac's Gatekeeper/TCC behavior.
7. **Publish manually when ready.** Review assets and notes, then publish the draft in GitHub. Ensure its appcast URL and ZIP URL work without signing in. A private or draft release returning 404 is not a successful public update service.

## Build / draft commands

```sh
./Scripts/build.sh Release
# The pinned Swift package already supplies generate_appcast/generate_keys/sign_update:
# build/SourcePackages/artifacts/sparkle/Sparkle/bin/
gh auth status
./Scripts/prepare-update.sh "$PWD/build/Build/Products/Release/lucent.app" v0.1 Updates/notes.html
```

Use `v0.1.1` after changing the app to version 0.1.1/build 4. `SPARKLE_TOOLS_DIR` can override the pinned tool directory if deliberately needed. Run `gh auth login` as the publishing account only if not already authenticated. No CI secrets are required for the local workflow.

`prepare-update.sh` refreshes assets only while a release is still a draft. Published releases are left intact; bump the version/build instead. Each generated feed points at the full latest archive, so older clients do not need every intermediate version. Avoid automatic/delta publishing until you actually need it.

## Controlled private update test

```sh
./Scripts/build.sh Release
python3 Scripts/test-updater.py
```

Quit a previous fixture app before rerunning. The script creates two ignored copies under `build/UpdaterTest`, with a distinct bundle ID and a random loopback-only feed. In its app, choose Settings → Misc → Check for updates, then Install Update, then Install and Relaunch. Confirm v0.1.1/build 101 replaces v0.1/build 100. Ctrl-C stops the local server; quit the fixture afterward and reopen the main Release app. This uses the real Sparkle signature/download/install lifecycle without putting GitHub credentials in the client.

## Reference

- [Sparkle setup](https://sparkle-project.org/documentation/)
- [Sparkle publishing/signing](https://sparkle-project.org/documentation/publishing/)
- [Apple: opening an unnotarized app](https://support.apple.com/en-us/102445)
