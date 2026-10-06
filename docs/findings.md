# Findings and remaining checks

Nightwatch is an instructions-only project. Its deliverable is a repeatable setup using Color Filters, Shortcuts, and Focus. A website can present this guide once the full setup is verified. No app purchase, app installation, or subscription is required by this guide.

## Device observations

- A custom Nightwatch Focus and a manually created Focus-triggered Color Filters toggle worked for the project owner. The Off automation was not yet created at the time of that report.
- The Set Color Filters shortcut action toggled the existing selection. It did not configure red hue or intensity. Grayscale was the initial selection during testing.
- Color Tint could be configured manually in Settings and then toggled with a shortcut.
- The iPad's description-based shortcut builder worked. The simulator remained on a preparation screen. Generated actions still need inspection.
- Tested Settings links either failed or opened the Apps page. Searching Settings for Color Filters was usable.
- The app's shortcut URL opened Shortcuts before returning. Native controls are the guide's everyday entry points.
- The grayscale accessibility reading identified grayscale in the device test. A false reading did not identify red tint or prove the filter was off.
- White text on a bright red button became unreadable under the physical red filter. Help content should use a dark background and strong light/dark contrast.
- The experimental hidden-text completion pattern has not been validated. It is not a setup requirement.

Apple has received a support request about direct accessibility API control, including the entitlement question. No response resolving that question is recorded here. The guide does not depend on that outcome.

## What the guide must not promise

A shared shortcut does not package the user's custom Focus, personal automation triggers, saved Color Tint, or screen assignments into one installation. The guide walks through those settings explicitly. A successful shortcut run is not proof that red hue and intensity are correct.

Red at maximum Color Tint intensity is a provisional setup target. We have not established an optimal tint or measured its effect on night vision.

## Before publishing install links or a website

- [ ] Verify separate On and Off shortcuts and Focus automations on a physical iPhone, including locked execution and switching to a different Focus.
- [ ] Verify the scheduled end time restores regular colors.
- [ ] Verify Control Center and Lock Screen controls, including any unlock requirement.
- [ ] Verify Focus-linked Lock Screen and Apple Watch face behavior, including cross-device Focus settings.
- [ ] Export and test shared shortcut links on a device without the shortcuts installed. Keep the manual action recipes available for repair.
- [ ] Capture iPhone instructions showing Color Tint and the below-the-fold Intensity and Hue controls. Use physical-screen photos only when demonstrating the applied tint.
- [ ] Research the red preset before making night-vision performance claims.

The local SwiftUI experiment is preserved outside the tracked files. This repository contains the setup guide, reference image, and these findings.
