# GTFS API Documentation

## Executive Summary

The General Transit Feed Specification (GTFS) is an open-standard data format designed for public transportation schedules, geographic information, and real-time service updates. Developed initially by Google in partnership with TriMet and now maintained by MobilityData, GTFS enables transit agencies to publish machine-readable data that applications can consume for trip planning, real-time tracking, and service analysis. 

GTFS comprises two distinct but complementary specifications: GTFS Schedule (static data) and GTFS Realtime (dynamic updates). This documentation provides comprehensive technical guidance for implementing, consuming, and maintaining GTFS feeds in production environments. 

## Technical Architecture

### Data Format Standards

GTFS Schedule utilizes CSV (Comma-Separated Values) text files encoded in UTF-8 without BOM (Byte Order Mark), packaged within a ZIP archive. GTFS Realtime employs Protocol Buffers (protobuf), a binary serialization format developed by Google that provides superior performance characteristics compared to XML or JSON. 

### Protocol Buffer Architecture

Protocol Buffers provide a language-neutral, platform-neutral, extensible mechanism for serializing structured data. The binary format offers: 

- Reduced payload size (3-10x smaller than XML)
- Faster parsing performance (20-100x faster than XML)
- Strongly-typed schema definitions
- Forward and backward compatibility
- Automatic code generation for multiple programming languages

The GTFS Realtime specification is defined in `gtfs-realtime.proto`, which serves as the canonical schema definition. 

### Extension Mechanism

GTFS supports extensibility through designated field ranges (1000-1999) reserved for custom extensions, allowing transit agencies to supplement standard specifications with proprietary data without breaking compatibility. 



## API Specifications

### Base URL Convention

```
https://{agency-domain}/gtfs/{feed-type}[/{entity-type}]
```

**Parameters:**
- `agency-domain`: Transit agency's API hostname
- `feed-type`: Either `static` or `realtime`
- `entity-type`: Optional realtime entity classifier (trip-updates, vehicle-positions, service-alerts)

### Authentication Mechanisms

Authentication requirements vary by provider implementation:

| Method | Usage | Security Level |
|--------|-------|----------------|
| None (public) | Open feeds with rate limiting | Low |
| API Key (query parameter) | `?api_key={key}` | Medium |
| API Key (header) | `X-API-Key: {key}` | Medium |
| OAuth 2.0 | Bearer token authentication | High |
| Client certificates | Mutual TLS authentication | Very High |

Consult individual transit agency API documentation for specific authentication requirements. 



## GTFS Schedule (Static) Specification

### Architecture Overview

GTFS Schedule datasets consist of up to 18 files organized in a hierarchical relational structure. The specification defines 7 required files for basic compliance and 11 optional files for extended functionality. 

### HTTP Endpoint

```
GET /gtfs/static.zip
GET /gtfs/google_transit.zip  (legacy naming convention)
```

### Request Headers

```http
GET /gtfs/static.zip HTTP/1.1
Host: api.transit-agency.com
Accept: application/zip, application/octet-stream
If-Modified-Since: Mon, 02 Feb 2026 08:00:00 GMT
User-Agent: TransitApp/3.2.1 (GTFS Consumer)
```

### Response Headers

```http
HTTP/1.1 200 OK
Content-Type: application/zip
Content-Length: 3458921
Last-Modified: Tue, 03 Feb 2026 04:00:00 GMT
Cache-Control: public, max-age=3600
ETag: "5f8a9c7e2b1d3e4f"
Content-Disposition: attachment; filename="gtfs_20260203.zip"
```

### Dataset Files Structure

#### Required Files

1. **agency.txt** - Transit agencies operating in the feed
2. **stops.txt** - Physical locations where vehicles pick up or drop off passengers
3. **routes.txt** - Transit routes (lines, services)
4. **trips.txt** - Individual journey instances on routes
5. **stop_times.txt** - Arrival/departure times at stops for each trip
6. **calendar.txt** - Service patterns using weekly schedules with start/end dates

#### Conditionally Required Files

7. **calendar_dates.txt** - Service exceptions (required if calendar.txt is omitted)

#### Optional Files

8. **fare_attributes.txt** - Fare information (legacy)
9. **fare_rules.txt** - Rules for applying fares (legacy)
10. **fare_media.txt** - Payment methods (Fares v2)
11. **fare_products.txt** - Purchasable fare items (Fares v2)
12. **fare_leg_rules.txt** - Fare rules for journey legs (Fares v2)
13. **fare_transfer_rules.txt** - Transfer fare policies (Fares v2)
14. **areas.txt** - Area definitions for location groups
15. **stop_areas.txt** - Assignment of stops to areas
16. **networks.txt** - Route network groupings
17. **route_networks.txt** - Route to network assignments
18. **shapes.txt** - Geographic path representations for vehicle routes 
19. **frequencies.txt** - Headway-based service schedules 
20. **transfers.txt** - Transfer rules between routes at stops
21. **pathways.txt** - Pedestrian pathways within stations 
22. **levels.txt** - Station level definitions for multi-story facilities 
23. **translations.txt** - Translations for rider-facing text
24. **feed_info.txt** - Dataset metadata and validity periods
25. **attributions.txt** - Data attribution information

### File Format Specifications

#### Document Conventions

- **File encoding:** UTF-8 without BOM
- **Field delimiter:** Comma (`,`)
- **Record terminator:** CRLF (`\r\n`) or LF (`\n`)
- **Header row:** Required as first line
- **Field enclosure:** Double quotes (`"`) for fields containing commas, quotes, or newlines
- **Quote escaping:** Double quotes within quoted fields must be escaped as `""`
- **Empty values:** Represented as consecutive delimiters (`,,`)
- **Case sensitivity:** Field names are case-sensitive
- **Column order:** Not significant; fields may appear in any order
- **Extra columns:** Additional columns are permitted but may be ignored by consumers

#### Data Type Definitions

| Type | Description | Format | Constraints |
|------|-------------|--------|-------------|
| ID | Unique identifier | Alphanumeric, underscore, hyphen | Max 255 characters |
| Text | Plain text string | UTF-8 encoded | No explicit length limit |
| URL | Fully qualified URL | Must begin with http:// or https:// | Valid URL syntax |
| Email | Email address | Standard email format | Valid email syntax |
| Phone | Telephone number | Any format | No validation required |
| Timezone | TZ database timezone | IANA timezone identifier | e.g., America/Chicago |
| Language | ISO 639-1 code | Two-letter language code | e.g., en, es, fr |
| Currency | ISO 4217 code | Three-letter currency code | e.g., USD, EUR, CAD |
| Date | Calendar date | YYYYMMDD | e.g., 20260203 |
| Time | Time of day | HH:MM:SS | 00:00:00 to 27:59:59 (allows next-day service) |
| Color | RGB color value | 6-character hexadecimal | No leading # symbol |
| Latitude | WGS84 latitude | Decimal degrees | -90.0 to 90.0 |
| Longitude | WGS84 longitude | Decimal degrees | -180.0 to 180.0 |
| Float | Floating-point number | Decimal notation | Standard float precision |
| Integer | Whole number | Numeric digits | 32-bit signed integer |
| Enum | Enumerated value | Integer code | Specific values per field |



### Comprehensive File Specifications

#### agency.txt

Defines transit agencies operating services represented in the feed.

**Primary Key:** `agency_id` (conditionally required when multiple agencies exist)

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| agency_id | ID | Conditionally Required | Uniquely identifies a transit agency. Required when dataset represents multiple agencies. | Must be unique across dataset |
| agency_name | Text | Required | Full legal or commercial name of transit agency | Maximum rider-facing visibility |
| agency_url | URL | Required | Primary website URL for transit agency | Must be fully qualified |
| agency_timezone | Timezone | Required | Timezone where agency operates | IANA timezone identifier |
| agency_lang | Language | Optional | Primary language used by agency for rider-facing information | ISO 639-1 code |
| agency_phone | Phone | Optional | Voice telephone number for customer service | Any format acceptable |
| agency_fare_url | URL | Optional | URL for purchasing tickets online | Must be fully qualified |
| agency_email | Email | Optional | Customer service email address | Valid email format |

**Example:**
```csv
agency_id,agency_name,agency_url,agency_timezone,agency_lang,agency_phone,agency_fare_url,agency_email
CTA,Chicago Transit Authority,https://www.transitchicago.com,America/Chicago,en,1-888-968-7282,https://www.ventra.com,feedback@transitchicago.com
```

**Business Rules:**
- When feed represents a single agency, `agency_id` may be omitted
- All trips must reference a valid `agency_id` (through routes)
- Agency timezone is critical for interpreting all time values in feed



#### stops.txt

Defines locations where vehicles pick up or drop off passengers, stations, station entrances/exits, and generic nodes for pathways.

**Primary Key:** `stop_id`

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| stop_id | ID | Required | Uniquely identifies a stop, station, or station entrance/exit | Must be unique across dataset |
| stop_code | Text | Optional | Short text or number identifying location for riders | Often displayed on signage |
| stop_name | Text | Conditionally Required | Name of location visible to riders. Required for `location_type` 0, 1, 2. | Avoid abbreviations when possible |
| tts_stop_name | Text | Optional | Readable version of stop name for text-to-speech | Use for pronunciation guidance |
| stop_desc | Text | Optional | Description providing useful information beyond name | Avoid redundancy with stop_name |
| stop_lat | Latitude | Conditionally Required | Latitude of stop location. Required for `location_type` 0, 1, 2. | WGS84 coordinates, -90 to 90 |
| stop_lon | Longitude | Conditionally Required | Longitude of stop location. Required for `location_type` 0, 1, 2. | WGS84 coordinates, -180 to 180 |
| zone_id | ID | Optional | Identifies fare zone for stop | Required if fare rules reference zones |
| stop_url | URL | Optional | URL of web page about location | Must be fully qualified |
| location_type | Enum | Optional | Type of location | 0=stop/platform (default), 1=station, 2=entrance/exit, 3=generic node, 4=boarding area |
| parent_station | ID | Conditionally Required | Defines hierarchy. Required for `location_type` 2, 3, 4. | Must reference `stop_id` with `location_type` 1 |
| stop_timezone | Timezone | Optional | Timezone of location | Overrides agency timezone |
| wheelchair_boarding | Enum | Optional | Wheelchair accessibility | 0=unknown, 1=accessible (some vehicles), 2=not accessible |
| level_id | ID | Optional | Level of location in station | References levels.txt |
| platform_code | Text | Optional | Platform identifier for boarding area | Visible to riders at station |

**Enumeration Details:**

**location_type:**
- `0` or empty: Stop or platform - Location where passengers board or alight from vehicles
- `1`: Station - Physical structure or area containing one or more platforms
- `2`: Entrance/Exit - Location where passengers can enter or exit a station
- `3`: Generic Node - Location within a station used for pathways
- `4`: Boarding Area - Specific area of a platform where passengers can board

**wheelchair_boarding:**
- `0` or empty: No accessibility information available
- `1`: Some vehicles at this stop can be boarded by wheelchair
- `2`: Wheelchair boarding not possible at this stop

**Example:**
```csv
stop_id,stop_code,stop_name,stop_lat,stop_lon,location_type,parent_station,wheelchair_boarding
STAT_CLARK_LAKE,40380,Clark/Lake,41.885737,-87.630886,1,,1
STOP_CLARK_BLUE_IB,30069,Clark/Lake (Blue Line - O'Hare),41.885737,-87.630886,0,STAT_CLARK_LAKE,1
ENT_CLARK_NORTH,,,41.886150,-87.630750,2,STAT_CLARK_LAKE,1
```

**Business Rules:**
- Stops with `location_type=0` may have `parent_station` referencing a station (`location_type=1`)
- Stations (`location_type=1`) must not have a `parent_station`
- Entrances, nodes, and boarding areas must have `parent_station` defined
- Geographic coordinates use WGS84 datum (EPSG:4326)
- Coordinates should be as accurate as possible (ideally within 1-2 meters)



#### routes.txt

Defines transit routes, representing a group of trips that are displayed to riders as a single service.

