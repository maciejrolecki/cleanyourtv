# Proximus V7 Decoder (Technicolor UIW4020PXM)

Belgian operator IPTV box: Broadcom BCM7271, 2 GB RAM (1.8 GB usable), Android TV 12, firmware
`PXM-SW-12.0.5` (STT5.251221.001, security patch 2026-07-01). Verified boot is green, it's a `user` build
with release-keys, and there is no OEM unlock. It was used here without a Proximus TV subscription, on Ethernet.

## Result

| | Before | After |
|---|---|---|
| MemAvailable 60 s after boot | ~515 MB (after 6 days of uptime) | ~800–930 MB |
| MemAvailable after 8 days of use | — | ~570 MB, ~820 MB after closing two big apps |

Stable over about two weeks and more than ten reboots, with no crashes or ANRs. All settings below survived the reboots.

## Things specific to this box

- **Stock Home** is the Proximus TV app, `be.px.stbtvclient`. With it disabled, a third-party launcher
  takes Home through `cmd role add-role-holder` plus `set-home-activity`.
- **Six PERSISTENT Technicolor services** (`ota`, `remotecontrolservice`, `dtvinput`, `cas.provider`,
  `user.provider`, `onetouchplay`) keep running when disabled, so disabling them saves nothing. Leave them on.
- **Netflix can't be stopped from starting at boot.** It's on the system deviceidle whitelist, so
  `RUN_ANY_IN_BACKGROUND ignore` has no effect. Its `BootCompleteReceiver` can't be disabled from the shell
  (SecurityException on Android 12). `com.technicolor.dm` sends it the ESN at boot. Android unloads it
  under memory pressure, so it's harmless in practice.
- **Other streaming apps** (Prime Video, HBO Max, Arte, YouTube, VLC, Canal+, RTL play, Auvio, Play NOW)
  do respect `cmd appops set <pkg> RUN_ANY_IN_BACKGROUND ignore`. Run `cmd appops write-settings` before an
  `adb reboot`, or the change is lost: appops are written lazily and `adb reboot` skips the save.
- **Languages:** the framework and TvSettings ship only `en`, `fr` and `nl`, so the Language page lists only those.
  Another locale can be set with a reflection call to `IActivityManager.updatePersistentConfiguration`
  run through `app_process` as the shell user (the shell holds CHANGE_CONFIGURATION). Apps and the
  launcher switch, while the system menus stay in English.
- **The IP can change** after a reboot (DHCP). A DHCP reservation in the router avoids it.
- **ADB authorisation expires** after about 7 days without a connection (Android 11+). Accept the prompt again.
- After `adb reboot`, `sys.boot_completed` comes up in about 30 s.

## Updates

`com.technicolor.android.customota` and `com.technicolor.OmniSciOTA` were disabled on this unit for a
while. They were re-enabled to let Proximus firmware updates arrive. Keep them, and `com.technicolor.ota`, enabled.
