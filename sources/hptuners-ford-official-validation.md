# HP Tuners + Ford Official Validation Notes

Validated: 2026-03-06

Scope: This note validates the repo against primary sources from HP Tuners and Ford only. It is intentionally narrower than the broader synthesized tuning guides in this repo.

## Primary Sources Used

### Local HP Tuners PDFs

- `sources/VCMEditorUserGuide.pdf`
- `sources/VCMScannerUserGuide.pdf`
- `sources/MPVI4UserGuide.pdf`

### Official Web Sources

- HP Tuners Ford support list: <https://www.hptuners.com/vehicles/ford-tuning/>
- HP Tuners MG1 launch announcement: <https://www.hptuners.com/articles/hp-tuners-announces-first-to-market-tuning-support-for-ford-s650-mustang-f-150-bronco-and-more/>
- HP Tuners MPVI4 product info: <https://www.hptuners.com/product/mpvi4/>
- HP Tuners PROLINK+ product info: <https://www.hptuners.com/product/pro-link-plus/>
- Ford Ranger Raptor technical specs PDF: <https://www.fromtheroad.ford.com/content/dam/fordmediasite/us/en/library/2025/specs/2025_Ford_Ranger_Raptor_Technical_Specifications.pdf>
- Ford Performance calibration page: <https://performanceparts.ford.com/part/M-9603-REB30>
- Ford Performance calibration article: <https://www.fromtheroad.ford.com/us/en/articles/2024/performance-upgrade-available-for-ranger-bronco-raptors>
- Ford Ranger Raptor media material confirming anti-lag / CGI block:
  - <https://media.ford.com/content/fordmedia/feu/de/en/news/2022/02/22/next-gen-ford-ranger-raptor-rewrites-the-rulebook-for-ultimate-o.html>
  - <https://media.ford.com/content/fordmedia/img/im/en/media-kits/2022/ranger-raptor.html/1000>

## Confirmed By Official Sources

### HP Tuners Platform

- HP Tuners officially supports the Ford Ranger Raptor 3.0 platform on the Ford support page.
- HP Tuners officially announced first-to-market Ford MG1 support on 2025-07-11 with direct OBD-II tuning support.
- The repo's `MPVI4-only` focus is a repo workflow choice, not an HP Tuners limitation. Official HP Tuners support is broader than the repo scope.
- MPVI4 is a valid primary hardware target for this repo and includes the Pro Feature Set by default.
- PROLINK+ support, standalone data logging, Bluetooth pairing, and MPVI4 registration flow are all documented in official HP Tuners material.

### HP Tuners Workflow

- HP Tuners officially recommends checking DTCs before tuning work.
- HP Tuners officially notes that reading a vehicle clears DTCs, so the user should document unresolved issues before proceeding.
- VCM Editor officially includes the Ford `Torque Inverse Calculator`.
- VCM Editor officially includes the Ford `ETC Effective Area Calculator`.
- VCM Scanner officially supports:
  - standalone data logging
  - Math Parameters
  - Vehicle Profiles
  - Controls & Special Functions
- HP Tuners documents the basic read/write procedure, including ignition on, accessories off, doors closed, and waiting 10-15 seconds before writing.

### Ford Ranger Raptor / Ford Performance

- Ford officially documents the Ranger Raptor with a `3.0L EcoBoost V6`, `405 horsepower`, `430 lb-ft`, and `10-speed automatic`.
- Ford official media confirms the 3.0L engine uses a `compacted graphite-iron` block.
- Ford official media confirms the Baja-mode anti-lag behavior that keeps the turbos spinning for up to `3 seconds`.
- Ford Performance officially documents the Ranger Raptor / Bronco Raptor calibration at `455 horsepower` and `536 lb-ft`.

## Not Confirmed By Official Sources Alone

The following repo content may still be reasonable, but it is not fully validated by HP Tuners and Ford official material alone:

- exact model-specific table numbers as a safe tuning recipe
- recommended starting values for those tables
- model-specific scanner channel packs and math parameters for this vehicle
- KOM behavior targets as a validated official tuning strategy
- HDFX-specific table usage details for Ranger Raptor
- exact limiter encounter order for this specific OS
- detailed wastegate / TIP calibration strategy for this platform
- injector angle recommendations and exact values
- claims about what constitutes safe or optimal WOT thresholds on this vehicle
- most dyno claims, tuner rankings, and stage power outcomes

## Practical Conclusion

Official HP Tuners + Ford sources are enough for:

- confirming hardware and software support
- learning the HP Tuners toolchain
- understanding the basic vehicle specs
- verifying the Ford Performance calibration
- grounding the repo's high-level platform descriptions

Official HP Tuners + Ford sources are not enough for:

- a high-confidence self-tuning playbook
- table-by-table calibration decisions on this vehicle
- safe limit recommendations without additional evidence
- validating aggressive tuning claims from community or tuner sources

## Implication For This Repo

This repo is currently strong as:

- a platform guide
- a tuning theory reference
- a future tuning assistant knowledge base

This repo is not yet sufficient as:

- a fully validated "tune the truck from scratch" manual

To reach that standard, one of these must be added:

1. Carefully selected specialist tuning sources beyond HP Tuners and Ford.
2. Real vehicle assets: stock `.hpt` files, compare files, and VCM Scanner logs.
3. A validated conservative workflow for this exact OS and fuel/mod configuration.
