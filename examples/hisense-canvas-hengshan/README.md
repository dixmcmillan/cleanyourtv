# Hisense CanvasTV S7N (hengshan_canvas), Google TV 12

The US CanvasTV, Hisense's art TV, runs Google TV on the same `hengshan` board and `STT2.230310.002` build as
the [U75](../hisense-u75-hengshan/README.md); `ro.product.name` is `hengshan_canvas`. Read the U75 notes first:
its Live TV, display and remote-key findings come from the same firmware. Outside the US the CanvasTV ships with
VIDAA, which has no ADB, and nothing here applies to it.

## Art Mode and the Art button

Art Mode is the system app `com.vt.arttv.artmode` ("Art Gallery", `/system_ext/app/Vtp_ArtMode`, runs as
`android.uid.system`). The remote's Art button sends keycode **708**, and logcat shows the same route the U75's
app buttons take:

1. The window policy consumes 708 (`Dropped event because policy consumed it`).
2. `com.vt.tv.smartsense`'s `GlobalKeyReceiver` takes it and calls `sendBroadcastToLauncherX` for
   `com.vt.arttv.artmode`.
3. launcherx (`GlobalButtonLaunch`) starts `com.vt.arttv.artmode/.ArtModeActivity` with
   `category.LEANBACK_LAUNCHER` and extras; the activity logs `start reason = REMOTE_BUTTON`.

So once launcherx is disabled, the Art button does nothing. `config.json` maps it back with
`"708:com.vt.arttv.artmode"` in `app_keys`, which opens the same Leanback activity, for `HomeKeyService`
(`HOME_KEY_SERVICE=1`). That mapping is untested: the activity may start with a different start reason
when launcherx does not add its extras. Check it, and the items below, before living with the list.

- Art button opens and leaves Art Mode.
- Motion detection (a radar sensor; `getMotionDetectorStatus` read level 2) blanks and wakes the art.
- The light sensor still dims the art (`GetLightSensorValue` logs every second in Art Mode).
- Art downloads and photos from the phone still arrive with VIDAA Channels Plus and the content tab off.
- Power-on behaviour: if the set is set to come back into Art Mode, `power_on_home` must stay off.

Over ADB, `input keyevent 708` enters Art Mode exactly like the button (tested with launcherx enabled), and
`input keyevent KEYCODE_HOME` leaves it straight to Home. Back opens an exit confirmation instead. This is the
hook for a home-automation "Art Mode at sunset" routine.

## Picture modes

The picture mode is not in `settings`: Settings sets it with a private HSP call (`setPictureMode` on
`vt_hs_av_service`), so there is no public command for it. The mode numbers, from Settings' log:

| Value | Mode |
|---|---|
| 1 | Standard |
| 2 | Energy Saving |
| 3 | Theater Day |
| 5 | Sports |
| 6 | Theater Night |
| 7 | FILMMAKER MODE |

## Status

Probed and backed up 2026-10-25 on one US unit (wired, firmware above). The list has not been applied yet;
it reaches upstream only after a few days of living with it, per CONTRIBUTING.
