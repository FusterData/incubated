# DigitalPerformanceAssetMetric

The observation of one paid-media campaign at a given date. The campaign runs
on a `DigitalPerformanceAsset` (the advertising account). There is **one
entity per campaign** and **one observation per day**.

## Cumulative semantics

Each observation carries the **totals of the campaign up to `dateObserved`**,
not the increment of the day:

- the daily increment is the difference between two consecutive observations;
- the current state of a campaign is its latest observation;
- adding up the observations of a period counts the same spend several times.

This is how advertising platforms consolidate attribution: the figures of a
past day keep changing for several days, and a cumulative snapshot is always
consistent with what the platform reports at that moment.

## Properties

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | URN | yes | `urn:ngsi-ld:DigitalPerformanceAssetMetric:<Brand>:<Platform>:<campaignId>:<temporalResolution>`. It does not include the date. |
| `type` | string | yes | Always `"DigitalPerformanceAssetMetric"`. |
| `dateObserved` | date-time | yes | Day of the observation, start of the day in UTC. |
| `temporalResolution` | enum | yes | Periodicity of the observations. `P1D`. |
| `refDigitalPerformanceAsset` | Relationship → `DigitalPerformanceAsset` | yes | Advertising account the campaign belongs to. |
| `refDataProvider` | Relationship → `DataProvider` | yes | Technical pipeline that supplied the observation. |
| `campaignId` | string | yes | Native identifier of the campaign in the platform. |
| `campaignType` | string | no | Type of the campaign as reported by the platform, without translation. |
| `accountRef` | string | no | Native identifier of the advertising account. Omitted when the platform data does not provide it. |
| `adsSpend` | number | no | Cumulative advertising spend. Monetary, with `unitCode` (ISO 4217, default EUR). |
| `adsImpressions` | integer | no | Cumulative ad impressions. |
| `adsClicks` | integer | no | Cumulative clicks that lead to the advertiser destination. |
| `adsConversions` | integer | no | Cumulative conversions, according to the objective of the campaign. |
| `adsConversionsValue` | number | no | Cumulative monetary value of the conversions. Monetary, with `unitCode`. |

`adsSpend`, `adsImpressions`, `adsClicks`, `adsConversions` and
`adsConversionsValue` have the same name and type as in `DigitalAssetMetric`.

## Cross-platform mapping

The five metrics are comparable across platforms. This is the mapping used by
the first adopter:

| Property | Google Ads | Meta Ads | LinkedIn Ads |
| --- | --- | --- | --- |
| `adsSpend` | cost | spend | cost |
| `adsImpressions` | impressions | impressions | impressions |
| `adsClicks` | clicks | outbound clicks | landing page clicks |
| `adsConversions` | conversions | conversions of the campaign objective | external website conversions |
| `adsConversionsValue` | conversions value | action values | conversion value |
| `campaignType` | advertising channel type | normalized campaign objective | objective type |
| `accountRef` | customer id | ad account id | not available |

## Identifier and publication order

The identifier does not include the date. All the days of a campaign are
writes on the same entity: the context broker keeps the latest observation
and the history is kept by a time-series store (e.g. QuantumLeap) indexed by
`dateObserved`. Consequently, the observations of a campaign must be
published in ascending `dateObserved` order, so that the state left in the
broker is the most recent day.

## Platform-native columns

Publishers may attach the native columns of the advertising platform as
additional properties with their original names (for instance
`bidding_strategy` in Google Ads or `optimization_goal` in Meta Ads), so
that a consumer can reproduce platform-specific analyses from the data
space. Those columns are not comparable across platforms and are **not part
of this data model**; only the properties listed above are.

## Relation to DigitalAssetMetric

`DigitalAssetMetric` observes an asset as a whole and reports the value of a
period. `DigitalPerformanceAssetMetric` observes each campaign of an
advertising account and reports cumulative totals. See the comparison in the
specification of `DigitalPerformanceAsset`.
