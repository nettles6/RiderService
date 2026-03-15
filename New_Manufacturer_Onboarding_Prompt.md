# New Part Manufacturer Onboarding Prompt

Use this prompt as the starting point whenever a new manufacturer's parts listing is being added to the database. Paste it at the beginning of a new conversation along with the relevant files.

---

## PROMPT

I am working on a motorcycle parts lookup database. Please help me integrate a new manufacturer's parts listing into the existing data. The following files are attached:

- **Parts.ods** — the master parts catalogue
- **vehicle_parts.ods** — the vehicle-to-part compatibility mapping table
- **Model_Listing_Enriched.xlsx** — the master vehicle listing
- **[MANUFACTURER_FILE]** — the new manufacturer's parts/application data

---

### Database Structure

**Parts.ods** contains the parts catalogue with the following columns:

| Column | Description |
|--------|-------------|
| `Part_ID` | Unique part identifier. Format: `PA` followed by 4-digit zero-padded number (e.g. `PA0579`). Continue sequentially from the last existing ID. |
| `Part_Type_ID` | Category code. See reference table below. |
| `Part Type` | Human-readable part type. See reference table below. |
| `Part_Name` | The manufacturer's own part number/name (e.g. `KN-141`, `RMC100`). |
| `Brand` | Manufacturer brand name. |
| `Notes` | Part-level notes. See notes conventions below. |

**Part Type Reference:**

| Part_Type_ID | Part Type |
|---|---|
| PC0001 | Engine Oil Filter |
| PC0002 | Air Filter |
| PC0007 | Spark Plugs |

> If a new part type is required that does not exist in this table, flag it for human review rather than creating a new type ID unilaterally.

---

**vehicle_parts.ods** contains vehicle-to-part compatibility mappings with the following columns:

| Column | Description |
|--------|-------------|
| `partId` | References `Part_ID` in Parts.ods |
| `vehicleId` | References `Vehicle_ID` in Model_Listing_Enriched.xlsx |
| `Part_Name` | The manufacturer's part number (denormalised for convenience) |
| `Brand` | The manufacturer brand name (denormalised for convenience) |
| `Notes` | Mapping-specific notes (e.g. fitment caveats for a specific vehicle) |

---

**Model_Listing_Enriched.xlsx** contains 12,098 vehicles across 23 makes:

APRILIA, BENELLI, BMW, BUELL, CF MOTO, DUCATI, HONDA, HUSABERG, HUSQVARNA, HYOSUNG, INDIAN, KAWASAKI, KTM, KYMCO MOTOR CYCLES, MOTO GUZZI, MV AGUSTA, PIAGGIO, ROYAL ENFIELD, SUZUKI, TRIUMPH, VESPA, VICTORY, YAMAHA

Key columns: `Vehicle_ID`, `Year`, `Make`, `Model`, `Common Name`, `Capacity CC`

> Matching against the vehicle listing should use the `Common Name` field as the primary match target.

---

### Step 1 — Parts Catalogue Integration

1. Read the new manufacturer's file and identify all unique part numbers.
2. Cross-reference against the existing Parts.ods to identify parts **not already listed**.
3. For each new part, assign the next sequential `Part_ID` and the correct `Part_Type_ID`.
4. **Do not duplicate** any part already present in the catalogue. Check by `Part_Name` exact match.
5. Apply the suffix conventions and notes rules below.
6. If a part's type cannot be confidently determined, add it with a `Notes` value of `REVIEW: Part type unconfirmed — please verify before publishing.`

---

### Step 2 — Vehicle Compatibility Mapping

1. Cross-reference the manufacturer's application data against **Model_Listing_Enriched.xlsx**.
2. Use `Common Name` as the primary match field (normalised: lowercase, strip spaces and punctuation).
3. Apply year range filtering strictly — only map a part to a vehicle year that falls within the manufacturer's stated range.
4. **Only commit exact name matches** to vehicle_parts. Do not infer or assume compatibility beyond what is explicitly stated in the source data.
5. Where the manufacturer's model name is shorter or more generic than the `Common Name` in the vehicle listing (i.e. a partial/prefix match), **do not auto-add** — instead compile these into a separate review report.
6. Check for existing entries in vehicle_parts before adding — **do not create duplicate** `(partId, vehicleId)` pairs.
7. Manufacturers whose vehicles are not present in the model listing should be skipped silently (not flagged as errors).