**Primary Key:** `route_id`

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| route_id | ID | Required | Uniquely identifies a route | Must be unique across dataset |
| agency_id | ID | Conditionally Required | Agency operating the route. Required if dataset includes multiple agencies. | References agency.txt |
| route_short_name | Text | Conditionally Required | Short name for route (e.g., "32", "Red"). Required if `route_long_name` is empty. | Maximum 6 characters recommended |
| route_long_name | Text | Conditionally Required | Full name of route. Required if `route_short_name` is empty. | Should differ from `route_short_name` |
| route_desc | Text | Optional | Description distinguishing route from others | Should differentiate similar routes |
| route_type | Enum | Required | Type of transportation used on route | See enumeration table below |
| route_url | URL | Optional | URL of web page about specific route | Must be fully qualified |
| route_color | Color | Optional | Color designation for route | 6-character hex without # |
| route_text_color | Color | Optional | Legible color for text on `route_color` background | 6-character hex without # |
| route_sort_order | Integer | Optional | Orders routes for presentation | Ascending order |
| continuous_pickup | Enum | Optional | Continuous pickup allowed along route | 0=continuous, 1=no continuous, 2=phone agency, 3=coordinate with driver |
| continuous_drop_off | Enum | Optional | Continuous drop-off allowed along route | Same values as continuous_pickup |
| network_id | ID | Optional | Network to which route belongs | References networks.txt |

**route_type Enumeration (Extended):**

| Value | Route Type | Description | Examples |
|-------|------------|-------------|----------|
| 0 | Tram, Streetcar, Light rail | Light rail surface transit | Boston Green Line, San Francisco Muni Metro |
| 1 | Subway, Metro | Underground rail service | New York Subway, London Underground |
| 2 | Rail | Intercity or long-distance rail | Amtrak, VIA Rail |
| 3 | Bus | Short- and long-distance bus service | City buses, express buses |
| 4 | Ferry | Boat service | Staten Island Ferry, BC Ferries |
| 5 | Cable tram | Street-level cable car | San Francisco Cable Car |
| 6 | Aerial lift | Gondola, aerial tramway | Roosevelt Island Tram |
| 7 | Funicular | Rail system on steep inclines | Angels Flight, Dubrovnik Cable Car |
| 11 | Trolleybus | Electric bus drawing power from overhead wires | San Francisco Trolleybus |
| 12 | Monorail | Single-rail system | Seattle Monorail, Las Vegas Monorail |

**Example:**
```csv
route_id,agency_id,route_short_name,route_long_name,route_type,route_color,route_text_color
RED,CTA,Red,Red Line,1,C60C30,FFFFFF
BLUE,CTA,Blue,Blue Line - O'Hare,1,00A1DE,FFFFFF
22,CTA,22,Clark,3,565A5C,FFFFFF
```

**Business Rules:**
- Either `route_short_name` or `route_long_name` must be specified (or both)
- `route_color` and `route_text_color` should provide sufficient contrast (WCAG AA minimum)
- When `route_type` indicates rail service (0,1,2), coordinates should follow track alignment
- Bus routes (`route_type=3`) should represent distinct rider-facing services



#### trips.txt

Defines individual journey instances on routes, representing vehicles traveling along routes at specific times.

**Primary Key:** `trip_id`

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| route_id | ID | Required | Route on which trip occurs | References routes.txt |
| service_id | ID | Required | Service calendar defining when trip operates | References calendar.txt or calendar_dates.txt |
| trip_id | ID | Required | Uniquely identifies a trip | Must be unique across dataset |
| trip_headsign | Text | Optional | Text appearing on vehicle signage identifying trip destination | Visible to riders |
| trip_short_name | Text | Optional | Public-facing identifier for trip | Often train/flight number |
| direction_id | Enum | Optional | Direction of travel for trip | 0 or 1 (must be consistent per route) |
| block_id | ID | Optional | Identifies block of sequential trips made by same vehicle | Used for transfer optimization |
| shape_id | ID | Optional | Geographic path vehicle travels on trip | References shapes.txt |
| wheelchair_accessible | Enum | Optional | Wheelchair accessibility of vehicle used on trip | 0=unknown, 1=accessible, 2=not accessible |
| bikes_allowed | Enum | Optional | Whether bicycles are allowed on trip | 0=unknown, 1=allowed, 2=not allowed |

**Enumeration Details:**

**direction_id:**
- `0`: Travel in one direction (e.g., outbound, northbound)
- `1`: Travel in opposite direction (e.g., inbound, southbound)

**Note:** Values are arbitrary but must be consistent across all trips for a route. Different routes may use opposite conventions.

**wheelchair_accessible:**
- `0` or empty: No accessibility information
- `1`: Vehicle on this trip can accommodate at least one wheelchair
- `2`: No wheelchairs can be accommodated

**bikes_allowed:**
- `0` or empty: No bicycle information
- `1`: Vehicle on this trip can accommodate at least one bicycle
- `2`: No bicycles allowed

**Example:**
```csv
route_id,service_id,trip_id,trip_headsign,direction_id,block_id,shape_id,wheelchair_accessible,bikes_allowed
RED,WEEKDAY,RED_20260203_WD_1234,95th/Dan Ryan,1,RED_BLK_01,RED_SHAPE_SB,1,1
BLUE,WEEKDAY,BLUE_20260203_WD_5678,O'Hare,0,BLUE_BLK_02,BLUE_SHAPE_NB,1,1
```

**Business Rules:**
- All trips on a route with `direction_id` specified should have values assigned
- `block_id` enables "seated transfer" identification where passenger remains on vehicle
- `shape_id` should accurately represent actual vehicle path, not airline distance
- Trips operate on dates defined by referenced `service_id`



#### stop_times.txt

Defines arrival and departure times for each stop on each trip. This file is typically the largest in a GTFS dataset.

**Primary Key:** (`trip_id`, `stop_sequence`)

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| trip_id | ID | Required | Trip containing this stop time | References trips.txt |
| arrival_time | Time | Conditionally Required | Arrival time at stop. Required for first and last stop, required for timepoints. | HH:MM:SS format, allows >24 hours |
| departure_time | Time | Conditionally Required | Departure time from stop. Required for first and last stop, required for timepoints. | HH:MM:SS format, allows >24 hours |
| stop_id | ID | Required | Stop where time occurs | References stops.txt |
| stop_sequence | Integer | Required | Order of stop in trip | Non-negative, ascending |
| stop_headsign | Text | Optional | Headsign text overriding trip headsign for this stop | Visible to riders |
| pickup_type | Enum | Optional | Pickup availability at stop | 0=regular, 1=none, 2=phone agency, 3=coordinate with driver |
| drop_off_type | Enum | Optional | Drop-off availability at stop | Same values as pickup_type |
| continuous_pickup | Enum | Optional | Continuous pickup allowed between this and next stop | 0=continuous, 1=no continuous, 2=phone agency, 3=coordinate with driver |
| continuous_drop_off | Enum | Optional | Continuous drop-off allowed between this and next stop | Same values as continuous_pickup |
| shape_dist_traveled | Float | Optional | Distance along shape from first stop | Meters or feet (consistent units) |
| timepoint | Enum | Optional | Whether times are exact or approximate | 0=approximate, 1=exact (default) |

**Time Format Specifications:**
- Format: `HH:MM:SS` (e.g., `14:30:00` for 2:30 PM)
- Hours may exceed 24 for service after midnight (e.g., `25:30:00` for 1:30 AM next day)
- Hours, minutes, and seconds must be zero-padded
- Times are relative to service day, not calendar day
- Times use 24-hour clock (no AM/PM)

**Example:**
```csv
trip_id,arrival_time,departure_time,stop_id,stop_sequence,stop_headsign,pickup_type,drop_off_type,shape_dist_traveled,timepoint
RED_20260203_WD_1234,08:00:00,08:00:00,HOWARD,1,,0,0,0.0,1
RED_20260203_WD_1234,08:03:45,08:04:00,JARVIS,2,,0,0,1247.3,1
RED_20260203_WD_1234,08:06:30,08:06:45,MORSE,3,,0,0,2183.7,1
RED_20260203_WD_1234,08:45:00,08:45:00,95TH,28,,0,0,38421.5,1
```

**Business Rules:**
- `stop_sequence` must increase along trip but need not be consecutive (allows gaps for schedule flexibility)
- If no `arrival_time` or `departure_time` specified for intermediate stops, times are interpolated
- `timepoint=1` indicates scheduled time that must be met; `timepoint=0` indicates estimated time
- `shape_dist_traveled` must be non-decreasing along trip when specified
- First and last stop must have both `arrival_time` and `departure_time`
- For intermediate stops, if only one time is specified, it applies to both arrival and departure



#### calendar.txt

Defines service patterns using weekly schedules with start and end dates. Represents regular recurring service.

**Primary Key:** `service_id`

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| service_id | ID | Required | Uniquely identifies a set of dates when service is available | Must be unique across dataset |
| monday | Enum | Required | Service operates on Mondays | 0=no service, 1=service available |
| tuesday | Enum | Required | Service operates on Tuesdays | 0=no service, 1=service available |
| wednesday | Enum | Required | Service operates on Wednesdays | 0=no service, 1=service available |
| thursday | Enum | Required | Service operates on Thursdays | 0=no service, 1=service available |
| friday | Enum | Required | Service operates on Fridays | 0=no service, 1=service available |
| saturday | Enum | Required | Service operates on Saturdays | 0=no service, 1=service available |
| sunday | Enum | Required | Service operates on Sundays | 0=no service, 1=service available |
| start_date | Date | Required | First date of service availability | YYYYMMDD format |
| end_date | Date | Required | Last date of service availability | YYYYMMDD format, must be >= start_date |

**Example:**
```csv
service_id,monday,tuesday,wednesday,thursday,friday,saturday,sunday,start_date,end_date
WEEKDAY,1,1,1,1,1,0,0,20260201,20260630
SATURDAY,0,0,0,0,0,1,0,20260201,20260630
SUNDAY,0,0,0,0,0,0,1,20260201,20260630
```

**Business Rules:**
- Service dates are inclusive (both start_date and end_date operate if day of week matches)
- At least one day of week must have service (`1` value)
- Feed should have valid service for minimum 7 days, recommended 30+ days 
- Dataset validity should not exceed one year 
- Combine with calendar_dates.txt to define exceptions (holidays, special events)



#### calendar_dates.txt

Defines service exceptions - dates when service differs from regular patterns. Can represent service additions or removals.

**Primary Key:** (`service_id`, `date`)

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| service_id | ID | Required | Service identifier affected by exception | References calendar.txt or standalone service |
| date | Date | Required | Date when exception occurs | YYYYMMDD format |
| exception_type | Enum | Required | Type of exception | 1=service added, 2=service removed |

**exception_type:**
- `1`: Service has been added for specified date (operates even if calendar.txt indicates no service)
- `2`: Service has been removed for specified date (does not operate even if calendar.txt indicates service)

**Example:**
```csv
service_id,date,exception_type
WEEKDAY,20260216,2
WEEKDAY,20260528,2
SATURDAY,20260704,2
SUNDAY_HOLIDAY,20260704,1
```

**Business Rules:**
- If dataset includes no calendar.txt, calendar_dates.txt defines all service dates
- calendar_dates.txt takes precedence over calendar.txt
- Commonly used for holidays, special events, and service disruptions
- Can define entirely date-based services without calendar.txt weekly patterns



#### shapes.txt

Defines geographic paths that vehicles travel, enabling accurate visualization of routes on maps. 

**Primary Key:** (`shape_id`, `shape_pt_sequence`)

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| shape_id | ID | Required | Identifies a shape | Referenced by trips.txt |
| shape_pt_lat | Latitude | Required | Latitude of shape point | WGS84 coordinates, -90 to 90 |
| shape_pt_lon | Longitude | Required | Longitude of shape point | WGS84 coordinates, -180 to 180 |
| shape_pt_sequence | Integer | Required | Order of point in shape | Non-negative, ascending |
| shape_dist_traveled | Float | Optional | Distance from first point | Meters or feet (consistent units) |

**Example:**
```csv
shape_id,shape_pt_lat,shape_pt_lon,shape_pt_sequence,shape_dist_traveled
RED_SHAPE_SB,42.019063,-87.672892,1,0.0
RED_SHAPE_SB,42.018562,-87.672889,2,55.7
RED_SHAPE_SB,42.018021,-87.672886,3,116.0
RED_SHAPE_SB,41.720489,-87.625336,528,38421.5
```

**Best Practices:**
- Use GPS traces from actual vehicle paths when available 
- Points should align with actual travel path, not stop locations
- Provide sufficient density to accurately represent curves and turns
- Typical spacing: 50-200 meters on straight sections, 10-20 meters on curves
- Must use WGS84 coordinate system (EPSG:4326) 
- `shape_dist_traveled` should match units in stop_times.txt



