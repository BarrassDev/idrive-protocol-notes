# Car Data Service properties

`cds_*` lets an app subscribe to live car data. An app creates a handle (`cds_create`),
registers a property with a minimum update interval
(`cds_addPropertyChangedEventHandler`), optionally fetches it once (`cds_getPropertyAsync`),
and receives JSON change events (`cds_onPropertyChangedEvent`). Each property name maps to a
numeric ident; values are small JSON objects keyed by the property's leaf name.

These names match the open-source IDriveConnectKit `CDSProperty` list and the
[BMWConnectedAnalysis CDS table](https://bimmergestalt.github.io/BMWConnectedAnalysis/cds/),
which document idents and example payloads. Reading the official apps turned up no car-data
properties beyond that list, so the upstream table is complete. Reproduced here as a quick
index; use the upstream table for idents and sample values.

## Properties by family

- **navigation**: `GPSPosition`, `GPSExtendedInfo` (speed, heading, altitude),
  `currentPositionDetailedInfo`, `nextDestination`, `nextDestinationDetailedInfo`,
  `finalDestination`, `finalDestinationDetailedInfo`, `infoToNextDestination`,
  `infoToFinalDestination`, `guidanceStatus`, `routeElapsedInfo`, `units`.
- **driving**: `speedActual`, `speedDisplayed`, `averageSpeed`, `acceleration`,
  `acceleratorPedal`, `clutchPedal`, `brakeContact`, `gear`, `shiftIndicator`, `mode`,
  `drivingStyle`, `parkingBrake`, `odometer`, `averageConsumption`, `ecoRange`, `ecoTip`,
  `ecoRangeWon`, `displayRangeElectricVehicle`, `electricalPowerDistribution`,
  `SOCHoldState`, `DSCActive`, `FDRControl`, `steeringWheel`, `keyPosition`, `keyNumber`.
- **engine**: `RPMSpeed`, `consumption`, `temperature`, `torque`, `status`, `info`,
  `rangeCalc`, `electricVehicleMode`.
- **sensors**: `fuel`, `battery`, `batteryTemp`, `SOCBatteryHybrid`, `temperatureExterior`,
  `temperatureInterior`, `doors`, `trunk`, `lid`, `seatbelt`, `seatOccupiedDriver`,
  `seatOccupiedPassenger`, `seatOccupiedRearLeft`, `seatOccupiedRearRight`, `PDCStatus`,
  `PDCRangeFront`, `PDCRangeRear`.
- **climate**: `ACCompressor`, `ACMode`, `ACSystemTemperatures`, `airConditionerCompressor`,
  `driverSettings`, `passengerSettings`, `residualHeat`, `seatHeatDriver`,
  `seatHeatPassenger`.
- **controls**: `lights`, `headlights`, `turnSignal`, `windshieldWiper`, `defrostRear`,
  `sunroof`, `convertibleTop`, `cruiseControl`, `parkingBrake`, `startStopStatus`,
  `startStopLEDs`, `windowDriverFront`, `windowDriverRear`, `windowPassengerFront`,
  `windowPassengerRear`.
- **communication**: `currentCallInfo`, `lastCallInfo`.
- **entertainment**: `multimedia`, `radioStation`, `radioFrequency`.
- **vehicle**: `VIN`, `type`, `brand`, `country`, `language`, `productiondate`,
  `productline`, `bordnet`, `time`, `systemTime`, `units`, `unitSpeed`.
- **hmi**: `type`, `version`, `tts`, `iDrive`, `graphicalContext`.

## Notes for app builders

- `navigation.GPSPosition` gives latitude/longitude; `navigation.GPSExtendedInfo` gives
  speed, heading and altitude, but in car-specific encodings (speed is a signed value offset
  by 32768; heading is counter-clockwise on some generations). IDriveConnectKit's
  `CDSMetrics` has the decode math.
- `hmi.tts` reports whether the car can read text aloud. `rhmi_getCapabilities` also returns
  `tts` and `navi` as capability strings at connect time.
- There is **no** property that provides the active route geometry, waypoints, or road
  closures, and nothing that accepts a route as input. Guidance can only be started toward a
  single destination via the `navigate` link action in a layout.
