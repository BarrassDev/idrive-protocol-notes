# App certificate capability flags

Every Connected App authenticates with a BMW-signed certificate. Beyond identity, the
certificate carries an **X.509 extension `1.3.6.1.4.1.513.59.4.2`** whose value is a
`key=value&key=value` string. It declares which car services the app may use and the
SHA-256 digests of the UI layouts, image databases and text databases it is allowed to
upload. The car enforces both: a call into a service the certificate does not grant comes
back as a permission error, and an uploaded layout whose digest is not listed is rejected.

This is the practical gate on what an unofficial app can do. You cannot unlock a service by
asking for it in code; it has to be granted by the certificate you log in with. That is why,
for example, the map overlay works only when you log in with a certificate that grants it.

## Flags seen in the wild

Read from the official apps (app cert common-name → extension value, trimmed):

| App / cert | Notable flags |
|---|---|
| SmartThings | `AVS=true CDS=true RHMI=true Make=BMW,MINI` |
| BMW/MINI Connected Classic (all bundled apps) | `AVS=true CDS=true RHMI=true` |
| BMW Connected / MINI Connected — `bmwone` | `rhmi.tts=true rhmi.contact=true rhmi.padfocusevents=true rhmi.padfocusevents.permanentoverride=true cds.cds.apiRegistry=3` |
| BMW Connected / MINI Connected — `calendar` | `AVS=true CDS=true RHMI=true cds.api.ipaThirdParty=3 APPID=BMWOne` |
| MyBMW — `cdsbaseapp` | `RHMI=true MAP=true rhmi.navigation=true rhmi.tts=true rhmi.phone=true rhmi.notification.center=true rhmi.menuchange.onevent=true rhmi.padfocusevents.permanentoverride=true rhmi.unlimitedfocusevents=true cds.api.carcloud=3 cds.api.ipamediaplayer=3 cds.api.ipaThirdParty=3 cds.hmi.graphicalContext=3 Make=BMW,MINI,J29 AVS=false` |
| MyBMW — `applicationmanager` | `AVS=true cds.api.ipaThirdParty=3` |

## What the flags mean

- `RHMI`, `CDS`, `AVS` — the three base services: upload UI / read car data / play audio.
  Almost every app has these.
- `MAP=true` — permission for the `map_*` POI overlay service. Only **MyBMW's `cdsbaseapp`**
  carries it among the apps inspected. The Connected Classic apps that contain the overlay
  code do **not** carry the flag, which explains why the overlay may be refused when logging
  in with their certificates.
- `rhmi.navigation=true` — lets the app drive navigation-related RHMI (again MyBMW only).
- `rhmi.tts=true` — the car reads text aloud in its own voice (the `readout` link action).
  Carried by `bmwone` and MyBMW's `cdsbaseapp`.
- `rhmi.phone`, `rhmi.contact` — phone/contacts integration.
- `rhmi.notification.center=true` — post into the car's notification centre.
- `rhmi.padfocusevents`, `rhmi.unlimitedfocusevents`, `rhmi.menuchange.onevent` — richer
  focus/scroll and menu events than the base profile.
- `cds.api.*=3`, `cds.cds.apiRegistry=3`, `cds.hmi.graphicalContext=3` — extended CDS
  registries (cloud, media player, third-party IPA, graphical context). The `=3` is a
  permission level.
- `Make=` — which brands the cert is valid for (`BMW`, `MINI`, `J29` = Toyota Supra).
- `APPID=` — ties the cert to a named app slot.
- `TUI`, `rhmi.hmicontrol` — seen set to `false` in MyBMW.

## Practical takeaway for app builders

- For UI + car data + audio, any of the common certificates works. This covers a full-screen
  map, lists, popups, the status bar, and reading GPS/speed.
- For the **car's own voice** (`readout`), you need a certificate with `rhmi.tts=true`
  (`bmwone`, MyBMW `cdsbaseapp`).
- For the **map overlay** (`map_*`), you need `MAP=true` (MyBMW `cdsbaseapp` among those
  seen). Without it, expect a permission error from `map_create` / `map_initializeImport`.
- The certificate must also whitelist the digest of whatever UI layout you upload, so you
  reuse an official app's layout unchanged rather than authoring your own.