#### frequencies.txt

Defines headway-based service where trips run at regular intervals rather than fixed schedules. 

**Primary Key:** (`trip_id`, `start_time`)

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| trip_id | ID | Required | Trip operating at specified frequency | References trips.txt |
| start_time | Time | Required | Time when frequency service begins | HH:MM:SS format |
| end_time | Time | Required | Time when frequency service ends | HH:MM:SS format, must be > start_time |
| headway_secs | Integer | Required | Time between departures | Seconds between consecutive departures |
| exact_times | Enum | Optional | Type of frequency service | 0=frequency-based (default), 1=schedule-based |

**exact_times:**
- `0` or empty: Frequency-based trips (vehicle departs every `headway_secs` starting at `start_time`)
- `1`: Schedule-based trips with exact times (times from stop_times.txt are exact)

**Example:**
```csv
trip_id,start_time,end_time,headway_secs,exact_times
BLUE_TEMPLATE_RUSH,06:00:00,09:00:00,300,0
BLUE_TEMPLATE_MIDDAY,09:00:00,15:00:00,600,0
BLUE_TEMPLATE_EVENING,15:00:00,18:00:00,300,0
```

**Business Rules:**
- When frequencies.txt defines a trip, it represents a pattern, not a single journey
- Trips with `exact_times=0` have arrival/departure times calculated dynamically
- Headway represents service frequency (e.g., 300 seconds = 5-minute headway)
- Multiple frequency periods can be defined for same trip_id



#### transfers.txt

Defines rules for making connections between routes at stops.

**Primary Key:** (`from_stop_id`, `to_stop_id`, `from_route_id`, `to_route_id`, `from_trip_id`, `to_trip_id`)

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| from_stop_id | ID | Conditionally Required | Stop where connection begins. Required if `from_trip_id` not defined. | References stops.txt |
| to_stop_id | ID | Conditionally Required | Stop where connection ends. Required if `to_trip_id` not defined. | References stops.txt |
| from_route_id | ID | Optional | Route where connection begins | References routes.txt |
| to_route_id | ID | Optional | Route where connection ends | References routes.txt |
| from_trip_id | ID | Optional | Trip where connection begins | References trips.txt |
| to_trip_id | ID | Optional | Trip where connection ends | References trips.txt |
| transfer_type | Enum | Required | Type of connection | 0=recommended, 1=timed, 2=min time, 3=not possible, 4=in-seat, 5=in-seat not allowed |
| min_transfer_time | Integer | Optional | Minimum time required for transfer | Seconds |

**transfer_type:**
- `0`: Recommended transfer point
- `1`: Timed transfer (departing vehicle waits for arriving vehicle)
- `2`: Requires minimum time defined in `min_transfer_time` for safe transfer
- `3`: Transfer not possible between routes at this location
- `4`: In-seat transfer (passengers remain on vehicle, defined by block_id)
- `5`: In-seat transfer not allowed despite same block_id

**Example:**
```csv
from_stop_id,to_stop_id,from_route_id,to_route_id,transfer_type,min_transfer_time
CLARK,CLARK,RED,BLUE,0,180
WASHINGTON,WASHINGTON,RED,ORANGE,2,240
```



#### pathways.txt

Defines pedestrian pathways within stations, essential for accessibility and wayfinding. 

**Primary Key:** `pathway_id`

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| pathway_id | ID | Required | Uniquely identifies pathway | Must be unique across dataset |
| from_stop_id | ID | Required | Location where pathway begins | References stops.txt (location_type 1,2,3,4) |
| to_stop_id | ID | Required | Location where pathway ends | References stops.txt (location_type 1,2,3,4) |
| pathway_mode | Enum | Required | Type of pathway | 1=walkway, 2=stairs, 3=moving sidewalk, 4=escalator, 5=elevator, 6=fare gate, 7=exit gate |
| is_bidirectional | Enum | Required | Pathway can be used in both directions | 0=unidirectional, 1=bidirectional |
| length | Float | Optional | Horizontal pathway length | Meters (non-negative) |
| traversal_time | Integer | Optional | Average time to traverse pathway | Seconds |
| stair_count | Integer | Optional | Number of stairs | Positive for up, negative for down |
| max_slope | Float | Optional | Maximum slope of pathway | Ratio (e.g., 0.083 for 1:12 slope) |
| min_width | Float | Optional | Minimum pathway width | Meters |
| signposted_as | Text | Optional | Text on physical signage | For wayfinding |
| reversed_signposted_as | Text | Optional | Signage text in reverse direction | For bidirectional pathways |

**Example:**
```csv
pathway_id,from_stop_id,to_stop_id,pathway_mode,is_bidirectional,length,traversal_time,stair_count
PATH_01,ENT_MAIN,NODE_CONCOURSE,1,1,45.0,60,
PATH_02,NODE_CONCOURSE,NODE_PLATFORM_LEVEL,2,1,15.0,90,20
PATH_03,NODE_PLATFORM_LEVEL,PLATFORM_NB,1,1,30.0,40,
```



#### fare_attributes.txt (Legacy Fares v1)

Defines fare information for routes. 

**Primary Key:** `fare_id`

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| fare_id | ID | Required | Uniquely identifies fare class | Must be unique |
| price | Float | Required | Fare price | Currency amount |
| currency_type | Currency | Required | Currency of payment | ISO 4217 code |
| payment_method | Enum | Required | When fare must be paid | 0=on board, 1=before boarding |
| transfers | Enum | Required | Number of transfers permitted | 0=no transfers, 1=once, 2=twice, (empty)=unlimited |
| agency_id | ID | Optional | Agency for fare | References agency.txt |
| transfer_duration | Integer | Optional | Length of time in seconds before transfer expires | Seconds |

**Example:**
```csv
fare_id,price,currency_type,payment_method,transfers,transfer_duration
REGULAR_FARE,2.50,USD,1,,7200
DAY_PASS,10.00,USD,1,,86400
```



#### fare_products.txt (Fares v2)

Defines purchasable fare products in modern fare system. 

**Primary Key:** (`fare_product_id`, `fare_media_id`)

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| fare_product_id | ID | Required | Identifies a fare product | Must be unique per fare_media_id |
| fare_product_name | Text | Optional | Customer-facing name of fare product | For display |
| fare_media_id | ID | Optional | Fare media on which product can be used | References fare_media.txt |
| amount | Float | Required | Product price | Currency amount |
| currency | Currency | Required | Currency of amount | ISO 4217 code |



#### fare_leg_rules.txt (Fares v2)

Defines fare rules for individual journey legs. 

**Primary Key:** `leg_group_id` or combination of specific fields

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| leg_group_id | ID | Optional | Identifies group of legs | For grouping rules |
| network_id | ID | Optional | Network for leg | References networks.txt |
| from_area_id | ID | Optional | Origin area | References areas.txt |
| to_area_id | ID | Optional | Destination area | References areas.txt |
| fare_product_id | ID | Required | Fare product for leg | References fare_products.txt |



#### fare_transfer_rules.txt (Fares v2)

Defines fare transfer policies. 

**Primary Key:** (`from_leg_group_id`, `to_leg_group_id`)

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| from_leg_group_id | ID | Optional | Leg group starting transfer | References fare_leg_rules.txt |
| to_leg_group_id | ID | Optional | Leg group ending transfer | References fare_leg_rules.txt |
| transfer_count | Integer | Optional | Maximum number of transfers | -1=unlimited |
| duration_limit | Integer | Optional | Transfer window duration | Seconds |
| duration_limit_type | Enum | Optional | How duration is calculated | 0=departure to departure, 1=departure to arrival, 2=arrival to departure, 3=arrival to arrival |
| fare_transfer_type | Enum | Required | Cost structure of transfer | 0=sum of fares, 1=flat fee, 2=no cost |
| fare_product_id | ID | Optional | Transfer fee product | References fare_products.txt |

**Example showing 90-minute free transfer window:**
```csv
from_leg_group_id,to_leg_group_id,transfer_count,duration_limit,duration_limit_type,fare_transfer_type
LOCAL_BUS,LOCAL_BUS,-1,5400,1,0
```



#### feed_info.txt

Provides metadata about the GTFS dataset itself.

**Primary Key:** None (single-row file)

| Field Name | Type | Required | Description | Constraints |
|------------|------|----------|-------------|-------------|
| feed_publisher_name | Text | Required | Organization publishing feed | Full legal name |
| feed_publisher_url | URL | Required | Publisher website | Must be fully qualified |
| feed_lang | Language | Required | Default language for text in feed | ISO 639-1 code |
| default_lang | Language | Optional | Default language when consumer language unavailable | ISO 639-1 code |
| feed_start_date | Date | Optional | First date of service in feed | YYYYMMDD format |
| feed_end_date | Date | Optional | Last date of service in feed | YYYYMMDD format |
| feed_version | Text | Optional | Version identifier for feed | Publisher-defined |
| feed_contact_email | Email | Optional | Contact email for feed issues | Valid email format |
| feed_contact_url | URL | Optional | Contact URL for feed issues | Must be fully qualified |

**Example:**
```csv
feed_publisher_name,feed_publisher_url,feed_lang,feed_start_date,feed_end_date,feed_version,feed_contact_email
Chicago Transit Authority,https://www.transitchicago.com,en,20260201,20260630,2026.02.03,gtfs@transitchicago.com
```

**Best Practices:**
- Update `feed_version` with each publication 
- Set `feed_start_date` and `feed_end_date` to actual service validity period 
- Provide contact information for data consumers to report issues



## GTFS Realtime Specification

### Architecture Overview

GTFS Realtime provides continuous updates about transit system state using Protocol Buffers for efficient binary serialization. The specification defines three primary entity types delivered through distinct or combined feeds. 

### Protocol Buffer Schema Version

Current specification version: **2.0**
Protocol Buffer syntax: **proto2**
Schema definition file: **gtfs-realtime.proto**

### HTTP Endpoints

```
GET /gtfs/realtime/trip-updates
GET /gtfs/realtime/vehicle-positions  
GET /gtfs/realtime/service-alerts
GET /gtfs/realtime/combined  (all entity types)
```

### Request Specifications

**Headers:**
```http
GET /gtfs/realtime/trip-updates HTTP/1.1
Host: api.transit-agency.com
Accept: application/x-protobuf, application/octet-stream
Accept-Encoding: gzip, deflate
If-Modified-Since: Tue, 03 Feb 2026 17:40:00 GMT
User-Agent: TransitTracker/2.1.0
X-API-Key: {api_key}  (if required)
```

### Response Specifications

**Headers:**
```http
HTTP/1.1 200 OK
Content-Type: application/x-protobuf
Content-Length: 28491
Content-Encoding: gzip
Last-Modified: Tue, 03 Feb 2026 17:41:00 GMT
Cache-Control: max-age=30, must-revalidate
ETag: "abc123def456"
Access-Control-Allow-Origin: *
```

### Core Message Structure

#### FeedMessage

Root message containing all realtime data.

```protobuf
message FeedMessage {
  required FeedHeader header = 1;
  repeated FeedEntity entity = 2;
  extensions 1000 to 1999;
}
```

**Fields:**
- `header` (required): Metadata about the feed
- `entity` (repeated): Array of feed entities (trip updates, vehicle positions, or alerts)
- `extensions` (1000-1999): Reserved for custom extensions 



#### FeedHeader

Metadata describing the feed contents.

```protobuf
message FeedHeader {
  required string gtfs_realtime_version = 1;
  enum Incrementality {
    FULL_DATASET = 0;
    DIFFERENTIAL = 1;
  }
  optional Incrementality incrementality = 2 [default = FULL_DATASET];
  optional uint64 timestamp = 3;
  extensions 1000 to 1999;
}
```

**Fields:**
- `gtfs_realtime_version` (required): Version of GTFS Realtime specification (e.g., "2.0")
- `incrementality` (optional): Feed update strategy 
  - `FULL_DATASET` (0): Complete data snapshot (default)
  - `DIFFERENTIAL` (1): Only changed entities included
- `timestamp` (optional): Unix timestamp (epoch seconds) when feed generated 
- `extensions` (1000-1999): Reserved for custom extensions

**Best Practices:**
- Always set `timestamp` to feed generation time 
- Timestamp must not decrease between sequential feed iterations 
- Use `FULL_DATASET` unless implementing differential updates with proper invalidation logic