---

### Step 3 — Review Report

Produce a separate `.xlsx` review report for any mappings that could not be confidently auto-assigned. Include:

| Column | Description |
|--------|-------------|
| `Part_ID` | Catalogue ID |
| `Part_Name` | Manufacturer part number |
| `Part_Type` | Part type |
| `Manufacturer` | Source manufacturer name |
| `Mfr_Model` | Model name as listed in the source file |
| `Mfr_Year_Range` | Year range from source file |
| `Vehicle_ID` | Matched vehicle ID |
| `Vehicle_Make` | Vehicle make |
| `Vehicle_Model_Code` | Model code from listing |
| `Vehicle_Common_Name` | Common Name from listing |
| `Vehicle_Year` | Specific vehicle year |
| `Review_Reason` | Explanation of why this was withheld |

Include an **Instructions** tab explaining the purpose of the file and the action required.

---

### Notes Field Conventions

**Parts.ods Notes — Part-level notes only.** Use for:
- Manufacturer supersessions or replacements
- Structural/visual variant relationships
- Type corrections or flags

**vehicle_parts.ods Notes — Fitment-specific notes only.** Use for:
- Caveats specific to a part fitting a particular vehicle
- Installation requirements unique to a vehicle

**Established suffix conventions for K&N-style filter part numbers** (apply to any manufacturer using similar suffix patterns):

| Suffix | Meaning | Standard Note Format |
|--------|---------|----------------------|
| `DK` | Drycharger — secondary cover over an air filter | `Variant of [BASE]. Drycharger — secondary cover over an air filter.` |
| `PR` | Precharger — durable polyester wrap to extend service interval in dusty conditions | `Variant of [BASE]. Precharger — durable polyester wrap over base filter to extend service interval in extremely dusty or dirty conditions by stopping smaller particles without significantly restricting airflow.` |
| `XD` | Extreme Duty version of the base filter | `Variant of [BASE]. Extreme Duty version of the base filter.` |
| `PK` | Precharger Wrap — extra protection in dusty conditions | `Variant of [BASE]. Precharger Wrap — extra protection in dusty conditions.` |
| `PL` | PreCharger filter wrap for extra protection in dusty conditions | `Variant of [BASE]. PreCharger filter wrap designed to fit over the filter to provide extra protection in dusty conditions.` |
| `PY` | PreCharger filter wrap for extra protection in dusty conditions | `Variant of [BASE]. PreCharger filter wrap designed to fit over the filter to provide extra protection in dusty conditions.` |
| `R` | Race use — reduced filtration for improved airflow | `Variant of [BASE]. Race use — reduced filtration for improved airflow.` |
| `-U` | Universal or updated packaging/stocking code | `Variant of [BASE]. "-U" designation typically denotes a universal or slightly updated packaging/stocking code.` |
| `-1` | Performance variant of the base filter | `Variant of [BASE]. Performance variant of the base filter.` |

> For any suffix pattern not listed above, flag the affected parts with `REVIEW: Unrecognised suffix — confirm variant relationship before publishing.`

---

### Invalidated Parts

If a `Part_ID` has been retired, its `Notes` field will contain `INVALIDATED —`. **Never reference an invalidated Part_ID** in vehicle_parts or any new mappings.

---

### Quality & Conservatism Rules

- **When in doubt, do not add.** It is always better to flag for review than to create an incorrect mapping.
- **Never infer fitment** beyond what the source data explicitly states.
- **Never overwrite** existing Notes unless specifically instructed.
- **Flag, don't guess** on part type, suffix meaning, or vehicle matching.
- All new part numbers should have their type independently verifiable from the source file before being committed. If the source file contains a type column, trust it. If type must be inferred from part number prefix or naming convention, flag it.

---

### Output Files

Please produce:
1. **Updated Parts.ods** — with all new parts appended
2. **Updated vehicle_parts.ods** — with all confidently matched vehicle mappings appended
3. **[MANUFACTURER]_Mapping_Review.xlsx** — containing all withheld mappings requiring human review

Provide a summary at the end stating:
- How many new parts were added to the catalogue
- How many vehicle mappings were added
- How many items were withheld for review, and why
- Any data quality issues or anomalies observed in the source file
