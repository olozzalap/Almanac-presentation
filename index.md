# Almanac - centralized Program->Field state machine & Program->Event schemas

### Glossary
- **Event**: a single, discrete, Agronomic action occurring over a period of time (typically single day) on a single Field. May be complete/incomplete for program and considered a draft. Types:
  - Planting
  - Harvest
  - Chemical Application (Fertilizer)
  - Organic Amendment (Manure)
  - Cover Crop Planting
  - Cover Crop Harvest/Termination
  - Irrigation
  - Tillage
  - Grazing
- **Field**: a single, contiguous, piece of land used for crop growing, represented by geo coordinates. Has many Events and can be sub-divided into sub-geos.
- **Program**: a specific collection of Fields incentivized towards a specific regenerative goal. Has it's own Enrollment/Submit/File deadlines, historical data requirements and state machine flow from unenrolled through filed.
- **Program Schema**: a full declarative outline of what Events are available/required for the given program along with their customized attributes and labels.
- **Validation Results**: specific data errors, inconcistencies or inaccuracies (or lack thereof) that prevent us from fully verifying a given Fields set of eligible Events for a given program. If all passing we should be able to Final File.
- **Filing**: also called "Final Filing" this is the last step a grower will take once everything is validated indicating their data is correct and they should be getting some payout for the regen program.
- **SWA**: Sustainability Web App (UI)
- **ADS** Agronomy Data Service
- **QAQC** Quality Assurance Quality Control (Validation) service
- **Almanac** centralized Program->Field state machine & Program->Event schemas

## Problems to solve
- We are unable to add different regenerative programs as our system is only designed around the specific rules for the (previously sole) program: "Carbon"
- Fields are assumed to only ever be enrolled or not enrolled in "Carbon", not multiple
- Enrollment/Submit/File deadlines are hardcoded and across different services
- Susty Web App sole source of truth for Field state (is enrolled, can submit, can file etc.) based off patchwork of data
- Susty Web App sole source of truth for Events schema as hardcoded explicitly for "Carbon"
- QAQC Validation data all flows coupled through ADS

## Architecture before:
![Architecture before](https://github.com/olozzalap/Almanac-presentation/blob/main/Architecture%20-%20PRE.jpg)
- SWA was solely in charge of each Events schema, attrbiutes and required form values
- ADS sends all Events for the given Field to SWA
- Issuance metadata api from ADS would send the yearly "Carbon" program id and deadlines
- If before the deadline, SWA would query Issuance summaries api from ADS to get coarse-grained info (`data_entry_status`, `filing_status`)
- SWA parses the state machine for a given field to indiciate what new Events are allowed, do we have enough historical data to submit, are we ineligible etc.
- If enrolled & open for data entry the Field may be submitted to QAQC for Validation (also factored into SWA state machine)
- Every submit to QAQC requires an explicit step by the grower and can have some delay
- Only SWA is properly aware of a Fields full progress through program and visibility
- No ability to support multiple, configurable, programs at once
- All 5 services inflexible to new requirements or programs

## Architecture after:
![Architecture after](https://github.com/olozzalap/Almanac-presentation/blob/main/Architecture%20-%20POST.jpg)
- Almanac now central source of truth for Events schema, attributes and required/optional values via api. SWA refactored to build dynamic Event forms per the schema.
- Almanac sends list of available programs to SWA
- Almanac sends active program per-each-Field, powering Field UI experience
- Based on what's active SWA queries for certain program schemas
- Almanac sends Field-Program state (`not_enrolled, data_entry, data_entry_complete, verifying`) along with QAQC results if in `data_entry`
- (Not in "Architecture after" diagram) Field-Program state can update immediately on entering new events as Almanac recieves Events data updates via ADS
- Almanac schemas provide and enforce centralized program deadlines
- QAQC now ingests Field+Events from ADS & Program from Almanac allowing Program-specific validation rules
- No QAQC back-and-forth through ADS, only Almanac
- Fields now functional for programs outside of Carbon