#### FeedEntity

Container for individual realtime entities.

```protobuf
message FeedEntity {
  required string id = 1;
  optional bool is_deleted = 2 [default = false];
  optional TripUpdate trip_update = 3;
  optional VehiclePosition vehicle = 4;
  optional Alert alert = 5;
  extensions 1000 to 1999;
}
```

**Fields:**
- `id` (required): Unique identifier for entity within feed
- `is_deleted` (optional): Indicates entity removal in differential feed
- `trip_update` (optional): Trip update entity
- `vehicle` (optional): Vehicle position entity
- `alert` (optional): Service alert entity

**Constraints:**
- Exactly one of `trip_update`, `vehicle`, or `alert` must be populated
- Entity `id` must be unique within feed instance
- When using `DIFFERENTIAL` incrementality, `is_deleted=true` removes previously sent entity



### Entity Type: Trip Updates

Provides real-time predictions for arrivals, departures, and service disruptions.

#### TripUpdate Message

```protobuf
message TripUpdate {
  required TripDescriptor trip = 1;
  optional VehicleDescriptor vehicle = 2;
  repeated StopTimeUpdate stop_time_update = 3;
  optional uint64 timestamp = 4;
  optional int32 delay = 5;
  enum TripUpdateScheduleRelationship {
    SCHEDULED = 0;
    ADDED = 1;
    UNSCHEDULED = 2;
    CANCELED = 3;
    DUPLICATED = 5;
    DELETED = 6;
  }
  optional TripUpdateScheduleRelationship trip_properties = 6;
  extensions 1000 to 1999;
}
```

**Fields:**
- `trip` (required): Identifies affected trip from GTFS Schedule
- `vehicle` (optional): Vehicle servicing trip 
- `stop_time_update` (repeated): Predictions for individual stops
- `timestamp` (optional): Unix timestamp when prediction generated
- `delay` (optional): Current delay in seconds (positive=late, negative=early) 
- `trip_properties` (optional): Relationship to schedule
  - `SCHEDULED` (0): Trip running normally
  - `ADDED` (1): Extra trip not in static schedule
  - `UNSCHEDULED` (2): Trip running without fixed schedule
  - `CANCELED` (3): Trip canceled, no service
  - `DUPLICATED` (5): Additional trip duplicating existing service
  - `DELETED` (6): Trip removed from service



#### TripDescriptor

Identifies a specific trip from GTFS Schedule.

```protobuf
message TripDescriptor {
  optional string trip_id = 1;
  optional string route_id = 2;
  optional uint32 direction_id = 3;
  optional string start_time = 4;
  optional string start_date = 5;
  enum ScheduleRelationship {
    SCHEDULED = 0;
    ADDED = 1;
    UNSCHEDULED = 2;
    CANCELED = 3;
    DUPLICATED = 5;
  }
  optional ScheduleRelationship schedule_relationship = 6;
  extensions 1000 to 1999;
}
```

**Fields:**
- `trip_id` (optional): References trips.txt from GTFS Schedule 
- `route_id` (optional): References routes.txt (required if trip_id omitted)
- `direction_id` (optional): References trips.txt direction_id
- `start_time` (optional): Initial departure time (HH:MM:SS) for frequency-based trips
- `start_date` (optional): Service date (YYYYMMDD) when trip operates
- `schedule_relationship` (optional): Relationship to static schedule

**Identification Rules:**
- For scheduled trips: Provide `trip_id` or combination of `route_id`, `start_time`, `start_date`
- For frequency-based trips: Provide `trip_id`, `start_time`, `start_date`
- All IDs must exactly match GTFS Schedule identifiers 



#### StopTimeUpdate

Prediction for individual stop within trip.

```protobuf
message StopTimeUpdate {
  optional uint32 stop_sequence = 1;
  optional string stop_id = 4;
  optional StopTimeEvent arrival = 2;
  optional StopTimeEvent departure = 3;
  enum ScheduleRelationship {
    SCHEDULED = 0;
    SKIPPED = 1;
    NO_DATA = 2;
    UNSCHEDULED = 3;
  }
  optional ScheduleRelationship schedule_relationship = 5 [default = SCHEDULED];
  extensions 1000 to 1999;
}
```

**Fields:**
- `stop_sequence` (optional): References stop_times.txt stop_sequence
- `stop_id` (optional): References stops.txt (required if stop_sequence omitted)
- `arrival` (optional): Predicted arrival
- `departure` (optional): Predicted departure
- `schedule_relationship` (optional):
  - `SCHEDULED` (0): Normal service at stop
  - `SKIPPED` (1): Vehicle skips this stop
  - `NO_DATA` (2): No prediction available
  - `UNSCHEDULED` (3): Unscheduled stop

**Rules:**
- At least one of `stop_sequence` or `stop_id` must be provided
- If only arrival or departure provided, both are assumed equal
- Must provide either `stop_sequence` (preferred) or `stop_id`



#### StopTimeEvent

Predicted arrival or departure time.

```protobuf
message StopTimeEvent {
  optional int32 delay = 1;
  optional int64 time = 2;
  optional int32 uncertainty = 3;
  extensions 1000 to 1999;
}
```

**Fields:**
- `delay` (optional): Seconds of delay relative to schedule (positive=late, negative=early) 
- `time` (optional): Absolute predicted time as Unix timestamp
- `uncertainty` (optional): Prediction uncertainty in seconds

**Usage Patterns:**
- Provide either `delay` (relative to schedule) OR `time` (absolute prediction)
- When providing `delay`, scheduled time from stop_times.txt is used as baseline 
- `uncertainty` represents confidence interval (e.g., ±60 seconds)

**Best Practice:**
Always provide `delay` field when making arrival/departure predictions. 



### Entity Type: Vehicle Positions

Provides real-time geographic location and status of vehicles.

#### VehiclePosition Message

```protobuf
message VehiclePosition {
  optional TripDescriptor trip = 1;
  optional VehicleDescriptor vehicle = 2;
  optional Position position = 3;
  optional uint32 current_stop_sequence = 4;
  optional string stop_id = 5;
  enum VehicleStopStatus {
    INCOMING_AT = 0;
    STOPPED_AT = 1;
    IN_TRANSIT_TO = 2;
  }
  optional VehicleStopStatus current_status = 6 [default = IN_TRANSIT_TO];
  optional uint64 timestamp = 7;
  enum CongestionLevel {
    UNKNOWN_CONGESTION_LEVEL = 0;
    RUNNING_SMOOTHLY = 1;
    STOP_AND_GO = 2;
    CONGESTION = 3;
    SEVERE_CONGESTION = 4;
  }
  optional CongestionLevel congestion_level = 8;
  enum OccupancyStatus {
    EMPTY = 0;
    MANY_SEATS_AVAILABLE = 1;
    FEW_SEATS_AVAILABLE = 2;
    STANDING_ROOM_ONLY = 3;
    CRUSHED_STANDING_ROOM_ONLY = 4;
    FULL = 5;
    NOT_ACCEPTING_PASSENGERS = 6;
    NO_DATA_AVAILABLE = 7;
    NOT_BOARDABLE = 8;
  }
  optional OccupancyStatus occupancy_status = 9;
  optional uint32 occupancy_percentage = 10;
  repeated CarriageDetails multi_carriage_details = 11;
  extensions 1000 to 1999;
}
```

**Fields:**
- `trip` (optional): Trip vehicle is servicing 
- `vehicle` (optional): Identifies specific vehicle 
- `position` (optional): Geographic coordinates and bearing
- `current_stop_sequence` (optional): Stop vehicle is at or approaching
- `stop_id` (optional): Stop ID vehicle is at or approaching
- `current_status` (optional): Vehicle status relative to stop
  - `INCOMING_AT` (0): Vehicle approaching stop
  - `STOPPED_AT` (1): Vehicle stopped at stop
  - `IN_TRANSIT_TO` (2): Vehicle traveling toward stop (default)
- `timestamp` (optional): Unix timestamp of position measurement
- `congestion_level` (optional): Traffic congestion affecting vehicle 
- `occupancy_status` (optional): Passenger occupancy level
- `occupancy_percentage` (optional): Percentage of capacity (0-100)
- `multi_carriage_details` (optional): Per-carriage details for trains



#### VehicleDescriptor

Identifies a specific vehicle.

```protobuf
message VehicleDescriptor {
  optional string id = 1;
  optional string label = 2;
  optional string license_plate = 3;
  extensions 1000 to 1999;
}
```

**Fields:**
- `id` (optional): Internal vehicle identifier
- `label` (optional): User-visible vehicle identifier (e.g., vehicle number shown to passengers) 
- `license_plate` (optional): Vehicle license plate number

**Best Practices:**
- Each vehicle must have consistent identifier across all updates 
- Use `id` for internal tracking, `label` for passenger-facing display
- Vehicle identifier should remain constant for vehicle's operational lifetime



#### Position

Geographic location and movement data.

```protobuf
message Position {
  required float latitude = 1;
  required float longitude = 2;
  optional float bearing = 3;
  optional double odometer = 4;
  optional float speed = 5;
  extensions 1000 to 1999;
}
```

**Fields:**
- `latitude` (required): WGS84 latitude in decimal degrees (-90 to 90)
- `longitude` (required): WGS84 longitude in decimal degrees (-180 to 180)
- `bearing` (optional): Compass direction in degrees (0-360, 0=North, 90=East)
- `odometer` (optional): Total distance traveled by vehicle in meters
- `speed` (optional): Instantaneous speed in meters/second

**Accuracy Requirements:**
- Position should be accurate to within 10 meters when possible
- Update frequency: Every 30-90 seconds recommended 
- Bearing should indicate direction of travel, not vehicle orientation
- Use WGS84 coordinate system (EPSG:4326)



### Entity Type: Service Alerts

Provides information about service disruptions, delays, and other incidents.

#### Alert Message

```protobuf
message Alert {
  repeated TimeRange active_period = 1;
  repeated EntitySelector informed_entity = 2;
  enum Cause {
    UNKNOWN_CAUSE = 1;
    OTHER_CAUSE = 2;
    TECHNICAL_PROBLEM = 3;
    STRIKE = 4;
    DEMONSTRATION = 5;
    ACCIDENT = 6;
    HOLIDAY = 7;
    WEATHER = 8;
    MAINTENANCE = 9;
    CONSTRUCTION = 10;
    POLICE_ACTIVITY = 11;
    MEDICAL_EMERGENCY = 12;
  }
  optional Cause cause = 3 [default = UNKNOWN_CAUSE];
  enum Effect {
    NO_SERVICE = 1;
    REDUCED_SERVICE = 2;
    SIGNIFICANT_DELAYS = 3;
    DETOUR = 4;
    ADDITIONAL_SERVICE = 5;
    MODIFIED_SERVICE = 6;
    OTHER_EFFECT = 7;
    UNKNOWN_EFFECT = 8;
    STOP_MOVED = 9;
    NO_EFFECT = 10;
    ACCESSIBILITY_ISSUE = 11;
  }
  optional Effect effect = 4 [default = UNKNOWN_EFFECT];
  optional TranslatedString url = 5;
  optional TranslatedString header_text = 6;
  optional TranslatedString description_text = 7;
  optional TranslatedString tts_header_text = 8;
  optional TranslatedString tts_description_text = 9;
  enum SeverityLevel {
    UNKNOWN_SEVERITY = 1;
    INFO = 2;
    WARNING = 3;
    SEVERE = 4;
  }
  optional SeverityLevel severity_level = 10 [default = UNKNOWN_SEVERITY];
  extensions 1000 to 1999;
}
```

**Fields:**
- `active_period` (repeated): Time ranges when alert is active (empty=always active)
- `informed_entity` (repeated): Transit entities affected by alert
- `cause` (optional): Reason for alert
- `effect` (optional): Impact on service
- `url` (optional): URL with more information
- `header_text` (optional): Brief alert summary (160 characters recommended)
- `description_text` (optional): Detailed description
- `tts_header_text` (optional): Text-to-speech version of header
- `tts_description_text` (optional): Text-to-speech version of description
- `severity_level` (optional): Alert severity



#### TimeRange

Defines time period for alert applicability.

```protobuf
message TimeRange {
  optional uint64 start = 1;
  optional uint64 end = 2;
  extensions 1000 to 1999;
}
```

**Fields:**
- `start` (optional): Unix timestamp when alert becomes active (empty=beginning of time)
- `end` (optional): Unix timestamp when alert ends (empty=indefinite)



