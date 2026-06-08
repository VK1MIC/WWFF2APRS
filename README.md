# WWFF2APRS

A Node-RED flow that polls live WWFF (World Wide Flora and Fauna) activation spots from two sources, resolves park coordinates, deduplicates across sources, and publishes APRS object packets to APRS-IS via raw TCP.

## What it does

- Caches park locations from ParksnPeaks every 12 hours (lat/lon by WWFF reference)
- Polls `spots.wwff.co` and `parksnpeaks.org` every 30 seconds for live activations
- Normalises both feeds into a common format
- Fills missing lat/lon from the park location cache
- Deduplicates spots across sources using a 5-minute window keyed on reference, callsign, mode, frequency, and time bucket
- Builds a correctly formatted APRS object packet with 9-character object name, DDHHMMz timestamp, and compressed position
- Transmits to APRS-IS via `rotate.aprs2.net:14580`
- Handles APRS-IS login on connect and re-login on reconnect
- Enforces a 10-minute per-park transmit interval to avoid flooding

## Prerequisites

- Node-RED (no additional nodes required beyond built-ins)
- A valid amateur radio callsign with APRS-IS access
- Your APRS passcode (generate at https://apps.magicbug.co.uk/passcode if needed)

## Configuration

Edit the following values in the flow before importing:

| Node | Field | Description |
|---|---|---|
| `Build raw APRS object packet` function | `APRS_CALLSIGN` | Your amateur radio callsign |
| `APRS-IS login on connect` function | `APRS_CALLSIGN` | Your amateur radio callsign |
| `APRS-IS login on connect` function | `APRS_PASS` | Your APRS-IS passcode |

These are marked as `YOUR_CALLSIGN` and `YOUR_APRS_PASSCODE` in the flow JSON.

The APRS-IS server (`rotate.aprs2.net:14580`) is a public rotating tier-2 server and requires no change for most users.

## Data sources

- WWFF spots: https://spots.wwff.co/static/spots.json (public, no auth)
- ParksnPeaks spots: https://parksnpeaks.org/api/WWFF (public, no auth)
- ParksnPeaks site locations: https://parksnpeaks.org/api/SITES/WWFF (public, no auth)

## APRS object format

Objects are transmitted using the alternate symbol table (`\`) with the object symbol (`;`), producing a fixed-point APRS object beacon. The object name is the WWFF reference truncated or padded to exactly 9 characters. The comment field includes activator callsign, frequency, mode, reference, and park name.
