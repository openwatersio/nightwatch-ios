# Apple's answer on app control of Color Tint

We asked Apple Developer Technical Support (DTS) whether an App Store app can turn Color Tint on and off. The answer is no: iOS has no public API or entitlement for it. This plan records what Apple said, what it means for Nightwatch, and the follow-up we owe Apple.

DTS case: Case-ID 22642708, answered by Travis Trotto (DTS Engineer) on 2026-11-03. Put the Case-ID line in every follow-up email.

## What we asked

We sent a code-level support request with a focused sample project, NightwatchAccessibilityProbe. It begins and ends a grayscale override with `AXFeatureOverrideSessionManager` and records errors and the grayscale state. The questions were:

1. Is there a public API or entitlement that lets an App Store app, with the user's consent, turn Color Filters > Color Tint on and off?
2. Does the grayscale override force grayscale, or can it turn on a Color Tint the user has already configured?
3. Is the `com.apple.developer.accessibility.merchant-api-control` entitlement available for a consumer app like this?

## What Apple said

DTS ran the sample project and answered each question.

| Question | Answer |
| --- | --- |
| API or entitlement for Color Tint | None that DTS knows of, including with user consent. [`AXFeatureOverrideSession.Options`](https://developer.apple.com/documentation/accessibility/axfeatureoverridesession/options) covers only grayscale, invert colors, Voice Control, VoiceOver and zoom. |
| What the grayscale override does | It applies grayscale. It does not turn on an existing Color Tint. Grayscale and Color Tint are both Color Filters options and only one can be selected, so grayscale replaces the tint while the override is active. |
| How long the override lasts | Only while the app is in the foreground. DTS observed the override ending when they switched apps. This isn't documented. |
| Merchant entitlement | The override requires `com.apple.developer.accessibility.merchant-api-control`. DTS can't say whether it would be granted for this use. App Review is a separate decision that neither DTS nor engineering can promise. |

DTS also confirmed a bug. Ending the override is expected when the app leaves the foreground, but the user's Color Tint should come back afterwards, and it doesn't. Color Filters stays off until the user turns it on again in Settings.

## What this means for Nightwatch

No supported API gives an app a red tint that stays on across apps. The Shortcuts and Focus setup in the [user guide](../README.md) stays the only working route, and the instructions-only approach stays the deliverable.

Nightwatch must not use the grayscale override. It would turn the screen gray instead of red, end as soon as the user leaves the app, and leave the user's saved tint switched off.

The [ideal app experience](ideal-app-experience.md) still depends on Shortcuts for the actual switch. An app can guide setup and launch the shortcuts, but it can't switch the filter itself.

## Follow-up with Apple

DTS asked for two separate Feedback Assistant reports and will route them to the right engineering teams once we reply with the IDs.

- [x] Run NightwatchAccessibilityProbe on an iPhone and record the steps.
- [x] File the bug report: "Color Tint is not restored after an AXFeatureOverrideSession grayscale override ends."
- [x] File the enhancement request: "Allow apps to turn the user's Color Filters Color Tint on and off with consent." It asks for either a Color Tint option for `AXFeatureOverrideSession` that can stay on across apps, or a consent-gated switch for the user's existing Color Filters setting. In both cases the user sets the tint in Settings and the app only switches it on and off.
- [x] Reply to DTS on Case-ID 22642708 with both FB numbers.

| Report | Feedback ID |
| --- | --- |
| Bug: Color Tint not restored | FB25437254 |
| Enhancement: app control of Color Tint | FB25437269 |

Screenshots and screen recordings don't capture Color Filters, so a recording can't show the tint itself. It shows the Color Filters switch in Settings instead: on before the override, off after switching apps. Only a photo or video of the phone taken with another device shows the red.

## What would change the plan

Revisit app control if Apple ships either of these:

- A Color Tint or general Color Filters option in `AXFeatureOverrideSession.Options`, or a new API that switches the user's Color Filters setting.
- A reply on the enhancement request saying such an API is planned, or that the merchant entitlement can be used for this kind of app.

Check the Accessibility framework release notes for each major iOS release, and check the enhancement report's status in Feedback Assistant.
