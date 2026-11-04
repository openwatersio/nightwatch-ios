# Technical findings and API limits

The Nightwatch spike explored a system-wide red display mode controlled by an iOS app. The working path uses Apple's saved Color Tint and Shortcuts. We did not establish an app-accessible way to configure that tint, read its live state, or install the complete Focus setup. This document preserves the implementation lessons without requiring the local SwiftUI project.

Evidence here comes from the local source, recorded device observations, and linked Apple documentation. A build passing proves compilation, not filter behavior. An unresolved API question is not a claim that Apple can never support it.

## Capability map

| Capability | Evidence and limit | Consequence |
| --- | --- | --- |
| Toggle the saved Color Filter | Observed on iPad; the initial selection was grayscale | Configure Color Tint separately in Settings |
| Set the filter explicitly On or Off | Project owner reported a successful Focus On/Off cycle on iPhone; locked and scheduled execution remain unchecked | Use explicit states in automations, not Toggle |
| Set hue, intensity, or filter type from Shortcuts | The tested action exposed only enable/disable/toggle | A generated description cannot add parameters the action lacks |
| Read grayscale | UIKit API and positive device observation | Useful during setup, not a red-state sensor |
| Read red tint or its intensity | No reliable app path established in the spike | Keep live state unknown |
| Inspect installed shortcuts silently | No reliable app path established | A helper can return names; it cannot prove action correctness |
| Open Color Filters directly | Tested experimental links failed or landed on Apps | Copy the search term and explain Settings navigation |
| Run a shortcut from the app | URL handoff and callbacks implemented; app switching observed | Expect Shortcuts to appear, rather than promising an inline toggle |
| Turn Color Tint on or off from the app | Apple DTS confirmed no public API or entitlement; the grayscale override replaces the tint and ends when the app leaves the foreground | Keep the switch in Shortcuts |
| Create the user's Focus and personal automations | No supported installation path established for this app | The user creates them in Apple's interfaces |
| Run from Control Center or Lock Screen | Native Shortcut controls provide entry points | A custom app control would not itself grant Color Filters access |

The [user guide](../README.md) contains the actual shortcut and automation recipes. [Findings and remaining checks](findings.md) tracks the outstanding device work.

## Reading state without inventing certainty

