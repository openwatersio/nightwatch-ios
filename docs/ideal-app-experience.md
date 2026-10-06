# The ideal Nightwatch app

Nightwatch's intended app is a $5.99 one-time purchase that makes a system-wide red display easy to configure, activate, schedule, and repair. This is the product concept, not a description of a shipping app. The current deliverable is the [setup guide](../README.md); the [technical findings](technical-findings.md) explain the gaps between that guide and this experience.

## The problem

A red mode inside Slackwater only helps while Slackwater is visible. Switching apps, opening the app switcher, or checking the Lock Screen can expose bright interfaces again. People need the whole phone to remain in the chosen display mode while they move through their usual tasks.

The aim is to make nighttime use less disruptive to night vision. We still need research and measurement before claiming a particular tint or intensity achieves that aim.

## What someone is paying for

The value is a coherent setup: a chosen red preset, a dependable switch back to regular colors, familiar system controls, optional scheduling, and help when something breaks. The person should not have to understand accessibility APIs or design a set of automations to get that result.

The intended price is $5.99 once, with no subscription. This price belongs to the app concept; the instructions repository does not require a purchase. A wrapper that mostly sends people elsewhere to configure everything does not yet deliver the experience we wanted to charge for.

## First use

In the ideal version, Nightwatch explains what will change, obtains any required system authorization, applies the default red preset, and lets the person try switching between red and regular colors. We choose the preset. There is no “choose your red” task during onboarding; research should inform the default before we present it as the right nighttime setting.

The app should do each supported setup step itself. If system approval or manual configuration is unavoidable, it should explain the next action exactly where it is needed, preserve progress, and make it easy to come back.

Creating a Focus automatically was an early entry-point idea. It is not a requirement for the core manual switch. Focus becomes useful when someone wants a schedule, a dedicated Lock Screen, or a watch face to accompany Nightwatch. Any automatic Focus creation remains an unproven capability.

### The Shortcuts-based setup we explored

1. Create the shortcut first. One button copies the prepared description and opens Shortcuts; the person pastes it into the builder where available. A manual single-action recipe is always available.
2. Run it. On a device using the default filter, grayscale is useful evidence that filtering is active. An existing red configuration must also be allowed. A successful callback is evidence of execution, not proof of correct contents.
3. Configure Color Tint. Copy “Color Filters” and open Settings, with search instructions. Show a tight reference image of Color Tint, then explain scrolling to Hue and Intensity. Tell the person to set red and maximum intensity as the provisional preset.
4. Return and continue. Expect multiple trips because the tint may be wrong or intensity may be incomplete. Keep progress and the relevant visual guidance available.
5. Confirm the visible result and leave setup. Do not toggle automatically merely because the person returned from Settings: the app does not know whether filtering is on. Keep an explicit way to return to regular colors.

A later experiment proposed two visual sections: setup instructions readable without the desired tint, and a completion section intended to become readable with it. The completion action would take the person straight out of setup, leaving the tint on. This avoids a “which can you see?” quiz, but the pattern remains unvalidated. It cannot be the only way to finish or repair setup, and it cannot justify a claim that the app detected red tint.

## Everyday use

The normal screen is calm and brief: the Nightwatch identity, the main color switch, access to automation, and setup or repair when needed. This is the screen intended for product screenshots. Debug probes and lengthy onboarding instructions do not belong on it.

In the ideal implementation, the switch operates without a trip through another app and shows accurate current state. In the Shortcuts implementation, the app handoff is visible and live red state is unknown. The copy must reflect that: “Switch colors” can describe an action, while “Red is on” requires evidence we did not have.

Nightwatch is not a high-open-frequency app. Someone sets it up, then uses Control Center, the Lock Screen, Focus, or schedules. They should not need to keep Nightwatch open for automation to work.

## System controls and automation

The intended package includes a manual control, reliable automatic entry and exit, and optional nighttime screens.

| Entry point | Desired experience | Current instructions approach |
| --- | --- | --- |
| App's main button | Change the whole display immediately | Run the toggle through Shortcuts; expect an app handoff |
| Control Center | Reach Nightwatch without opening the app | Use Focus, or a native Shortcut control for a filter-only override |
| Lock Screen | Activate or leave Nightwatch from a familiar control | Configure the native control and verify any unlock requirement |
| Time schedule | Enter at the chosen time and leave at the end time | Schedule Nightwatch Focus and use separate On/Off automations |
| Focus | Coordinate tint, notification preferences, and chosen screens | User creates Nightwatch Focus and its personal automations |
| Watch face | A dark face with red details when Nightwatch is active | Link a face to Focus; this does not apply the phone's filter to the Watch |

Separate On and Off actions are essential for automation. Repeating On should keep Nightwatch on. A manual Toggle is an optional override, and the interface must explain that it can make the tint disagree with Focus until the next transition.

Screen and notification customization should remain optional. Someone who only needs a reliable red switch should not have to choose a wallpaper, build a watch face, or change notification rules.

## Returning because something broke

Assume a person reopening Nightwatch may have accidentally deleted or edited a shortcut. Recheck what the platform allows on every opening. Show a specific repair path rather than trusting a flag saved during onboarding.

Distinguish a missing shortcut, an execution failure, and a filter that needs configuring. Retain known setup progress so repairing a shortcut does not force every Settings step again. If the shortcut runs but the screen looks wrong, send the person to the Color Tint guidance. If only automated exit fails, show the Off automation instructions.

The ideal app verifies installed shortcuts and their contents without a disruptive round trip. The spike could not deliver that check: names and callbacks each provide only partial evidence. A repair experience must state those limits honestly.

## Readability under the filter

Keep essential navigation and recovery controls readable in regular colors, grayscale, red tint, and partial intensity. The physical test showed that white text on red could disappear. Use strong light/dark contrast for ordinary controls and test on the actual display. Respect larger text, VoiceOver, and Reduce Motion; color-dependent experiments must have an accessible alternative.

Setup images need to show exactly where to tap and when to scroll. Treat them as references, not measurements of the person's settings. The screenshot-capture limitation belongs in a footnote, not in the main explanation of the product.[^screenshots]

## What would make the app worth revisiting

We need a supported way to remove a substantial part of the manual work: configuring the system tint, changing it without app switching, reading enough state for an honest control, or reliably installing and repairing the necessary pieces. Apple's pending support answer may clarify part of this, but we have no commitment that it will.

If a direct API becomes available for this use, keep the familiar controls and simplify setup. Offer a deliberate migration that disables conflicting personal automations before relying on direct control. The person should not have to run both systems or guess which one changed the screen.

Until then, the guide carries the useful experience: clear defaults, explicit On/Off recipes, native controls, and practical repair instructions.

[^screenshots]: The red tint was visible on the physical display but absent from our screenshots. Product imagery must distinguish an illustrative red rendering from a capture of the actual filtered display.
