# Balancer 2.0 Changes vs Original

This document describes the functional changes added to `balancer 2.0.js` compared with the original script.

## Tribal Wars Group Clusters

The main addition is support for using existing Tribal Wars village groups as balancing clusters.

Original behavior:

- The script always creates clusters automatically with k-means from village coordinates.
- `number of clusters` controls how many automatic geographic clusters are created.

Balancer 2.0 behavior:

- A new `cluster source` setting was added.
- The user can choose:
  - `automatic clusters` - old k-means behavior.
  - `Tribal Wars groups` - selected in-game village groups become clusters.
- When group mode is active, `number of clusters` is hidden because selected groups define the clusters.

## Group Selection UI

Added a `cluster groups` selector shown when `Tribal Wars groups` mode is selected.

Controls added:

- group search field
- `refresh` button
- `all visible` button
- `clear` button
- selected/visible/total counter
- selected group summary showing chosen group names and IDs

The search supports both:

- group name
- group ID

Selected groups are visually highlighted in the list.

## Saved Settings

Two new localStorage keys were added:

- `game_data.world + "settings_resources_balancer2_cluster_mode"`
- `game_data.world + "settings_resources_balancer2_cluster_groups"`

This persists:

- selected cluster source
- selected group IDs

If a saved group ID is not currently found in the group menu, the UI keeps it visible as:

```text
saved group <id> (not found in menu)
```

## Improved Group Discovery

Group discovery was expanded beyond the original page scan.

The script now tries to find group IDs and names from:

- current page document
- `overview_villages&mode=prod`
- `overview_villages&mode=prod&group=0`
- `groups`
- `groups&mode=overview`
- select options containing group data
- links containing `group=` or `group_id=`
- elements with `data-group-id` or `data-group`

The discovered list is cached in `window.resBalancerGroupCache`.

Clicking `refresh` clears the cache and reads the groups again.

## Group Cluster Construction

When group mode is used:

- each selected Tribal Wars group becomes one cluster
- village coordinates are loaded from the production overview filtered by that group
- paginated group pages are supported
- each village can be assigned to only one cluster
- if a village exists in multiple selected groups, it is assigned to the first selected group and skipped in later groups

This prevents duplicate balancing of the same village.

## Balancing Scope

Original script:

- final statistics and result table are based on all production villages.

Balancer 2.0:

- in group mode, only villages included in selected cluster groups are balanced and shown in result calculations.
- villages outside selected groups are ignored for that balancing run.

This is important when intentionally excluding a group, for example leaving a D cluster out.

## Cluster List Refactor

Balancer 2.0 adds a shared `getClusterLists()` step.

It converts cluster coordinate data into:

- `list_production_cluster`
- `list_production_home_cluster`
- filtered cluster list
- map drawing data

It also filters out clusters that do not contain any production villages.

If no selected group contains usable villages, the script stops with:

```text
no villages found in selected cluster groups
```

## Map Display Changes

Map drawing now works with both:

- automatic cluster labels
- Tribal Wars group cluster labels

For group clusters, cluster stats can show the group name with the coordinate center.

## CounterAPI Tracking Disabled

The original script included CounterAPI calls for usage counting.

In `balancer 2.0.js`, the `hitCountApi()` call is disabled:

```js
// hitCountApi();
```

The original `hitCountApi()` function is kept in the source as a commented reference, but it is not executed.

This prevents the modified fork from sending run counters, device counters, or player-specific counter keys to CounterAPI.

## Result Meaning

The group-cluster mode changes the optimization target.

Automatic clusters optimize mostly by geography:

- usually fewer resources sent
- usually fewer individual launches
- shorter or similar travel distance

Tribal Wars group clusters optimize by selected strategic groups:

- balances each selected group internally
- may send more resources
- may create more launches
- better when each group needs to be self-sufficient

In testing, group clusters balanced selected groups much more evenly, but with a higher transport cost.
