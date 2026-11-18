# Map overlay (`map_*`) POI format

The car exposes a POI overlay service that draws pins on its own navigation map. It is in
the open IDL but was undocumented in practice. Reading BMW Connected Classic 1.8 recovered
the file format its SDK builds for the import.

## The calls

```
int  map_create()
int  map_initializeImport(handle, filename, transferId, size)
void map_importData(handle, transferId, seq, byte[] chunk)
void map_finalizeImport(handle, transferId)
void map_abortImport(handle, transferId, reason)
void map_showOverlay(handle, filename, overlayId)
void map_hideOverlay(handle, filename, overlayId)
void map_highlight(handle, filename, overlayId, location)
void map_setMode(handle, MapMode)        // ALL_LOCATIONS_WITH_CCP | ALL_LOCATIONS_WITHOUT_CCP | HIGHLIGHTED_LOCATION
void map_remove(handle, filename)
void map_dispose(handle)
// callback: map_onEvent(handle, transferId, MapEvent)
//   MapEvent: MAP_READY, MAP_IMPORT_READY, MAP_IMPORT_ABORT, MAP_IMPORT_DONE, MAP_DELETE_DONE
```

Flow: `map_create` → `map_initializeImport` with the file name and total byte size → one or
more `map_importData` chunks (seq 0,1,…) → `map_finalizeImport` → `map_showOverlay`. The car
reports progress through `map_onEvent`. Requires a certificate with `MAP=true` (see
[app-permissions.md](app-permissions.md)).

## The file: a KMZ

`map_initializeImport` takes a filename ending `.kmz`. The payload is a ZIP containing
`doc.kml` plus one `normal_<n>.png` and one `highlight_<n>.png` icon per POI. This is what
BMW's `com.bmwgroup.connected.util.kmz` helper writes.

### doc.kml structure

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<kml xmlns="http://www.opengis.net/kml/2.2"
     xmlns:gx="http://www.google.com/kml/ext/2.2"
     xmlns:kml="http://www.opengis.net/kml/2.2"
     xmlns:atom="http://www.w3.org/2005/Atom">
  <Folder>
    <name>a4a</name>
    <!-- repeated per POI, index n = 0,1,2,... -->
    <Style id="normal_0">
      <IconStyle><Icon><href>normal_0.png</href></Icon></IconStyle>
    </Style>
    <Style id="highlight_0">
      <IconStyle><Icon><href>highlight_0.png</href></Icon></IconStyle>
    </Style>
    <StyleMap id="style_0">
      <Pair><key>normal</key><styleUrl>#normal_0</styleUrl></Pair>
      <Pair><key>highlight</key><styleUrl>#highlight_0</styleUrl></Pair>
    </StyleMap>
    <Placemark id="id0">
      <name>POI label</name>
      <address></address>
      <phoneNumber></phoneNumber>
      <styleUrl>#style_0</styleUrl>
      <Point><coordinates>LON,LAT</coordinates></Point>
    </Placemark>
  </Folder>
</kml>
```

Points to note:
- The folder name is the literal `a4a`.
- Each POI has a `normal`/`highlight` icon pair wired through a `StyleMap`; `map_highlight`
  switches a placemark to its highlight icon (the `location` argument is believed to be the
  placemark id, e.g. `id0`).
- Placemark ids are `id0`, `id1`, … in order.
- `<coordinates>` is written longitude-first, as standard KML. (BMW's code emits the two
  doubles straight from its POI fields without labels; standard KML order is lon,lat. If
  pins land in the wrong hemisphere, swap them.)
- `<address>` and `<phoneNumber>` are present even when empty.
- Icons are PNGs; BMW uses one normal and one highlighted variant per pin.

## Caveats

- Even when the overlay works, the car's native navigation does **not** raise proximity
  warnings for these pins. They are map decoration only.
- This is reconstructed from the SDK's builder, not from an observed car transaction. The
  coordinate order and the `map_highlight` location argument are the two points most worth
  confirming against a real head unit.