Apple describes [`UIAccessibility.isGrayscaleEnabled`](https://developer.apple.com/documentation/uikit/uiaccessibility/isgrayscaleenabled) as indicating that Color Filters and Grayscale are enabled. On the tested iPad it was true for grayscale and did not report red Color Tint as enabled.

The core read was:

```swift
@MainActor
func observeGrayscale() -> Bool {
    UIAccessibility.isGrayscaleEnabled
}
```

The app refreshed this value on foreground activation, after shortcut callbacks, and through `UIAccessibility.grayscaleStatusDidChangeNotification`.

| Observation | What it tells us |
| --- | --- |
| Grayscale is true | Grayscale is active; the desired red preset is not currently displayed |
| Grayscale is false | Filtering may be off, red may be on, or another filter may be selected |
| Shortcut returns success | The shortcut reports completion; its contents and visible effect remain unverified |
| User finishes tint setup | The user confirmed setup at that moment; it is not a live system-state reading |

Grayscale also does not prove our shortcut is installed: the user could have enabled it in Settings. An already-configured red device should not have to switch to grayscale to pass setup.

The spike persisted `setupStep`, `colorConfigured`, and `setupComplete`. These represent progress and user confirmation. Its `connected` flag represented recent shortcut activity, not verified red state; even setup completion could set it. Any future implementation must keep installation evidence, execution evidence, saved configuration, and live filter state distinct.

## Clipboard and navigation

One user tap can copy text and open another app. The spike used this for the description-based shortcut builder:

```swift
UIPasteboard.general.string = prompt
UIApplication.shared.open(URL(string: "shortcuts://create-shortcut")!)
```

This opens the editor; it does not paste or submit the prompt. Description-based creation worked on the iPad. The simulator stayed on “Preparing support for describe a shortcut.” That observation alone does not establish the cause or supported-device matrix. Manual action creation remains the dependable fallback. See Apple's [Shortcuts URL guide](https://support.apple.com/guide/shortcuts/apda283236d7/ios).

For Settings, the working fallback was:

```swift
UIPasteboard.general.string = "Color Filters"
UIApplication.shared.open(URL(string: UIApplication.openSettingsURLString)!)
```

`openSettingsURLString` opens the app's Settings destination. It does not prefill Settings search or promise the Settings root. The instructions must explain returning to the main Settings page, searching, and pasting. Merely opening Settings must not advance setup.

The debug tester tried these routes:

| Experimental URL | Recorded result |
| --- | --- |
| `settings-navigation://com.apple.Settings.Accessibility/DISPLAY_AND_TEXT/DISPLAY_FILTER_COLOR` | Declined on the tested iPad |
| `settings-navigation://com.apple.Settings.Accessibility/DISPLAY_AND_TEXT` | Declined |
| `settings-navigation://com.apple.Settings.Accessibility` | Declined |
| `prefs:root=ACCESSIBILITY&path=DISPLAY_AND_TEXT/DISPLAY_FILTER_COLOR` | No working Color Filters destination in the reported tests |
| `App-prefs:root=ACCESSIBILITY` | Opened Apps rather than Accessibility |

These are observations about tested routes, not stable API contracts. `UIApplication.open` accepting a URL does not establish which Settings page appeared.

## Shortcut execution and callback handling

The spike constructed `shortcuts://x-callback-url/run-shortcut` with `URLComponents` and `URLQueryItem`. Its parameters were the exact shortcut name and three callback URLs: `x-success`, `x-error`, and `x-cancel`. Each callback carried a fresh UUID token and a status, for example `nightwatch://callback?token=…&status=success`.

The receiver checked the callback scheme, host, pending request token, and status before using it. The token correlated a response with an outstanding request; it did not authenticate Shortcuts or attest to the filter state. There was one pending request at a time. See Apple's [x-callback-url documentation](https://support.apple.com/guide/shortcuts/apdcd7f20a6f/ios).

The implementation also handled opening failure, cancellation, an error callback, and a manual return with no callback. A 600 ms foreground delay attempted to avoid racing callback delivery. That delay is a prototype heuristic, not an iOS timing guarantee. The UI offered a way to cancel a pending operation.

The Foundation checks covered name encoding, success output, stale-token rejection, wrong-scheme rejection, error and cancellation callbacks, exact-name inventory matching, empty output, and CRLF-separated names. The recorded run passed 10 assertions. Those checks do not verify the actual Shortcuts action, locked execution, or visible red tint.

## Checking installation and repairing changes

An optional `Nightwatch Check` helper used this recipe:

```text
Get My Shortcuts
Get Name for each shortcut
Combine Text with New Lines
Return the combined text
```

The app compared exact names, after trimming whitespace, against Nightwatch Toggle, Nightwatch On, Nightwatch Off, and Nightwatch Check. A similarly named copy did not count. The parser was tested; end-to-end helper behavior was not established by those parser checks.

This approach adds a helper that can itself be deleted, requires a Shortcuts round trip, and sees names rather than action contents. A correctly named shortcut with the wrong action can still pass. The simplified spike used an explicit run test and repair instructions instead of requiring that inventory helper on every opening.

For a future app, checking on every opening remains a UX goal. The supported check must accurately describe what it verifies and handle deleted, renamed, and edited shortcuts. A stored “installed” Boolean cannot meet that goal.

## Focus, scheduling, and state drift

A personal automation watches a Focus transition; an ordinary shortcut does not remain running as a listener. The intended pairing is Nightwatch Focus on → Nightwatch On, and Nightwatch Focus off → Nightwatch Off. Apple's [Focus triggers](https://support.apple.com/guide/shortcuts/apde31e9638b/ios) support both transitions.

Explicit On and Off actions are repeatable without reversing the desired state. A Toggle shortcut remains useful as a manual override, but then Focus and filter state can disagree until another transition. A saved last-command value cannot detect changes made through Settings, another shortcut, or another control.

Scheduling the Focus lets the same automations handle time-based changes. A custom app timer was not implemented. Focus-linked Lock Screens and watch faces are configured by the user; the iPhone's filter does not tint Apple Watch screens. Sharing Focus is not evidence that the local automation or filter configuration exists on every device.

## Display and visual verification

The physical-screen photo showed white text on a red button becoming unreadable under the red filter. The spike changed essential controls to white text on black with a white outline. Settings guidance used an original screenshot clipped in SwiftUI, with separate Color Tint, Intensity, and Hue tips. On iPhone, the sliders require scrolling; the supplied iPad capture does not show the full Hue control.

An experimental completion panel used cyan text `(0, 0.55, 0.65)` on white, with a red/yellow line mask. A white-on-red setup panel was intended to disappear while the completion panel became readable. This assumes behavior resembling red-channel filtering; Apple's actual transformation and intermediate intensities were not validated. It must not be treated as automatic detection or reliable calibration. A completion tap is user confirmation, and readable help must remain available regardless of the pattern.

Red at maximum intensity is provisional. Neither an optimal preset nor night-vision performance was measured. Screenshots did not include the applied red filter in our device observations; assess the display directly or photograph it.[^captures]

## Apple's answer on direct API control

Apple Developer Technical Support confirmed that no public API or entitlement lets an app turn Color Filters or Color Tint on or off, even with the user's consent. [`AXFeatureOverrideSessionManager`](https://developer.apple.com/documentation/accessibility/axfeatureoverridesessionmanager) can override only grayscale, invert colors, Voice Control, VoiceOver and zoom. Its grayscale option replaces the user's Color Tint rather than turning it on, and it lasts only while the app is in the foreground.

The override requires the [`com.apple.developer.accessibility.merchant-api-control`](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.developer.accessibility.merchant-api-control) entitlement. DTS could not say whether it would be granted for an app like Nightwatch. Even if it were, the override couldn't produce a red tint.

DTS also confirmed that the user's Color Tint isn't restored when the override ends. The instructions-only approach remains the deliverable. [Apple's answer on app control of Color Tint](apple-color-tint-api.md) has the full answer and the Feedback Assistant follow-up.

[^captures]: Screenshots can document control locations and app layout. They cannot establish the physical filtered appearance observed in these tests.
