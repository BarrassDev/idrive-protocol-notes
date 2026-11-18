# iDrive Connected Apps — protocol notes

Community notes on the BMW/MINI "Apps for Android" (A4A) car interface: the Etch RPC
surface, the Car Data Service properties, the per-app capability flags, and the map
overlay file format. These are interoperability notes for building unofficial Connected
Apps, in the spirit of [BimmerGestalt](https://github.com/BimmerGestalt) (AAIdrive,
IDriveConnectKit, and the [BMWConnectedAnalysis](https://bimmergestalt.github.io/BMWConnectedAnalysis/)
write-up, which this builds on).

They were produced by reading the publicly distributed official apps, the same way
AAIdrive's own build does, to document facts about the interface. Nothing here is a secret
or a bypass: the car still requires a valid BMW-signed certificate to connect at all.

## What is NOT in this repo, on purpose

- **No certificate files.** The `.p7b` app certificates are BMW's signing credentials and
  are what authenticate an app to the car. They are not redistributed here. Like AAIdrive,
  you extract them yourself from an official app you have installed.
- **No BMW code, layouts, images or text databases.** Those are BMW's copyrighted resources.
  This repo documents their structure, not their contents.

If you are building an app, you still need an official app's `assets/carapplications`
resources locally, exactly as [AAIdrive's `external/README`](https://github.com/BimmerGestalt/AAIdrive/blob/master/external/README.md)
describes.

## Contents

- [services.md](services.md) — the Etch RPC services, including calls not in the
  open-source IDL.
- [app-permissions.md](app-permissions.md) — the capability flags each app certificate
  carries, and which car services they unlock. This is the main thing these notes add.
- [cds-properties.md](cds-properties.md) — Car Data Service property names.
- [map-overlay-format.md](map-overlay-format.md) — the KMZ format for the `map_*` POI
  overlay service.

## How the interface works, in one paragraph

The phone opens a TCP connection to the car (tunnelled over BCL via USB/Bluetooth) and
speaks Apache Etch RPC. It logs in by signing a challenge with a BMW-issued app
certificate (`sas_certificate` then `sas_login`). The certificate also declares, in an
X.509 extension, which services the app may use and the SHA-256 digests of the UI layouts
it is allowed to upload. After login the app registers an entry in the car menu (`am_*`),
uploads an RHMI UI description and resources (`rhmi_*`), and subscribes to live car data
(`cds_*`). Other service families (`map_*`, `voice_*`, `rcs_*`, `pia_*`) exist but are
gated by the certificate's capability flags.

## Sources

Method names, property names and formats were read from the official BMW Connected Classic,
BMW Connected / MINI Connected, MINI Connected Classic and MyBMW apps, and cross-checked
against IDriveConnectKit and the BMWConnectedAnalysis write-up. Versions are noted in each
file. Corrections welcome.
