# Flight Fuel and Emissions Dataset: Column Guide

This guide explains the 38 columns in `icao_openap_flight_fuel_emissions.csv` in plain language and describes how each may relate to `total_fuel_burn_kg`.

## Important data-quality warning

The file contains 100,000 rows. A check of its values found a serious concern with the aircraft weight and fuel scales:

- `tow_kg` (takeoff weight) is over 500,000 kg in 60,074 rows and reaches 30,120,914 kg.
- `total_fuel_burn_kg` has a median of 475,568.3 kg and reaches 29,898,407.7 kg.
- Those values are far beyond plausible weights and fuel amounts for the aircraft models named in this file.
- The four phase-fuel columns add up to `total_fuel_burn_kg` in all rows checked. This confirms that the total is internally summed, but does **not** show that the underlying values or units are realistic.

This suggests a serious scale, unit, or data-generation problem. The CSV does not include a data dictionary or provenance that establishes the cause. Validate the weight and fuel values against a trusted source before training a model or treating the correlations as real-world relationships.

## How to read the fuel-impact column

- **Likely direct** — a real operating condition that can change how much fuel the flight uses.
- **Indirect** — may influence fuel through another condition, or may act as a proxy.
- **No direct effect** — an identifier or label, not a physical cause by itself.
- **Target / derived** — fuel or emissions result. Do not use it as an input when predicting total fuel; that would give the model the answer or a close part of it.
- **Depends / check meaning** — likely relevant, but the CSV does not document enough detail to be certain.

These are expected relationships, not effects measured or proven by this dataset.

## Flight identity and aircraft

| Column | Plain-language meaning | Likely effect on `total_fuel_burn_kg` |
|---|---|---|
| `flight_callsign` | The flight's radio/operating identifier, such as `UAL613`. | **No direct effect.** It identifies a flight; don't treat the number or letters as a fuel measurement. |
| `tail_number` | Registration painted on the individual aircraft, such as `N293A`. | **No direct effect.** It identifies one aircraft. |
| `departure_utc` | The departure date and time in Coordinated Universal Time (UTC). | **Indirect.** Time can help identify season, weather, or congestion, but the timestamp itself does not burn fuel. |
| `origin_icao` | Four-letter ICAO code for the departure airport. | **Indirect.** Airport and route can relate to distance, altitude, and operating conditions. The code itself is not a physical fuel driver. |
| `dest_icao` | Four-letter ICAO code for the arrival airport. | **Indirect.** Helps identify the route and destination conditions. |
| `aircraft_model` | The aircraft type, for example `B777-300ER` or `A320neo`. | **Likely direct.** Different aircraft have different size, engines, and fuel efficiency. |
| `engine_variant` | The particular engine model fitted to the aircraft type. | **Likely direct.** Engine design and performance affect fuel use. |
| `aircraft_age_years` | Approximate age of the aircraft in years. | **Indirect / depends.** Age alone does not determine fuel burn. It might relate to aircraft generation or condition, but the file does not explain how it was used. |
| `cost_index` | A setting airlines use to balance flight time against operating cost, including fuel. Higher settings generally favor saving time over saving fuel. | **Likely indirect.** It can influence chosen speed and therefore fuel use. The scale and exact interpretation in this file are not documented. |

## People, cargo, and aircraft weight

| Column | Plain-language meaning | Likely effect on `total_fuel_burn_kg` |
|---|---|---|
| `passenger_count` | Number of passengers carried. | **Indirect.** More passengers usually mean more weight, which can increase fuel use. Passenger count does not show their actual combined weight. |
| `cargo_mass_kg` | Weight of cargo carried, in kilograms. | **Likely direct.** More payload makes the aircraft heavier and can increase fuel use. |
| `oew_kg` | Operating empty weight: the aircraft with its normal operating equipment, but without the trip's passengers, cargo, and usable trip fuel. | **Likely direct.** A heavier aircraft needs more fuel to fly. |
| `zfw_kg` | Zero-fuel weight: aircraft plus passengers and cargo, before usable trip fuel is counted. | **Likely direct.** Represents the loaded aircraft before trip fuel is added. |
| `tow_kg` | Takeoff weight: aircraft weight as it begins takeoff, including fuel. | **Likely direct.** A heavier takeoff weight generally requires more fuel. **This column has implausibly large values in this file; validate it before use.** |
| `lw_kg` | Landing weight: aircraft weight when it lands. | **Indirect / check meaning.** It is a result of the flight and fuel used, rather than a clean pre-flight input. It may reveal consistency problems. |

**Weight relationships to expect:** zero-fuel weight should include the empty aircraft and payload; takeoff weight should include fuel; landing weight should generally be lower than takeoff weight after fuel has been burned. The names alone do not guarantee the file follows these definitions correctly.

## Route, altitude, speed, and weather

