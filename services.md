# Etch RPC services

The car speaks Apache Etch. The open-source
[IDriveConnectKit IDL](https://github.com/BimmerGestalt/IDriveConnectKit) defines 66 calls
across the well-known families. Reading the official apps shows the full surface the BMW
client actually knows: about 120 call names. The extra ones are listed here so other apps
know they exist. Most are gated by certificate flags (see
[app-permissions.md](app-permissions.md)), so being in the protocol does not mean every app
can use them.

## Families in the open IDL (documented upstream)

`sas_*` login, `info_*`/`ver_*` version, `cds_*` car data, `am_*` app menu, `rhmi_*` UI,
`av_*` audio, `map_*` map overlay. See IDriveConnectKit and
[BMWConnectedAnalysis](https://bimmergestalt.github.io/BMWConnectedAnalysis/) for their
signatures.

## Calls present in the official client but not in the open IDL

Grouped by family. Parameter types were obfuscated in the app and are not all recovered;
names are the real Etch symbols (the wire protocol hashes them, but the client ships the
names).

### `rcs_*` — remote control service (21 calls)
Drives the iDrive controller and entertainment from the phone side:
`rcs_create`, `rcs_dispose`, `rcs_button`, `rcs_rotary`, `rcs_skip`, `rcs_volume`,
`rcs_mute`, `rcs_lock`, `rcs_control`, `rcs_coords`, `rcs_playbackControl`,
`rcs_playlistControl`, `rcs_headphoneControl`, `rcs_entSourceControl`, and the matching
event callbacks `rcs_muteEvent`, `rcs_lockEvent`, `rcs_playbackEvent`, `rcs_headphoneEvent`,
`rcs_entListEvent`, `rcs_entPlaylistEvent`, `rcs_entSourceEvent`.
Related enums: `ButtonState`, `RotaryTilt`, `PlaybackState`, `PlaylistCommand`, `SkipState`,
`VolumeState`.

### `pia_*` — personal information / data import (6 calls)
`pia_create`, `pia_dispose`, `pia_import`, `pia_request`, `pia_exportResult`, `pia_onEvent`.
Events (`PIAEvent`): `PIA_READY`, `PIA_IMPORT_REQUEST`, `PIA_IMPORT_DONE`,
`PIA_IMPORT_FAILED`, `PIA_REQUEST_RESULT`, `PIA_EXPORT`, `PIA_STOPPED`. A chunked
import/export handshake similar in shape to the map import.

### `voice_*` — voice sessions (5 calls)
`voice_create`, `voice_dispose`, `voice_requestSession`, `voice_stopSession`,
`voice_sessionStateChanged`. Enums `VoicePriorityType`, `VoiceSessionStateType`. Requires
`rhmi.tts`-class permissions.

### `vrs_*` — voice recording service (4 calls)
`vrs_create`, `vrs_dispose`, `vrs_exportResult`, `vrs_onEvent` (`VRSEvent`).

### `vds_*` — vehicle diagnostics (4 calls)
`vds_create`, `vds_dispose`, `vds_diagnosticHandler`, `vds_respond` (`VDSDiagnosticsType`).

### `rm_*` — resource/limitation summary (3 calls)
`rm_create`, `rm_dispose`, `rm_getSummary`. Returns a memory/HMI-process summary
(the SDK's `LimitationSummary`: two longs, a string, an int).

### `persist_*` — persistent resources (2 calls)
`persist_checkPresistentResources`, `persist_setResource` (note BMW's spelling). Lets an
app cache resources on the head unit across sessions (`PersistentResourceType`).

### `log_*` / `diag_*` — logging and latency probes
`log_create`, `log_dispose`, `log_log`; `diag_echo`, `diag_perf_start`, `diag_perf_end`,
`diag_perf_oneway`, `diag_perf_twoway`. Developer plumbing.

### Other
`cds_onPropertyChangedBinaryEvent` — the binary counterpart to the text CDS change event
(pairs with the open IDL's `cds_setPropertyBinaryAsync`). `am_showLoadingFailHint` — a
loading-failed hint (the open IDL has `am_showLoadedFailHint`).

## Caveat

Presence in the protocol is not access. The certificate's capability flags decide what a
given app may actually call, and the car returns a permission error otherwise. Treat this
list as a map of what exists, then check [app-permissions.md](app-permissions.md) for what a
certificate would need.