#### EntitySelector

Identifies transit entities affected by alert.

```protobuf
message EntitySelector {
  optional string agency_id = 1;
  optional string route_id = 2;
  optional int32 route_type = 3;
  optional TripDescriptor trip = 4;
  optional string stop_id = 5;
  extensions 1000 to 1999;
}
```

**Fields:**
- `agency_id` (optional): Affects all services by agency
- `route_id` (optional): Affects specific route
- `route_type` (optional): Affects all routes of type
- `trip` (optional): Affects specific trip
- `stop_id` (optional): Affects specific stop

**Selection Logic:**
- Empty EntitySelector = affects entire transit system
- Multiple fields = intersection (AND logic)
- Multiple EntitySelector messages = union (OR logic)



#### TranslatedString

Supports internationalization with multiple language translations.

```protobuf
message TranslatedString {
  message Translation {
    required string text = 1;
    optional string language = 2;
    extensions 1000 to 1999;
  }
  repeated Translation translation = 1;
  extensions 1000 to 1999;
}
```

**Fields:**
- `translation` (repeated): Array of translations
- `text` (required): Translated text
- `language` (optional): ISO 639-1 language code (defaults to feed_lang from feed_info.txt)



## Implementation Guidelines

### Polling Strategy

**Trip Updates and Vehicle Positions:**
- Recommended polling interval: 30-90 seconds 
- Maximum data age: 90 seconds 
- Use HTTP `If-Modified-Since` header to avoid unnecessary transfers 

**Service Alerts:**
- Recommended polling interval: 5-10 minutes 
- Maximum data age: 10 minutes 

**Implementation Pattern:**
```python
import time
import requests
from google.transit import gtfs_realtime_pb2

FEED_URL = "https://api.transit-agency.com/gtfs/realtime/vehicle-positions"
POLL_INTERVAL = 30  # seconds
last_modified = None

while True:
    headers = {}
    if last_modified:
        headers['If-Modified-Since'] = last_modified
    
    response = requests.get(FEED_URL, headers=headers)
    
    if response.status_code == 200:
        feed = gtfs_realtime_pb2.FeedMessage()
        feed.ParseFromString(response.content)
        
        # Process feed
        print(f"Timestamp: {feed.header.timestamp}")
        for entity in feed.entity:
            if entity.HasField('vehicle'):
                vehicle = entity.vehicle
                if vehicle.HasField('position'):
                    print(f"Vehicle {vehicle.vehicle.id}: "
                          f"{vehicle.position.latitude}, "
                          f"{vehicle.position.longitude}")
        
        last_modified = response.headers.get('Last-Modified')
    
    elif response.status_code == 304:
        print("Feed not modified")
    
    time.sleep(POLL_INTERVAL)
```



### Data Validation

**Critical Validation Rules:**

1. **ID Consistency**: All `trip_id`, `route_id`, `stop_id` in realtime must match GTFS Schedule exactly 
2. **Timestamp Monotonicity**: Feed timestamps must not decrease 
3. **Coordinate Validity**: Latitude (-90 to 90), Longitude (-180 to 180)
4. **Field Presence Checking**: Use Protocol Buffer `HasField()` method before accessing optional fields 

**Example Validation:**
```python
def validate_trip_update(trip_update):
    """Validates trip update against GTFS Schedule."""
    errors = []
    
    # Check required fields
    if not trip_update.HasField('trip'):
        errors.append("Missing trip descriptor")
    
    # Validate trip ID exists in static feed
    if trip_update.trip.trip_id not in static_trips:
        errors.append(f"Unknown trip_id: {trip_update.trip.trip_id}")
    
    # Validate stop references
    for stu in trip_update.stop_time_update:
        if stu.HasField('stop_id'):
            if stu.stop_id not in static_stops:
                errors.append(f"Unknown stop_id: {stu.stop_id}")
        
        # Validate delay field presence
        if stu.HasField('arrival'):
            if not stu.arrival.HasField('delay') and not stu.arrival.HasField('time'):
                errors.append("StopTimeEvent missing both delay and time")
    
    return errors
```



### Performance Optimization

#### Compression

Enable gzip compression to reduce bandwidth by 70-90%:

```python
headers = {
    'Accept-Encoding': 'gzip, deflate'
}
response = requests.get(FEED_URL, headers=headers)
```

#### Caching Strategy

Implement multi-tier caching:

1. **HTTP Cache**: Use `If-Modified-Since` and `ETag` headers
2. **Memory Cache**: Store parsed Protocol Buffer objects
3. **Database Cache**: Persist historical data for analysis

```python
import hashlib
from functools import lru_cache

@lru_cache(maxsize=128)
def get_static_feed(feed_url, etag):
    """Cached static feed retrieval."""
    return download_and_parse_static_feed(feed_url)

# Call with current ETag for automatic cache invalidation
feed = get_static_feed(STATIC_URL, current_etag)
```

#### Incremental Processing

For large feeds, process entities incrementally:

```python
def process_feed_streaming(feed_url):
    """Stream processing for large feeds."""
    response = requests.get(feed_url, stream=True)
    
    # Parse directly from stream
    feed = gtfs_realtime_pb2.FeedMessage()
    feed.ParseFromString(response.content)
    
    # Process entities one at a time
    for entity in feed.entity:
        process_entity(entity)
        yield entity  # Generator pattern
```



### Error Handling

#### Comprehensive Error Handling Pattern

```python
import logging
from requests.exceptions import RequestException, Timeout
from google.protobuf.message import DecodeError

def fetch_realtime_feed(url, timeout=10, max_retries=3):
    """Robust feed fetching with error handling."""
    for attempt in range(max_retries):
        try:
            response = requests.get(
                url,
                headers={'Accept-Encoding': 'gzip'},
                timeout=timeout
            )
            response.raise_for_status()
            
            # Validate content type
            content_type = response.headers.get('Content-Type', '')
            if 'protobuf' not in content_type and 'octet-stream' not in content_type:
                logging.warning(f"Unexpected content type: {content_type}")
            
            # Parse Protocol Buffer
            feed = gtfs_realtime_pb2.FeedMessage()
            feed.ParseFromString(response.content)
            
            # Validate feed structure
            if not feed.HasField('header'):
                raise ValueError("Feed missing header")
            
            if feed.header.gtfs_realtime_version != "2.0":
                logging.warning(f"Unexpected GTFS-RT version: {feed.header.gtfs_realtime_version}")
            
            return feed
        
        except Timeout:
            logging.error(f"Timeout fetching feed (attempt {attempt + 1}/{max_retries})")
        
        except RequestException as e:
            logging.error(f"HTTP error: {e} (attempt {attempt + 1}/{max_retries})")
        
        except DecodeError as e:
            logging.error(f"Protocol Buffer decode error: {e}")
            return None  # Don't retry decode errors
        
        except Exception as e:
            logging.error(f"Unexpected error: {e}")
            return None
        
        # Exponential backoff
        time.sleep(2 ** attempt)
    
    logging.error(f"Failed to fetch feed after {max_retries} attempts")
    return None
```



### Security Best Practices

#### API Key Management

```python
import os
from typing import Optional

def get_api_key() -> Optional[str]:
    """Retrieve API key from environment."""
    api_key = os.environ.get('GTFS_API_KEY')
    if not api_key:
        raise ValueError("GTFS_API_KEY environment variable not set")
    return api_key

headers = {
    'X-API-Key': get_api_key(),
    'User-Agent': 'MyTransitApp/1.0 (contact@example.com)'
}
```

#### Data Sanitization

Always sanitize user-facing text from GTFS feeds:

```python
import html
import re

def sanitize_text(text: str, max_length: int = 500) -> str:
    """Sanitize GTFS text for display."""
    # HTML escape
    text = html.escape(text)
    
    # Remove control characters
    text = re.sub(r'[\x00-\x1f\x7f-\x9f]', '', text)
    
    # Limit length
    if len(text) > max_length:
        text = text[:max_length] + '...'
    
    return text.strip()
```

#### HTTPS Enforcement

Always use HTTPS for feed endpoints:

```python
def validate_url(url: str) -> bool:
    """Ensure URL uses HTTPS."""
    if not url.startswith('https://'):
        raise ValueError(f"Insecure URL: {url}. HTTPS required.")
    return True
```



### Multi-Language Support

#### Consuming Translated Strings

```python
def get_translated_text(translated_string, preferred_language='en'):
    """Extract text in preferred language."""
    if not translated_string or not translated_string.translation:
        return None
    
    # Try preferred language
    for translation in translated_string.translation:
        if translation.language == preferred_language:
            return translation.text
    
    # Fallback to first available
    return translated_string.translation[0].text

# Usage
alert = feed_entity.alert
header = get_translated_text(alert.header_text, 'es')
```



## Advanced Topics

### GTFS Extensions

Transit agencies can extend GTFS using reserved field ranges (1000-1999). 

**Example Custom Extension:**

```protobuf
extend VehiclePosition {
  optional string custom_vehicle_type = 1001;
  optional bool has_wifi = 1002;
  optional int32 temperature_celsius = 1003;
}
```

**Consuming Extensions:**

```python
# Check if extension exists
if vehicle_position.HasExtension(custom_pb2.custom_vehicle_type):
    vehicle_type = vehicle_position.Extensions[custom_pb2.custom_vehicle_type]
```



### Historical Data Analysis

#### Archiving Strategy

```python
import sqlite3
from datetime import datetime

def archive_vehicle_positions(feed, db_path='gtfs_archive.db'):
    """Archive vehicle positions for historical analysis."""
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS vehicle_positions (
            timestamp INTEGER,
            vehicle_id TEXT,
            latitude REAL,
            longitude REAL,
            bearing REAL,
            speed REAL,
            trip_id TEXT,
            route_id TEXT
        )
    ''')
    
    for entity in feed.entity:
        if entity.HasField('vehicle'):
            vp = entity.vehicle
            cursor.execute('''
                INSERT INTO vehicle_positions VALUES (?, ?, ?, ?, ?, ?, ?, ?)
            ''', (
                feed.header.timestamp,
                vp.vehicle.id if vp.HasField('vehicle') else None,
                vp.position.latitude if vp.HasField('position') else None,
                vp.position.longitude if vp.HasField('position') else None,
                vp.position.bearing if vp.HasField('position') and vp.position.HasField('bearing') else None,
                vp.position.speed if vp.HasField('position') and vp.position.HasField('speed') else None,
                vp.trip.trip_id if vp.HasField('trip') else None,
                vp.trip.route_id if vp.HasField('trip') else None
            ))
    
    conn.commit()
    conn.close()
```



### Trip Matching Algorithm

Match vehicles to trips using fuzzy matching:

```python
from datetime import datetime, timedelta

def match_vehicle_to_trip(vehicle_position, static_data, tolerance_seconds=300):
    """Match vehicle position to most likely trip."""
    if vehicle_position.HasField('trip') and vehicle_position.trip.HasField('trip_id'):
        return vehicle_position.trip.trip_id
    
    # Fuzzy matching based on location and time
    current_time = datetime.fromtimestamp(vehicle_position.timestamp)
    position = (vehicle_position.position.latitude, vehicle_position.position.longitude)
    
    candidates = []
    for trip in static_data.get_active_trips(current_time):
        # Calculate expected position
        expected_pos = trip.get_position_at_time(current_time)
        distance = haversine_distance(position, expected_pos)
        
        if distance < 100:  # Within 100 meters
            candidates.append((trip.trip_id, distance))
    
    if candidates:
        # Return closest match
        return min(candidates, key=lambda x: x )[0]
    
    return None
```



### Real-Time Service Adjustment

Dynamically adjust displayed schedules based on realtime updates:

```python
def get_adjusted_arrival_time(stop_id, trip_id, static_schedule, realtime_updates):
    """Calculate real-time adjusted arrival time."""
    # Get static schedule
    scheduled_time = static_schedule.get_arrival_time(trip_id, stop_id)
    
    # Find realtime update
    for entity in realtime_updates.entity:
        if (entity.HasField('trip_update') and 
            entity.trip_update.trip.trip_id == trip_id):
            
            for stu in entity.trip_update.stop_time_update:
                if stu.stop_id == stop_id:
                    if stu.HasField('arrival'):
                        if stu.arrival.HasField('delay'):
                            # Apply delay to scheduled time
                            return scheduled_time + timedelta(seconds=stu.arrival.delay)
                        elif stu.arrival.HasField('time'):
                            # Use absolute time
                            return datetime.fromtimestamp(stu.arrival.time)
    
    # No realtime data, return scheduled
    return scheduled_time
```



