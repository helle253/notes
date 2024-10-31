---
id: nv1e9lajog4b3j1g7bk7bux
title: OSInt Mapping
desc: ''
updated: 1729546448721
created: 1704308910147
published: false
---

Conversation Log:

Other Person
> for a terrain map of ukraine:
>
> - integrate acars + marine open source data to form a better picture around vehicle movements
> - integrate planet labs scans + ability to request new ones to show photographs of key areas
> - use gpt vision to describe photographs and mark key objects
> - integrate internet radio recordings, particularly around uk and ru unencrypted transmissions, using whisper+gpt to translate foreign voice to english text
> - integrate telegram group channels, converting foreign language to text, and surfacing keywords and locations using proper noun recognition
>
> basically by using only internet services you can build a pretty-accurate picture of all vehicle and troop movements on both sides and put it on a constantly updated map, then integrate search and embeddings to be able to search for specific things like "blown up buildings" or "tanks" or "minefields"

Me
> telegram/sm posts from live conflicts often (not always but enough) carry location metadata. ive wondered how often thats spoofed - iirc a case where some idf geopositioning thing was listed as over in… north sinai or something? wish i could remember more.
>
> anyways i need to read up on how transformers work and noodle on it. would be nice to be able to show hourly snapshots.

Me
> back to tbis though, its basically boils down to:
>
> 1. ingesting data from disparate sources into text for indexing + ease of use 
> 2. making datapoint indexable (eg tagging images based on subjects in view)
> 3. where possible tag location data and render on  a map of some sort
>
> none of these are trivial but they are tractable (i think)

## Data Sources

### Acars

#### Airframes

<https://app.airframes.io/messages/live>

It's cool, relies on more than just ads-b. but no stations anywhere near any place of interest

#### Flightradar

### Marine Vessel Location Data

#### [aisstream.io](https://aisstream.io/)

Tracks more than just location data:

- Binary Ship To Ship Messages
- SAR Aircraft Position
- Accident And Danger Reports
- Ship Property and Voyage Data

### Planet Labs

### Internet Radio Recordings

### Telegram Group Channels

### (Unsecured) CCTV cameras

## Ingestion Tools

### GPT Vision

## Mapping/GIS Tools

<https://6sense.com/tech/mapping-and-gis>

I have the most familiarity with Mapbox but it is not a significant familiarity. Seems like it might be useful as it provides good mobile interface support?

OTOH ArcGIS has direct PG support, which might be handy to explore.
