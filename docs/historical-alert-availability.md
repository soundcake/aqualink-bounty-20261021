# Historical alert availability: local follow-up to PR #1406

This is a narrow follow-up to soundcake's historical observation date implementation:
https://github.com/aqualinkorg/aqualink-app/pull/1406

- Upstream master/base: `99207da6e1590c1c10be5ce298e86e49a4aefe8b`.
- Third-party implementation/head: `684103fd27fc462ddfc68f271f77784808431f03`.
- Original fork: `soundcake/aqualink-bounty-20261021`, branch `bounty/1162-historical-map-sites`.
- The date selector, API cutoff, freshness windows, navigation and historical detail behavior are soundcake's work. They are not created by this follow-up.
- The repository's MIT LICENSE and copyright notice are retained. New SVG assets in this follow-up are original simple vector markup.

## Observed problem and scope

An absent alert reaches the map as `collectionData?.tempWeeklyAlert === undefined`. Before this follow-up, the shared alert helper gives it the same blue color and icon as a measured zero. The heat-stress filter also treats it as zero. The PR already handles empty observations in the detail card; that guard does not apply to map markers.

When a historical date is selected, missing/null alerts now use a gray `no data` interval (with a distinct pin), a corresponding legend item, and a gray table indicator. Missing alerts do not match a selected numeric heat-stress filter. With no heat-stress filter, sites remain visible. Measured levels 0–4 retain their existing appearance. Live rendering/filtering retains its prior fallback.

The API deliberately omits stale observations. This change interprets that absence; it does not invent a new freshness cutoff, infer timestamps from summary values, or alter the API. Tests for expired data use the resulting API summary shape, not a real database query.

## Validation

Six regression test files now cover null/undefined runtime payloads, absent/empty summaries, another metric present but no alert, zero and levels 1–4, both marker shapes, a selected Spotter, wrapped marker copies, historical/live legends, the Redux selector's asOf propagation, historical visibility without a heat-stress filter, and return to live. All 32 targeted tests pass under the official Node 20.20.2 runtime isolated in the workspace. The original 26-test suite produced 10 failures on the unmodified third-party head. Deliberately breaking the no-filter ordering and asOf propagation in a temporary copy produces five expected failures in the strengthened tests.

The real Leaflet lifecycle test exposed that mutable circle color must use `pathOptions`; changing the initial `fillColor` prop alone leaves the circle at its previous color. This follow-up fixes that too, without changing the live color mapping.

Run from `packages/website`:

```sh
../../node_modules/.bin/vitest run src/helpers/bleachingAlertIntervals.test.ts src/helpers/siteUtils.alerts.test.ts src/routes/HomeMap/Map/Markers/SiteMarker.test.tsx src/routes/HomeMap/Map/Markers/SiteMarker.lifecycle.test.tsx src/store/Sites/sitesListSlice.alerts.test.ts src/routes/HomeMap/Map/alertLevelLegend.test.tsx
```

The local evidence/report outside this source patch contains the exact broader test results and limitations. The isolated Leaflet demo uses synthetic values and the production alert helper. Its 12-second MP4 is a montage of before/after browser captures, not a recording of the complete historical-map feature.

This is a supplementary contribution to PR #1406, not a standalone implementation of issue #1162. No authorship of the historical feature, entitlement to the full bounty, or guaranteed payment is claimed.