## Production Deployment

### Monitoring and Observability

#### Key Metrics

1. **Feed Availability**: Percentage of successful feed fetches
2. **Data Freshness**: Age of feed timestamp
3. **Update Frequency**: Actual vs. expected update interval
4. **Entity Coverage**: Percentage of trips/vehicles with realtime data
5. **Validation Errors**: Count of data quality issues

**Prometheus Metrics Example:**

```python
from prometheus_client import Counter, Gauge, Histogram

feed_requests = Counter('gtfs_feed_requests_total', 'Total feed requests', ['feed_type', 'status'])
feed_freshness = Gauge('gtfs_feed_age_seconds', 'Age of feed data', ['feed_type'])
feed_parse_duration = Histogram('gtfs_feed_parse_seconds', 'Feed parsing duration', ['feed_type'])
entity_count = Gauge('gtfs_entities_total', 'Number of entities in feed', ['feed_type', 'entity_type'])

def fetch_and_monitor(feed_url, feed_type):
    """Fetch feed with monitoring."""
    start_time = time.time()
    
    try:
        response = requests.get(feed_url)
        feed_requests.labels(feed_type=feed_type, status=response.status_code).inc()
        response.raise_for_status()
        
        feed = gtfs_realtime_pb2.FeedMessage()
        feed.ParseFromString(response.content)
        
        # Record metrics
        age = time.time() - feed.header.timestamp
        feed_freshness.labels(feed_type=feed_type).set(age)
        
        for entity in feed.entity:
            if entity.HasField('vehicle'):
                entity_count.labels(feed_type=feed_type, entity_type='vehicle').inc()
            elif entity.HasField('trip_update'):
                entity_count.labels(feed_type=feed_type, entity_type='trip_update').inc()
            elif entity.HasField('alert'):
                entity_count.labels(feed_type=feed_type, entity_type='alert').inc()
        
        return feed
    
    finally:
        duration = time.time() - start_time
        feed_parse_duration.labels(feed_type=feed_type).observe(duration)
```



### Load Balancing and Scaling

For high-traffic applications consuming multiple agencies:

```python
import asyncio
import aiohttp

async def fetch_feed_async(session, url):
    """Async feed fetching."""
    async with session.get(url) as response:
        content = await response.read()
        feed = gtfs_realtime_pb2.FeedMessage()
        feed.ParseFromString(content)
        return feed

async def fetch_multiple_feeds(feed_urls):
    """Fetch multiple feeds concurrently."""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_feed_async(session, url) for url in feed_urls]
        feeds = await asyncio.gather(*tasks, return_exceptions=True)
        return feeds

# Usage
agencies = [
    'https://api.agency1.com/gtfs/realtime/vehicle-positions',
    'https://api.agency2.com/gtfs/realtime/vehicle-positions',
    'https://api.agency3.com/gtfs/realtime/vehicle-positions'
]

feeds = asyncio.run(fetch_multiple_feeds(agencies))
```



### Database Schema for GTFS Storage

Optimized PostgreSQL schema for GTFS Static with PostGIS:

```sql
-- Enable PostGIS extension
CREATE EXTENSION IF NOT EXISTS postgis;

-- Agency table
CREATE TABLE agency (
    agency_id TEXT PRIMARY KEY,
    agency_name TEXT NOT NULL,
    agency_url TEXT NOT NULL,
    agency_timezone TEXT NOT NULL,
    agency_lang TEXT,
    agency_phone TEXT,
    agency_fare_url TEXT,
    agency_email TEXT
);

-- Stops table with spatial index
CREATE TABLE stops (
    stop_id TEXT PRIMARY KEY,
    stop_code TEXT,
    stop_name TEXT NOT NULL,
    stop_desc TEXT,
    stop_lat DOUBLE PRECISION NOT NULL,
    stop_lon DOUBLE PRECISION NOT NULL,
    zone_id TEXT,
    stop_url TEXT,
    location_type INTEGER DEFAULT 0,
    parent_station TEXT REFERENCES stops(stop_id),
    stop_timezone TEXT,
    wheelchair_boarding INTEGER,
    geom GEOMETRY(Point, 4326)
);

-- Create spatial index
CREATE INDEX idx_stops_geom ON stops USING GIST(geom);

-- Generate geometry from lat/lon
CREATE OR REPLACE FUNCTION update_stop_geom()
RETURNS TRIGGER AS $$
BEGIN
    NEW.geom = ST_SetSRID(ST_MakePoint(NEW.stop_lon, NEW.stop_lat), 4326);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_stop_geom
BEFORE INSERT OR UPDATE ON stops
FOR EACH ROW EXECUTE FUNCTION update_stop_geom();

-- Routes table
CREATE TABLE routes (
    route_id TEXT PRIMARY KEY,
    agency_id TEXT REFERENCES agency(agency_id),
    route_short_name TEXT,
    route_long_name TEXT,
    route_desc TEXT,
    route_type INTEGER NOT NULL,
    route_url TEXT,
    route_color TEXT,
    route_text_color TEXT,
    route_sort_order INTEGER
);

-- Trips table
CREATE TABLE trips (
    trip_id TEXT PRIMARY KEY,
    route_id TEXT NOT NULL REFERENCES routes(route_id),
    service_id TEXT NOT NULL,
    trip_headsign TEXT,
    trip_short_name TEXT,
    direction_id INTEGER,
    block_id TEXT,
    shape_id TEXT,
    wheelchair_accessible INTEGER,
    bikes_allowed INTEGER
);

CREATE INDEX idx_trips_route_id ON trips(route_id);
CREATE INDEX idx_trips_service_id ON trips(service_id);

-- Stop times table (largest table)
CREATE TABLE stop_times (
    trip_id TEXT NOT NULL REFERENCES trips(trip_id),
    arrival_time INTERVAL,
    departure_time INTERVAL,
    stop_id TEXT NOT NULL REFERENCES stops(stop_id),
    stop_sequence INTEGER NOT NULL,
    stop_headsign TEXT,
    pickup_type INTEGER DEFAULT 0,
    drop_off_type INTEGER DEFAULT 0,
    shape_dist_traveled DOUBLE PRECISION,
    timepoint INTEGER DEFAULT 1,
    PRIMARY KEY (trip_id, stop_sequence)
);

CREATE INDEX idx_stop_times_stop_id ON stop_times(stop_id);
CREATE INDEX idx_stop_times_trip_id ON stop_times(trip_id);

-- Calendar table
CREATE TABLE calendar (
    service_id TEXT PRIMARY KEY,
    monday INTEGER NOT NULL,
    tuesday INTEGER NOT NULL,
    wednesday INTEGER NOT NULL,
    thursday INTEGER NOT NULL,
    friday INTEGER NOT NULL,
    saturday INTEGER NOT NULL,
    sunday INTEGER NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL
);

-- Calendar dates (exceptions)
CREATE TABLE calendar_dates (
    service_id TEXT NOT NULL,
    date DATE NOT NULL,
    exception_type INTEGER NOT NULL,
    PRIMARY KEY (service_id, date)
);

CREATE INDEX idx_calendar_dates_date ON calendar_dates(date);

-- Query: Find nearest stops
CREATE OR REPLACE FUNCTION find_nearest_stops(
    lat DOUBLE PRECISION,
    lon DOUBLE PRECISION,
    max_distance_meters INTEGER DEFAULT 500,
    max_results INTEGER DEFAULT 10
)
RETURNS TABLE(stop_id TEXT, stop_name TEXT, distance_meters DOUBLE PRECISION) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        s.stop_id,
        s.stop_name,
        ST_Distance(
            s.geom::geography,
            ST_SetSRID(ST_MakePoint(lon, lat), 4326)::geography
        ) AS distance_meters
    FROM stops s
    WHERE ST_DWithin(
        s.geom::geography,
        ST_SetSRID(ST_MakePoint(lon, lat), 4326)::geography,
        max_distance_meters
    )
    AND location_type IN (0, 1)
    ORDER BY distance_meters
    LIMIT max_results;
END;
$$ LANGUAGE plpgsql;
```



## Testing and Quality Assurance

### Validation Tools

**Canonical GTFS Validator** 
- Official validation tool from MobilityData
- Validates both GTFS Schedule and Realtime
- Available as web service and command-line tool
- Required before production deployment

**Installation:**
```bash
# Download latest release
wget https://github.com/MobilityData/gtfs-validator/releases/latest/download/gtfs-validator.jar

# Run validation
java -jar gtfs-validator.jar -i gtfs_feed.zip -o validation_results
```



### Unit Testing GTFS Consumers

```python
import unittest
from unittest.mock import Mock, patch
from google.transit import gtfs_realtime_pb2

class TestGTFSConsumer(unittest.TestCase):
    
    def setUp(self):
        """Create mock feed for testing."""
        self.feed = gtfs_realtime_pb2.FeedMessage()
        self.feed.header.gtfs_realtime_version = "2.0"
        self.feed.header.timestamp = 1738612800
        
        # Add vehicle position
        entity = self.feed.entity.add()
        entity.id = "vehicle_1"
        entity.vehicle.trip.trip_id = "TRIP_123"
        entity.vehicle.position.latitude = 41.8781
        entity.vehicle.position.longitude = -87.6298
        entity.vehicle.timestamp = 1738612800
    
    def test_parse_vehicle_position(self):
        """Test vehicle position parsing."""
        entity = self.feed.entity[0]
        self.assertTrue(entity.HasField('vehicle'))
        self.assertEqual(entity.vehicle.trip.trip_id, "TRIP_123")
        self.assertAlmostEqual(entity.vehicle.position.latitude, 41.8781, places=4)
    
    def test_missing_optional_fields(self):
        """Test handling of missing optional fields."""
        entity = self.feed.entity[0]
        self.assertFalse(entity.vehicle.HasField('vehicle'))
        self.assertFalse(entity.vehicle.position.HasField('bearing'))
    
    @patch('requests.get')
    def test_feed_fetch_retry(self, mock_get):
        """Test retry logic on failure."""
        mock_get.side_effect = [
            Mock(status_code=500),
            Mock(status_code=200, content=self.feed.SerializeToString())
        ]
        
        result = fetch_realtime_feed('http://example.com/feed', max_retries=2)
        self.assertIsNotNone(result)
        self.assertEqual(mock_get.call_count, 2)

if __name__ == '__main__':
    unittest.main()
```



## Compliance and Legal Considerations

### Data Licensing

GTFS feeds are typically distributed under open licenses:

- **Creative Commons Attribution (CC BY)**: Most common 
- **CC0 (Public Domain)**: No restrictions
- **Custom license**: Check individual agency terms

**Required Attribution Example:**
```
Transit data provided by [Agency Name] under CC BY 4.0 License.
Schedule information current as of [Date].
```

### Privacy Considerations

When handling vehicle positions:
- Do not track individual operator behavior
- Aggregate historical data before analysis
- Comply with GDPR, CCPA for location data
- Implement data retention policies (typically 30-90 days for raw realtime data)

### Accessibility Requirements

Applications consuming GTFS must provide:
- Screen reader compatibility for visually impaired users
- Wheelchair accessibility information from feeds
- Alternative text for visual elements
- WCAG 2.1 AA compliance minimum



## Migration and Versioning

### Upgrading to GTFS Realtime 2.0

Major changes from version 1.0:
- New occupancy fields
- Multi-carriage details
- Enhanced accessibility information
- Additional cause and effect enumerations

**Backward Compatibility:**
All GTFS Realtime 2.0 feeds remain compatible with 1.0 consumers through Protocol Buffer's optional field mechanism.

### Transitioning to Fares v2

Fares v2 replaces legacy `fare_attributes.txt` and `fare_rules.txt` with:
- `fare_products.txt`: Purchasable items
- `fare_leg_rules.txt`: Rules per journey segment
- `fare_transfer_rules.txt`: Transfer policies 

**Migration Strategy:**
1. Maintain both v1 and v2 files during transition period
2. Update consumer applications to prioritize v2
3. Deprecate v1 files after 6-12 months



## Troubleshooting

### Common Issues and Solutions

#### Issue: Realtime IDs Don't Match Static Feed

**Problem:** `trip_id` in realtime not found in trips.txt

