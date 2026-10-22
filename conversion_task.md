# Agent Task: Convert a Chrome Web Store Extension to Firefox

## Objective

Take the specified Chrome Web Store extension, download its package, extract its files, adapt it to Firefox, and produce a build that can be tested in Firefox.

**This is a one-time conversion task, not a software-development task. Do not write a Python program, general-purpose converter, CLI, or conversion framework.** Perform the steps directly using available shell commands and existing tools. Make only the edits needed for this particular extension.

## Inputs

- **Chrome Web Store URL or extension ID:** `<CHROME_EXTENSION_URL_OR_ID>`
- **Output directory:** `./chrome-to-firefox-output/` (unless otherwise specified)

If the extension URL/ID is available elsewhere in the conversation or repository, use that value. If the input is missing, report that as a blocker rather than inventing an extension.

## Task

### 1. Download the Chrome extension

1. Determine the Chrome extension ID from the supplied Chrome Web Store URL or ID.
2. Download the extension's original `.crx` package using an available public Chrome Web Store download/update endpoint, if accessible. Check the response instead of assuming a fixed endpoint will always work.
3. Verify that the file is actually a CRX package and record its SHA-256 checksum.
4. Preserve the downloaded package in `original/`.
5. If the store cannot provide the package, check whether a copy is already available in the repository or given inputs. Otherwise, report the blocker; do not download from untrusted third-party mirrors or bypass access controls.

### 2. Unpack and inspect

1. Extract the package, accounting for CRX2/CRX3 headers and the embedded ZIP payload. Do not assume renaming `.crx` to `.zip` is sufficient.
2. Extract safely, preventing path traversal and unexpected filesystem writes.
3. Save the extracted files unchanged under `original/extracted/`.
4. Inspect `manifest.json`, background logic, content scripts, popup/options pages, permissions, web-accessible resources, and browser-specific API calls.
5. Identify minified or bundled JavaScript if present. Beautify a *separate copy* when useful for analysis, but do not claim to recover original unminified source code.
6. Keep copyright and license notices intact.

### 3. Convert for Firefox

1. Copy the extracted extension into `firefox/` and edit only this copy.
2. Update `manifest.json` as needed for Firefox. Keep the original manifest version unless there is a specific compatibility reason to change it.
3. For Manifest V3 extensions, adapt Chrome's `background.service_worker` to Firefox's supported background model (`background.scripts` or `background.page`), accounting for any required code changes, imports, and service-worker-specific behavior.
4. Add Firefox-specific `browser_specific_settings.gecko` metadata as needed. Choose a stable, valid extension ID and document it.
5. Check each Chrome-specific API or manifest feature actually used against Firefox support. Make targeted adjustments rather than indiscriminately replacing `chrome.*` with `browser.*`.
6. Check hard-coded `chrome-extension://` URLs, Chrome extension IDs, CSP restrictions, permission differences, and referenced assets.
7. Keep code local to the package; do not introduce remotely executed code, remove security checks, or broaden permissions simply to suppress validation errors.
8. If a Chrome-only feature has no Firefox equivalent, try a reasonable alternative. If it cannot be preserved, document precisely what will not work, including the relevant files or symbols. Do not silently delete behavior.

Use the existing extension implementation as the source of truth; do not rewrite the extension from scratch unless unavoidable.

### 4. Validate and build

1. Verify that `manifest.json` parses and that all referenced local files exist.
2. Run Mozilla's `web-ext lint` against the `firefox/` directory. Resolve actionable compatibility errors where possible; document unresolved warnings or errors.
3. Run `web-ext build` to create an unsigned Firefox extension ZIP in `dist/`. Keep the archive clean, with `manifest.json` at its root.
4. Verify the archive can be opened and contains the expected files.
5. If feasible in the environment, load the extension temporarily using `about:debugging` > **This Firefox** > **Load Temporary Add-on**, or test it with `web-ext run`. Inspect errors and test the extension's main behavior.
6. Distinguish passing lint/build checks from successful runtime testing. Do not report that the extension works fully unless that has been verified.

**Do not publish, upload, sign, or submit the extension to Mozilla automatically.** Permanent installation in standard Firefox generally requires Mozilla signing; the output from `web-ext build` is unsigned and intended for development/testing until signed through an appropriate process.

## Expected output

```text
chrome-to-firefox-output/
  original/
    extension.crx
    extracted/
      manifest.json
      ...
  firefox/
    manifest.json
    ...
  dist/
    <extension-name>-firefox.zip
  conversion_report.md
```

Adapt filenames if required. Keep the original and converted versions separate.

## Conversion report

Create `conversion_report.md` containing:

- Extension name, Chrome Web Store URL/ID, original version, and manifest version.
- Source download status and SHA-256 checksum.
- Files changed and why.
- Firefox compatibility issues, features that could not be ported, and any remaining manual work.
- Exact `web-ext lint` and `web-ext build` results.
- The build artifact path and clear instructions for temporary Firefox installation.
- Whether runtime functionality was tested, and the outcome.
- Overall status: **SUCCESS**, **PARTIAL**, or **FAILED**, with an explanation.

## Completion criteria

The task is complete when the agent has **actually attempted** the download, extraction, conversion, validation, and build for the specified extension, saved all available artifacts, and reported any blockers or unresolved incompatibilities.

Do not substitute a plan, instructions, or a newly written conversion script for the actual work. Run the necessary commands, modify the extension files directly, and provide the packaged Firefox build when technically possible. Never claim to have completed a step that was not executed.

## References

Check current documentation for the Firefox version being targeted:

- [Firefox background manifest](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/background)
- [Chrome incompatibilities in Firefox](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Chrome_incompatibilities)
- [Firefox-specific manifest settings](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/browser_specific_settings)
- [Mozilla web-ext](https://extensionworkshop.com/documentation/develop/getting-started-with-web-ext/)
- [Mozilla extension signing](https://extensionworkshop.com/documentation/publish/signing-and-distribution-overview/)