| Column | Plain-language meaning | Likely effect on `total_fuel_burn_kg` |
|---|---|---|
| `great_circle_dist_km` | Approximate shortest distance between the two airports, in kilometres. | **Likely direct.** Longer routes usually need more fuel. |
| `actual_flown_dist_km` | Distance the flight actually flew, in kilometres. It can be longer than the shortest airport-to-airport distance. | **Likely direct.** Extra distance usually means extra fuel. |
| `lateral_inefficiency_pct` | How much longer the flown route is than the great-circle distance, expressed as a percentage. For example, 5% means about 5% farther than the shortest route. | **Indirect / overlaps with distance.** A less direct route can use more fuel. This column is closely tied to the two distance columns, so using all three may duplicate information. |
| `initial_cruise_fl` | The first planned cruising altitude, written as a flight level. Flight level 350 means approximately 35,000 feet. | **Likely direct.** Altitude changes engine and aircraft performance. The best altitude also depends on weight and weather. |
| `step_climb_count` | Number of times the aircraft climbed to a higher cruising altitude during the trip. | **Likely indirect.** Climbing and the chosen altitude profile affect fuel; the count alone does not say how high or when. |
| `cruise_mach` | Cruise speed compared with the speed of sound. For example, 0.80 means 80% of the local speed of sound. | **Likely direct.** Flying faster or slower changes engine fuel use and flight time. |
| `tas_kts` | True airspeed in knots: the aircraft's speed through the surrounding air. One knot is one nautical mile per hour. | **Likely direct.** Speed changes fuel flow and time in the air. |
| `enroute_wind_kts` | Wind along the route while the aircraft is en route, in knots. The sign convention (which direction is positive) is not documented in this file. | **Likely direct / depends.** A headwind can increase flight time and fuel; a tailwind can reduce them. Confirm the sign convention first. |
| `metar_temp_c` | Temperature in degrees Celsius from a METAR, a standard airport weather report. The file does not say which airport or report time was used. | **Indirect.** Temperature affects air conditions and aircraft performance. |
| `metar_qnh_hpa` | QNH air-pressure setting from an airport weather report, in hectopascals (hPa). It helps relate pressure to altitude near the airport. | **Indirect / depends.** Pressure can affect aircraft performance, especially near the airport; the precise source and use are not documented here. |

## Time on the ground and in the air

| Column | Plain-language meaning | Likely effect on `total_fuel_burn_kg` |
|---|---|---|
| `taxi_out_mins` | Minutes spent moving on the ground from the airport stand toward the runway before takeoff. | **Likely direct.** Engines use fuel while taxiing; longer taxi time can use more. |
| `taxi_in_mins` | Minutes spent moving on the ground from the runway to the airport stand after landing. | **Likely direct.** Longer taxi time can use more fuel. |
| `holding_time_mins` | Minutes spent waiting in a pattern near an airport instead of proceeding directly to landing. | **Likely direct.** The aircraft continues using fuel while waiting. |
| `airborne_time_hours` | Time in the air from takeoff to landing, in hours. | **Likely direct.** Longer flights usually use more fuel, though fuel use per hour changes across flight stages. |

## Fuel columns

| Column | Plain-language meaning | Likely effect on `total_fuel_burn_kg` |
|---|---|---|
| `taxi_fuel_kg` | Fuel used while taxiing on the ground, in kilograms. | **Target component.** Added into the total; do not use it as an input when predicting that total. |
| `takeoff_climb_fuel_kg` | Fuel used during takeoff and the climb after leaving the runway, in kilograms. | **Target component.** Added into the total; do not use it as an input when predicting that total. |
| `cruise_fuel_kg` | Fuel used during the cruising part of the flight, in kilograms. | **Target component.** Added into the total; do not use it as an input when predicting that total. Check this column particularly carefully: it contains the largest fuel values in the file. |
| `approach_fuel_kg` | Fuel used while descending, approaching the airport, and preparing to land, in kilograms. | **Target component.** Added into the total; do not use it as an input when predicting that total. |
| `total_fuel_burn_kg` | Total fuel used for the flight, in kilograms. In this file, it equals the sum of the four phase-fuel columns in every row checked. | **Prediction target.** This is the value to predict, not an input feature. |

## Emissions columns

| Column | Plain-language meaning | Likely effect on `total_fuel_burn_kg` |
|---|---|---|
| `co2_emissions_kg` | Estimated carbon dioxide released, in kilograms. | **Target-related / likely derived.** It generally rises with fuel use. Do not use it to predict fuel unless you have confirmed it is an independent measurement; it may reveal the target. |
| `nox_emissions_kg` | Estimated nitrogen oxide gases released, in kilograms. | **Target-related / likely derived.** Related to fuel use and engine conditions; likely not an independent pre-flight input. |
| `h2o_emissions_kg` | Estimated water vapour released, in kilograms. | **Target-related / likely derived.** Usually linked to fuel burned; avoid as a predictor of fuel unless its calculation is independently documented. |
| `sox_emissions_kg` | Estimated sulphur oxide gases released, in kilograms. | **Target-related / likely derived.** Usually linked to the fuel burned; avoid as a predictor of fuel unless independently measured. |

## Practical guidance for a fuel-burn model

1. **Fix or verify the weight and fuel scales first.** The very large `tow_kg` and fuel values are a serious warning, not normal small outliers.
2. Use pre-flight or operating inputs such as aircraft/engine, payload, route distance, speed, altitude, wind, and taxi/airborne times.
3. Do not include `total_fuel_burn_kg` itself, the phase-fuel columns, or likely fuel-derived emissions as model inputs.
4. Treat identifiers (`flight_callsign`, `tail_number`) as labels, not numeric quantities. Airport and aircraft names are categories, not numbers.
5. Missing weather values occur in the file: wind is missing in 1,187 rows, temperature in 1,146, and QNH in 1,221. Decide how to handle these before modeling.
6. Correlation shows that two columns move together; it does not prove that one causes the other. Several columns describe overlapping things, such as the three route-distance measures and the four fuel components.

## Sources and limits

- The column names, units indicated by their suffixes, row count, missing values, and numeric ranges above were checked directly in this CSV.
- The [OpenAP project](https://github.com/junzis/openap) describes OpenAP as an aircraft-performance toolkit that can calculate performance, fuel consumption, and emissions. Its [handbook](https://openap.dev/) covers aircraft performance and fuel/emission modeling.
- The CSV itself does not include a source citation, detailed schema, model version, exact measurement method, or all unit/sign conventions. Definitions marked as inferred or dependent should be confirmed with whoever supplied the file.