**Solutions:**
- Verify static feed version matches realtime reference
- Check for date-specific trip IDs (some agencies append dates)
- Ensure both feeds from same agency/version
- Validate using `feed_info.txt` feed_version field 



#### Issue: Times After Midnight

**Problem:** Confusion with times like `25:30:00`

**Solution:**
GTFS uses service day time, not clock time. `25:30:00` means 1:30 AM the day after service started.

```python
def parse_gtfs_time(time_str, service_date):
    """Parse GTFS time relative to service date."""
    hours, minutes, seconds = map(int, time_str.split(':'))
    
    # Handle times >= 24:00:00
    days_offset = hours // 24
    hours = hours % 24
    
    dt = datetime.combine(service_date, time(hours, minutes, seconds))
    dt += timedelta(days=days_offset)
    return dt
```



#### Issue: Protocol Buffer Parse Errors

**Problem:** `DecodeError` when parsing feed

**Solutions:**
- Verify Content-Type header is correct
- Check for gzip compression (decompress before parsing)
- Validate feed URL returns binary data, not HTML error page
- Confirm gtfs-realtime.proto version matches feed version



#### Issue: Coordinates Appear Incorrect

**Problem:** Stops or vehicles appear in wrong location

**Solutions:**
- Verify coordinate system is WGS84 (EPSG:4326) 
- Check latitude/longitude aren't swapped
- Validate coordinates are decimal degrees, not other formats
- Ensure coordinates within valid ranges (-90 to 90, -180 to 180)


## Performance Benchmarks

### Expected Feed Sizes

| Transit System Size | Static Feed (ZIP) | Realtime Feed (protobuf) | Parse Time |
|---------------------|-------------------|--------------------------|------------|
| Small (1-50 routes) | 500 KB - 2 MB | 10-50 KB | <100ms |
| Medium (51-200 routes) | 2-15 MB | 50-200 KB | 100-500ms |
| Large (201-500 routes) | 15-50 MB | 200-500 KB | 500ms-2s |
| Very Large (500+ routes) | 50-200 MB | 500 KB - 2 MB | 2-10s |

### Optimization Recommendations

**For Large Static Feeds:**
- Use streaming CSV parsers to avoid loading entire file into memory
- Index critical tables (stop_times, trips) in database
- Implement lazy loading for shapes and optional files
- Cache parsed objects with LRU eviction policy

**For High-Frequency Realtime Updates:**
- Implement connection pooling for HTTP requests
- Use HTTP/2 for multiplexed requests to multiple agencies
- Enable persistent connections with Keep-Alive
- Implement incremental feed processing for differential updates

**Memory Optimization:**
```python
import sys
from pympler import asizeof

def analyze_memory_usage(gtfs_static):
    """Analyze memory consumption of GTFS objects."""
    components = {
        'agencies': gtfs_static.agencies,
        'stops': gtfs_static.stops,
        'routes': gtfs_static.routes,
        'trips': gtfs_static.trips,
        'stop_times': gtfs_static.stop_times,
        'calendar': gtfs_static.calendar
    }
    
    total_bytes = 0
    for name, component in components.items():
        size_bytes = asizeof.asizeof(component)
        size_mb = size_bytes / (1024 * 1024)
        print(f"{name}: {size_mb:.2f} MB")
        total_bytes += size_bytes
    
    print(f"\nTotal memory: {total_bytes / (1024 * 1024):.2f} MB")
```



## API Response Examples

### Static Feed ZIP Structure

```
gtfs_static.zip
├── agency.txt
├── stops.txt
├── routes.txt
├── trips.txt
├── stop_times.txt
├── calendar.txt
├── calendar_dates.txt
├── fare_attributes.txt (optional)
├── fare_rules.txt (optional)
├── shapes.txt (optional)
├── frequencies.txt (optional)
├── transfers.txt (optional)
├── pathways.txt (optional)
├── levels.txt (optional)
├── feed_info.txt (optional)
└── attributions.txt (optional)
```

### Trip Updates Response (JSON representation)

```json
{
  "header": {
    "gtfs_realtime_version": "2.0",
    "incrementality": "FULL_DATASET",
    "timestamp": 1738612800
  },
  "entity": [
    {
      "id": "trip_update_1",
      "trip_update": {
        "trip": {
          "trip_id": "RED_20260203_WD_1234",
          "route_id": "RED",
          "direction_id": 1,
          "start_date": "20260203",
          "schedule_relationship": "SCHEDULED"
        },
        "vehicle": {
          "id": "8045",
          "label": "8045"
        },
        "stop_time_update": [
          {
            "stop_sequence": 12,
            "stop_id": "CLARK",
            "arrival": {
              "delay": 120,
              "uncertainty": 30
            },
            "departure": {
              "delay": 135,
              "uncertainty": 30
            },
            "schedule_relationship": "SCHEDULED"
          },
          {
            "stop_sequence": 13,
            "stop_id": "CHICAGO",
            "arrival": {
              "delay": 125,
              "uncertainty": 45
            },
            "departure": {
              "delay": 140,
              "uncertainty": 45
            },
            "schedule_relationship": "SCHEDULED"
          }
        ],
        "timestamp": 1738612785,
        "delay": 120
      }
    },
    {
      "id": "trip_update_2",
      "trip_update": {
        "trip": {
          "trip_id": "BLUE_20260203_WD_5678",
          "route_id": "BLUE",
          "schedule_relationship": "CANCELED"
        },
        "timestamp": 1738612790
      }
    }
  ]
}
```

### Vehicle Positions Response (JSON representation)

```json
{
  "header": {
    "gtfs_realtime_version": "2.0",
    "incrementality": "FULL_DATASET",
    "timestamp": 1738612850
  },
  "entity": [
    {
      "id": "vehicle_8045",
      "vehicle": {
        "trip": {
          "trip_id": "RED_20260203_WD_1234",
          "route_id": "RED",
          "direction_id": 1,
          "start_date": "20260203"
        },
        "vehicle": {
          "id": "8045",
          "label": "8045"
        },
        "position": {
          "latitude": 41.885737,
          "longitude": -87.630886,
          "bearing": 175.5,
          "speed": 13.89
        },
        "current_stop_sequence": 12,
        "stop_id": "CLARK",
        "current_status": "IN_TRANSIT_TO",
        "timestamp": 1738612845,
        "congestion_level": "RUNNING_SMOOTHLY",
        "occupancy_status": "FEW_SEATS_AVAILABLE",
        "occupancy_percentage": 65
      }
    }
  ]
}
```

### Service Alerts Response (JSON representation)

```json
{
  "header": {
    "gtfs_realtime_version": "2.0",
    "incrementality": "FULL_DATASET",
    "timestamp": 1738612900
  },
  "entity": [
    {
      "id": "alert_weather_001",
      "alert": {
        "active_period": [
          {
            "start": 1738612800,
            "end": 1738699200
          }
        ],
        "informed_entity": [
          {
            "route_id": "RED"
          },
          {
            "route_id": "BLUE"
          }
        ],
        "cause": "WEATHER",
        "effect": "SIGNIFICANT_DELAYS",
        "url": {
          "translation": [
            {
              "text": "https://www.transitchicago.com/alerts/red-blue/",
              "language": "en"
            }
          ]
        },
        "header_text": {
          "translation": [
            {
              "text": "Red and Blue Lines: Delays Due to Winter Weather",
              "language": "en"
            },
            {
              "text": "Líneas Roja y Azul: Retrasos por Clima Invernal",
              "language": "es"
            }
          ]
        },
        "description_text": {
          "translation": [
            {
              "text": "Due to heavy snow conditions, Red and Blue Line trains are experiencing delays of 10-15 minutes. Please allow extra travel time. Crews are working to clear tracks.",
              "language": "en"
            },
            {
              "text": "Debido a las fuertes condiciones de nieve, los trenes de las Líneas Roja y Azul experimentan retrasos de 10 a 15 minutos. Por favor, considere tiempo adicional de viaje. Los equipos están trabajando para despejar las vías.",
              "language": "es"
            }
          ]
        },
        "severity_level": "WARNING"
      }
    }
  ]
}
```



## Appendix A: HTTP Status Code Reference

### Success Codes

| Code | Name | Description | Consumer Action |
|------|------|-------------|-----------------|
| 200 OK | Success | Request completed successfully | Parse and process feed data |
| 204 No Content | No Content | Request successful but no data available | Handle gracefully, retry later |
| 206 Partial Content | Partial | Partial data returned (range request) | Process partial data, request remainder |
| 304 Not Modified | Cached | Resource unchanged since last request | Use cached data |

### Client Error Codes

| Code | Name | Description | Consumer Action |
|------|------|-------------|-----------------|
| 400 Bad Request | Invalid Request | Malformed request syntax | Check request parameters and format |
| 401 Unauthorized | Not Authenticated | Missing or invalid credentials | Verify API key or authentication token |
| 403 Forbidden | Access Denied | Valid credentials but insufficient permissions | Contact provider for access |
| 404 Not Found | Not Found | Requested resource doesn't exist | Verify endpoint URL |
| 406 Not Acceptable | Not Acceptable | Requested format not available | Check Accept header |
| 410 Gone | Permanently Removed | Feed permanently discontinued | Update to new feed URL if available |
| 413 Payload Too Large | Request Too Large | Request entity too large | Reduce request size or batch requests |
| 414 URI Too Long | URI Too Long | Request URL too long | Use POST instead of GET with parameters |
| 415 Unsupported Media Type | Unsupported Type | Content-Type not supported | Check Content-Type header |
| 429 Too Many Requests | Rate Limited | Rate limit exceeded | Implement exponential backoff, reduce frequency |
| 451 Unavailable For Legal Reasons | Legal Block | Legally restricted access | Contact provider or use VPN if appropriate |

### Server Error Codes

| Code | Name | Description | Consumer Action |
|------|------|-------------|-----------------|
| 500 Internal Server Error | Server Error | Generic server error | Retry with exponential backoff |
| 501 Not Implemented | Not Implemented | Functionality not supported | Use alternative endpoint or method |
| 502 Bad Gateway | Gateway Error | Invalid response from upstream | Retry after delay |
| 503 Service Unavailable | Unavailable | Temporary service disruption | Retry with exponential backoff, check status page |
| 504 Gateway Timeout | Gateway Timeout | Upstream timeout | Retry with longer timeout |



## Appendix B: MIME Type Reference

### Content-Type Headers

| Format | MIME Type | Usage | Compression |
|--------|-----------|-------|-------------|
| GTFS Static (ZIP) | `application/zip` | Static feed delivery | Native ZIP compression |
| Protocol Buffer | `application/x-protobuf` | Realtime feed delivery (preferred) | Additional gzip recommended |
| Protocol Buffer (alt) | `application/octet-stream` | Realtime feed delivery (alternative) | Additional gzip recommended |
| JSON | `application/json` | API responses, error messages | gzip recommended |
| Plain Text | `text/plain` | Simple error messages | Not typically compressed |
| CSV | `text/csv` | Individual GTFS files (rare) | gzip recommended |

### Accept-Encoding Headers

```http
Accept-Encoding: gzip, deflate, br
```

**Supported Compression Algorithms:**
- `gzip`: Most widely supported, 70-90% size reduction
- `deflate`: Alternative to gzip, similar compression
- `br` (Brotli): Modern algorithm, better compression but less support
- `identity`: No compression (fallback)



## Appendix C: Time Zone Reference

### Common Transit Agency Timezones

| Region | IANA Timezone | UTC Offset (Standard) | UTC Offset (DST) |
|--------|---------------|----------------------|------------------|
| New York, NY | America/New_York | UTC-5 | UTC-4 |
| Chicago, IL | America/Chicago | UTC-6 | UTC-5 |
| Denver, CO | America/Denver | UTC-7 | UTC-6 |
| Los Angeles, CA | America/Los_Angeles | UTC-8 | UTC-7 |
| Phoenix, AZ | America/Phoenix | UTC-7 | UTC-7 (no DST) |
| Seattle, WA | America/Los_Angeles | UTC-8 | UTC-7 |
| Toronto, ON | America/Toronto | UTC-5 | UTC-4 |
| Vancouver, BC | America/Vancouver | UTC-8 | UTC-7 |
| London, UK | Europe/London | UTC+0 | UTC+1 |
| Paris, France | Europe/Paris | UTC+1 | UTC+2 |
| Berlin, Germany | Europe/Berlin | UTC+1 | UTC+2 |
| Tokyo, Japan | Asia/Tokyo | UTC+9 | UTC+9 (no DST) |
| Sydney, Australia | Australia/Sydney | UTC+10 | UTC+11 |

