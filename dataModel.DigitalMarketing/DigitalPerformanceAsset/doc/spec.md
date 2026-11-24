# DigitalPerformanceAsset

The advertising account of a `Brand` on a paid-media platform (Google Ads,
Meta Ads, LinkedIn Ads...). It is the stable entity that the campaign-level
observations (`DigitalPerformanceAssetMetric`) are attached to. It carries no
metrics.

There is one `DigitalPerformanceAsset` per `Brand` and platform: a brand that
advertises on Google Ads and Meta Ads has two.

## Properties

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | URN | yes | `urn:ngsi-ld:DigitalPerformanceAsset:<Brand>-<Platform>`. |
| `type` | string | yes | Always `"DigitalPerformanceAsset"`. |
| `name` | string | yes | Human-readable name, e.g. `"Acme Holdings S.L. Google Ads"`. |
| `alternateName` | string | no | Short or commercial alternative. |
| `description` | string | no | Free-text description. |
| `digitalAssetKind` | enum | no | Same enumeration as `DigitalAsset.digitalAssetKind`. Advertising accounts use `PaidAds`. |
| `refBrand` | Relationship → `Brand` | yes | Brand that owns the advertising account. |
| `refSoftwareApplication` | Relationship → `SoftwareApplication` | yes | Advertising platform where the account lives. |

## Relationships

- **`refBrand`** → exactly one `Brand`.
- **`refSoftwareApplication`** → exactly one `SoftwareApplication`
  (e.g. `urn:ngsi-ld:SoftwareApplication:GoogleAds`).

## Relation to DigitalAsset

`DigitalPerformanceAsset` mirrors `DigitalAsset` and reuses the rest of the
subject (`Brand`, `DataProvider`, `SoftwareApplication`, `Organization`)
unchanged. It is a distinct entity type because its observations have a
different grain and semantics:

| | `DigitalAsset` / `DigitalAssetMetric` | `DigitalPerformanceAsset` / `DigitalPerformanceAssetMetric` |
| --- | --- | --- |
| Grain of the observation | the asset | each campaign of the advertising account |
| Value semantics | value of the period (`P1D`, `P1M`, `P3M`, `P1Y`) | cumulative total of the campaign up to `dateObserved` |
| Typical use | cross-brand benchmarking | campaign investment optimisation and forecasting |

Time-series back ends that persist one table per entity type (QuantumLeap on
CrateDB) keep both families in separate collections, so the two use cases
can coexist in the same data space without mixing grains.