### Critical Timezone Considerations

1. **Always use IANA timezone identifiers** (e.g., `America/Chicago`), never abbreviations (e.g., `CST`)
2. **DST transitions** are handled automatically by timezone-aware datetime libraries
3. **Service day times** (e.g., `25:30:00`) are relative to service start, not clock time
4. **Feed timestamps** in GTFS Realtime use Unix epoch (UTC)
5. **Display times** to users in local timezone, not feed timezone



## Appendix D: Color Specification

### Route Color Guidelines

GTFS uses 6-character hexadecimal color codes without the `#` prefix.

**Accessibility Requirements:**
- Contrast ratio between `route_color` and `route_text_color` must meet WCAG AA standard (4.5:1 minimum)
- Test contrast at https://webaim.org/resources/contrastchecker/

**Common Transit Line Colors:**

| Line Type | Example Color (hex) | Color Name | Text Color |
|-----------|---------------------|------------|------------|
| Red Line | `C60C30` | Cardinal Red | `FFFFFF` (white) |
| Blue Line | `00A1DE` | Process Blue | `FFFFFF` (white) |
| Green Line | `009B3A` | Green | `FFFFFF` (white) |
| Orange Line | `F9461C` | Orange | `FFFFFF` (white) |
| Purple Line | `522398` | Purple | `FFFFFF` (white) |
| Yellow Line | `F9E300` | Yellow | `000000` (black) |
| Brown Line | `62361B` | Brown | `FFFFFF` (white) |
| Pink Line | `E27EA6` | Pink | `000000` (black) |
| Silver Line | `A0A3A7` | Silver | `000000` (black) |

**Validation:**
```python
import re

def validate_color(color_code: str) -> bool:
    """Validate GTFS color format."""
    if not color_code:
        return True  # Optional field
    
    pattern = r'^[0-9A-Fa-f]{6}$'
    return bool(re.match(pattern, color_code))

def calculate_luminance(color_hex: str) -> float:
    """Calculate relative luminance for contrast checking."""
    # Convert hex to RGB
    r = int(color_hex[0:2], 16) / 255
    g = int(color_hex[2:4], 16) / 255
    b = int(color_hex[4:6], 16) / 255
    
    # Apply gamma correction
    r = r / 12.92 if r <= 0.03928 else ((r + 0.055) / 1.055) ** 2.4
    g = g / 12.92 if g <= 0.03928 else ((g + 0.055) / 1.055) ** 2.4
    b = b / 12.92 if b <= 0.03928 else ((b + 0.055) / 1.055) ** 2.4
    
    return 0.2126 * r + 0.7152 * g + 0.0722 * b

def check_contrast_ratio(color1: str, color2: str) -> float:
    """Calculate WCAG contrast ratio between two colors."""
    lum1 = calculate_luminance(color1)
    lum2 = calculate_luminance(color2)
    
    lighter = max(lum1, lum2)
    darker = min(lum1, lum2)
    
    return (lighter + 0.05) / (darker + 0.05)

# Example usage
route_color = "C60C30"  # Red
text_color = "FFFFFF"   # White
contrast = check_contrast_ratio(route_color, text_color)
print(f"Contrast ratio: {contrast:.2f}:1")
print(f"Meets WCAG AA: {contrast >= 4.5}")
```



## Appendix E: Distance and Coordinate Calculations

### Haversine Distance Formula

Calculate distance between two geographic coordinates:

```python
import math

def haversine_distance(lat1: float, lon1: float, lat2: float, lon2: float) -> float:
    """
    Calculate distance between two points on Earth.
    
    Args:
        lat1, lon1: First point coordinates (decimal degrees)
        lat2, lon2: Second point coordinates (decimal degrees)
    
    Returns:
        Distance in meters
    """
    # Earth radius in meters
    R = 6371000
    
    # Convert to radians
    lat1_rad = math.radians(lat1)
    lat2_rad = math.radians(lat2)
    delta_lat = math.radians(lat2 - lat1)
    delta_lon = math.radians(lon2 - lon1)
    
    # Haversine formula
    a = (math.sin(delta_lat / 2) ** 2 +
         math.cos(lat1_rad) * math.cos(lat2_rad) *
         math.sin(delta_lon / 2) ** 2)
    
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    
    distance = R * c
    return distance

# Example
stop1_lat, stop1_lon = 41.885737, -87.630886  # Clark/Lake
stop2_lat, stop2_lon = 41.896075, -87.628176  # Chicago
distance = haversine_distance(stop1_lat, stop1_lon, stop2_lat, stop2_lon)
print(f"Distance: {distance:.2f} meters")
```

### Bearing Calculation

Calculate compass bearing between two points:

```python
def calculate_bearing(lat1: float, lon1: float, lat2: float, lon2: float) -> float:
    """
    Calculate initial bearing from point 1 to point 2.
    
    Returns:
        Bearing in degrees (0-360, where 0 = North, 90 = East)
    """
    lat1_rad = math.radians(lat1)
    lat2_rad = math.radians(lat2)
    delta_lon = math.radians(lon2 - lon1)
    
    x = math.sin(delta_lon) * math.cos(lat2_rad)
    y = (math.cos(lat1_rad) * math.sin(lat2_rad) -
         math.sin(lat1_rad) * math.cos(lat2_rad) * math.cos(delta_lon))
    
    bearing_rad = math.atan2(x, y)
    bearing_deg = math.degrees(bearing_rad)
    
    # Normalize to 0-360
    bearing_deg = (bearing_deg + 360) % 360
    
    return bearing_deg
```

### Coordinate Validation

```python
def validate_coordinates(lat: float, lon: float) -> tuple[bool, str]:
    """
    Validate geographic coordinates.
    
    Returns:
        (is_valid, error_message)
    """
    if not isinstance(lat, (int, float)) or not isinstance(lon, (int, float)):
        return False, "Coordinates must be numeric"
    
    if math.isnan(lat) or math.isnan(lon):
        return False, "Coordinates cannot be NaN"
    
    if math.isinf(lat) or math.isinf(lon):
        return False, "Coordinates cannot be infinite"
    
    if not -90 <= lat <= 90:
        return False, f"Latitude {lat} out of range [-90, 90]"
    
    if not -180 <= lon <= 180:
        return False, f"Longitude {lon} out of range [-180, 180]"
    
    return True, ""
```



## Appendix F: Protocol Buffer Compilation

### Generating Code from .proto File

**Python:**
```bash
# Install Protocol Buffer compiler
pip install protobuf grpcio-tools

# Download gtfs-realtime.proto
wget https://github.com/google/transit/raw/master/gtfs-realtime/proto/gtfs-realtime.proto

# Compile
python -m grpc_tools.protoc -I. --python_out=. gtfs-realtime.proto

# Import in Python
from gtfs_realtime_pb2 import FeedMessage
```

**Java:**
```bash
# Download protoc compiler
wget https://github.com/protocolbuffers/protobuf/releases/download/v21.12/protoc-21.12-linux-x86_64.zip
unzip protoc-21.12-linux-x86_64.zip

# Compile
./bin/protoc --java_out=./src gtfs-realtime.proto

# Use in Java
import com.google.transit.realtime.GtfsRealtime.FeedMessage;
```

**JavaScript/Node.js:**
```bash
# Install protobufjs
npm install protobufjs

# Use in Node.js
const protobuf = require('protobufjs');
protobuf.load('gtfs-realtime.proto', function(err, root) {
    const FeedMessage = root.lookupType('transit_realtime.FeedMessage');
    // Decode feed
    const message = FeedMessage.decode(buffer);
});
```

**C#:**
```bash
# Install Protocol Buffer compiler for C#
dotnet tool install --global protobuf-net.Protogen

# Compile
protogen --csharp_out=. gtfs-realtime.proto
```

**Go:**
```bash
# Install Go Protocol Buffer plugin
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest

# Compile
protoc --go_out=. gtfs-realtime.proto

# Use in Go
import pb "path/to/gtfs_realtime"
```



## Glossary

| Term | Definition |
|------|------------|
| **Agency** | Transit operator providing service represented in feed   |
| **Block** | Group of sequential trips served by same vehicle, enabling seated transfers |
| **Differential Feed** | Feed containing only changed entities since last update   |
| **Direction ID** | Binary indicator (0 or 1) for travel direction on route |
| **Entity** | Individual data object in GTFS Realtime feed (trip update, vehicle position, or alert) |
| **Fare Zone** | Geographic area with common fare pricing |
| **Feed** | Complete dataset conforming to GTFS specification   |
| **Frequency-Based Service** | Service operating at regular intervals rather than fixed schedules   |
| **Headway** | Time interval between consecutive vehicle departures |
| **Location Type** | Classification of stop (platform, station, entrance, etc.)   |
| **Parent Station** | Station containing multiple platforms or boarding areas |
| **Pathway** | Pedestrian connection within station complex   |
| **Protocol Buffer** | Binary serialization format used by GTFS Realtime   |
| **Route** | Group of trips presented to riders as single service   |
| **Service Calendar** | Pattern defining which days service operates |
| **Service ID** | Identifier linking trips to calendar patterns |
| **Shape** | Geographic path representing vehicle travel route   |
| **Stop** | Location where passengers board or alight vehicles   |
| **Stop Sequence** | Ordered position of stop within trip |
| **Stop Time** | Scheduled arrival/departure at specific stop on trip |
| **Timepoint** | Stop with exact scheduled time (vs. estimated) |
| **Transfer** | Connection between routes at stop or between stops |
| **Trip** | Single journey of vehicle along route at specific time   |
| **Trip Descriptor** | Identifier matching trip in static schedule to realtime update |
| **Vehicle Descriptor** | Identifier for specific physical vehicle   |
| **WGS84** | World Geodetic System 1984 - coordinate reference system for GTFS (EPSG:4326)   |



## Additional Resources

### Official Documentation
- **GTFS.org**: Canonical specification - https://gtfs.org 
- **Google Transit**: Developer documentation - https://developers.google.com/transit 
- **MobilityData**: GTFS community resources - https://mobilitydata.org

### Validation Tools
- **Canonical GTFS Validator**: https://github.com/MobilityData/gtfs-validator 
- **GTFS Realtime Validator**: https://github.com/CUTR-at-USF/gtfs-realtime-validator
- **FeedValidator**: Legacy Python validator - https://github.com/google/transitfeed

### Libraries and SDKs

**Python:**
- `gtfs-realtime-bindings`: Official Protocol Buffer bindings
- `partridge`: Fast GTFS loader using pandas
- `gtfs-kit`: Comprehensive GTFS processing toolkit
- `transitfeed`: Google's legacy GTFS library

**JavaScript:**
- `gtfs-realtime-bindings`: Official bindings
- `gtfs-stream`: Streaming GTFS parser
- `node-gtfs`: Complete Node.js GTFS library

**Java:**
- `onebusaway-gtfs-realtime-api`: Complete GTFS RT implementation
- `gtfs-java`: Static feed parsing

**Ruby:**
- `gtfs-realtime-bindings`: Official bindings
- `gtfs-reader`: Static feed parser

### Community Resources
- **GTFS Best Practices**: https://gtfs.org/schedule/best-practices/
- **GTFS Slack Community**: https://share.mobilitydata.org/slack
- **Transit Developers Google Group**: https://groups.google.com/g/transit-developers
- **Stack Overflow**: Tagged questions - `[gtfs]` and `[gtfs-realtime]`

### Transit Agency Feeds
- **Mobility Database**: Comprehensive catalog - https://database.mobilitydata.org
- **TransitFeeds**: Community-maintained archive - https://transitfeeds.com
- **Transit.land**: Open transit data aggregator - https://transit.land

## Copyright and License

This documentation is provided for informational purposes. The GTFS specification itself is maintained by MobilityData and the GTFS community under the Apache 2.0 License. 

**GTFS Specification License:** Apache License 2.0  
**Specification Maintainer:** MobilityData  
**Original Developer:** Google Inc. (in partnership with TriMet)

Individual transit agency feeds may have their own licensing terms. Always consult agency-specific documentation for usage rights and attribution requirements. 

*This comprehensive GTFS API documentation provides enterprise-grade technical guidance for implementing transit applications using the General Transit Feed Specification. For questions, updates, or contributions, consult the official GTFS community resources listed above.*