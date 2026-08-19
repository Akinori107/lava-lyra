# Use the Node class

The `Node` class is one of the main classes you will be interacting with when using Lyra.

The `Node` class has a couple functions you will be using frequently:

- `Node.get_player()`
- `Node.get_tracks()`
- `Node.get_recommendations()`
- `Node.build_track()`
- `Node.enable()`
- `Node.disable()`
- `Node.disconnect()`
- `Node.load_search()`


There are also properties the `Node` class has to access certain values:

:::{list-table}
:header-rows: 1

* - Property
  - Type
  - Description

* - `Node.bot`
  - `Client`
  - Returns the Discord.py or Py-cord client linked to this node.

* - `Node.identifier`
  - `str`
  - Returns this node's identifier.

* - `Node.enabled`
  - `bool`
  - Returns whether this node is currently enabled.

* - `Node.lyrics_enabled`
  - `bool`
  - Returns whether lyrics support is enabled for this node.

* - `Node.search_enabled`
  - `bool`
  - Returns whether LavaSearch plugin support is enabled for this node.

* - `Node.health_monitor`
  - `NodeHealthMonitor`
  - Returns the node's health monitor.

* - `Node.health_score`
  - `float`
  - Returns the node's current health score, based on latency, uptime, player load, and connection stability. Used by `NodeAlgorithm.by_health`.

* - `Node.route_planner`
  - `RoutePlanner`
  - Returns the node's route planner, used to manage banned IP addresses for the route planner API (Lavalink or NodeLink).

* - `Node.is_connected`
  - `bool`
  - Returns whether this node is connected or not.

* - `Node.latency` `Node.ping`
  - `float`
  - Returns the latency of the node.

* - `Node.player_count`
  - `int`
  - Returns how many players are connected to this node.

* - `Node.players`
  - `Dict[int, Player]`
  - Returns a dict containing the guild ID and the player object.

* - `Node.pool`
  - `NodePool`
  - Returns the pool this node is apart of.

* - `Node.stats`
  - `NodeStats`
  - Returns the nodes stats.

:::

## Getting a player

To get a player from the nodes list of players, we need to use `Node.get_player()`

```py
Node.get_player(...)
```

After you have initialized your function, you need to specify the `guild_id` of the player.

```py

Node.get_player(guild_id=<your guild ID here>)

```

If the node finds a player with the guild ID you provided, it'll return the [](../api/player.md) object associated with the guild ID.


## Getting tracks

To get tracks using Lavalink, we need to use `Node.get_tracks()`

You can also use `Player.get_tracks()` to do the same thing, but this can be used to fetch tracks regardless if a player exists.

```py
await Node.get_tracks(...)
```

After you have initialized your function, we need to fill in the proper parameters:

:::{list-table}
:header-rows: 1

* - Name
  - Type
  - Description

* - `query`
  - `str`
  - The string you want to search up

* - `ctx`
  - `Optional[commands.Context]`
  - Optional value which sets a `Context` object on the tracks you search.

* - `search_type`
  - `SearchType`
  - Enum which sets the provider to search from. Default value is `SearchType.ytsearch`

* - `filters`
  - `Optional[List[Filter]]`
  - Optional value which sets the filters that should apply when the track is played on the tracks you search.

:::

After you set those parameters, your function should look something like this:

```py

await Node.get_tracks(
    query="<your query here>",
    ctx=<optional ctx object here>,
    search_type=<optional search type here>,
    filters=[<optional filters here>]
)

```

:::{note}

Platform support (Spotify, Apple Music, etc.) is resolved by your Lavalink server's plugins.
No credentials are needed on the client side — configure them in your `application.yml` instead.

:::



You should get a list of `Track` in return after running this function for you to then do whatever you want with it.
Ideally, you should be putting all tracks into some sort of a queue. If you would like to learn about how to use
our queue implementation, you can refer to [](queue.md)


## Building a track from an identifier

If you already have a valid Lavalink track identifier and want to turn it back into a `Track`
object without running a new search, use `Node.build_track()`

You can also use `Player.build_track()` to do the same thing, but this can be used regardless
of whether a player exists.

```py
await Node.build_track(...)
```

After you have initialized your function, we need to fill in the proper parameters:

:::{list-table}
:header-rows: 1

* - Name
  - Type
  - Description

* - `identifier`
  - `str`
  - The Lavalink track identifier to build a track from

* - `ctx`
  - `Optional[commands.Context]`
  - Optional value which sets a `Context` object on the track it builds.

:::

After you set those parameters, your function should look something like this:

```py

await Node.build_track(
    identifier="<your track identifier here>",
    ctx=<optional ctx object here>,
)

```

## Enabling and disabling a node

To temporarily take a node out of rotation without removing it from the pool, we use
`Node.disable()`. To bring it back, we use `Node.enable()`

```py
await Node.disable()
```

Disabling a node closes its websocket connection and stops it from being selected by
`NodePool.get_best_node()` or a random `NodePool.get_node()` call.

```py
await Node.enable()
```

Enabling a node reconnects it if it isn't already connected, so it can be selected again.

## Disconnecting a node

To fully disconnect a node and remove it from the pool (destroying any players connected to
it in the process), use `Node.disconnect()`

```py
await Node.disconnect()
```

Unlike `Node.disable()`, this is not reversible — the node is removed from `NodePool` entirely
and must be re-added with `NodePool.create_node()` if you need it again.

## Searching with LavaSearch

If the node has the LavaSearch plugin installed (check `Node.search_enabled`), you can search
for tracks, albums, artists, playlists, and text in a single call using `Node.load_search()`

```py
result = await Node.load_search(
    query="<your query here>",
    types=[<LavaSearchType values here>],
    search_type=<optional SearchType, defaults to Node's configured search type>,
    ctx=<optional ctx object here>,
)
```

This returns a `SearchResult` (or `None` if nothing was found) grouping the matched tracks,
albums, artists, playlists, and text results by type. See [](search.md) for details on
`SearchResult` and `LavaSearchType`.

## Monitoring node health

`Node.health_monitor` returns a `NodeHealthMonitor`, which tracks a node's reliability over time
and backs the `NodeAlgorithm.by_health` selection algorithm. It has a few things you'll use:

:::{list-table}
:header-rows: 1

* - Member
  - Description

* - `NodeHealthMonitor.get_health_score(latency, player_count)`
  - Computes a health score from the given latency and player count. `Node.health_score` calls this for you with the node's current values.

* - `NodeHealthMonitor.is_circuit_open`
  - `bool` property. `True` if the circuit breaker has tripped (the node is being treated as unhealthy and temporarily skipped).

* - `NodeHealthMonitor.check_circuit_breaker()`
  - Re-evaluates whether the circuit breaker should open or close based on recent failures.

* - `NodeHealthMonitor.record_success()`
  - Records a successful request/operation against the node, improving its health score over time.

* - `NodeHealthMonitor.record_failure()`
  - Records a failed request/operation against the node, degrading its health score and contributing toward tripping the circuit breaker.

* - `NodeHealthMonitor.record_reconnection()`
  - Records that the node reconnected after a disconnect, factored into its stability score.

* - `NodeHealthMonitor.should_health_check()`
  - Returns whether it's time to run another health check on the node, based on the configured `health_check_interval`.

* - `NodeHealthMonitor.quality_tracker`
  - The underlying tracker object used to compute connection-quality statistics.

:::

You will rarely need to call these directly — Lyra's internal reconnect and node-selection logic
already uses them. They're most useful if you're building your own custom node-selection
algorithm or want to expose node health in a dashboard/status command.

```py
node = NodePool.get_node(identifier="MAIN")
print(f"Health score: {node.health_score}")
print(f"Circuit open: {node.health_monitor.is_circuit_open}")
```

## Getting recommendations

To get recommadations using Lavalink, we need to use `Node.get_recommendations()`

You can also use `Player.get_recommendations()` to do the same thing, but this can be used to fetch recommendations regardless if a player exists.

```py
await Node.get_recommendations(...)
```

After you have initialized your function, we need to fill in the proper parameters:

:::{list-table}
:header-rows: 1

* - Name
  - Type
  - Description

* - `track`
  - `Track`
  - The track to fetch recommendations for

* - `ctx`
  - `Optional[commands.Context]`
  - Optional value which sets a `Context` object on the recommendations you fetch.

:::

After you set those parameters, your function should look something like this:

```py

await Node.get_recommendations(
    track=<your track object here>,
    ctx=<optional ctx object here>,
)

```

You should get a list of `Track` in return after running this function for you to then do whatever you want with it.
Ideally, you should be putting all tracks into some sort of a queue. If you would like to learn about how to use
our queue implementation, you can refer to [](queue.md)

## Managing the route planner

:::{note}

The route planner API is used for IP rotation to avoid YouTube/source bans, and works on both
Lavalink and NodeLink nodes — NodeLink implements the same `routeplanner/status`,
`routeplanner/free/address`, and `routeplanner/free/all` endpoints at the same paths. It must be
configured on your node first (`application.yml` for Lavalink, `network.routePlanner` in
`config.ts` for NodeLink) before any of these calls will return meaningful data. If it isn't
configured, `get_status()` will raise a `NodeRestException`.

:::

If your node has IP rotation configured, `Node.route_planner` gives
you a `RoutePlanner` to check its status and manage banned/failing addresses.

```py
route_planner = node.route_planner
```

### Getting the route planner status

```py
status = await route_planner.get_status()
```

This returns a `RouteStats` object with the following attributes:

:::{list-table}
:header-rows: 1

* - Attribute
  - Type
  - Description

* - `strategy`
  - `Optional[RouteStrategy]`
  - The route planner strategy in use (e.g. rotating IP, nano IP), or `None` if it couldn't be parsed.

* - `ip_block_type`
  - `Optional[RouteIPType]`
  - The type of IP block configured (`RouteIPType.IPV4` or `RouteIPType.IPV6`), or `None` if it couldn't be parsed.

* - `ip_block_size`
  - `Optional[str]`
  - The size of the configured IP block.

* - `failing_addresses`
  - `List[FailingIPBlock]`
  - Addresses currently marked as failing, along with when they failed.

* - `block_index`
  - `Optional[str]`
  - The current block index.

* - `address_index`
  - `Optional[str]`
  - The current address index.

:::

### Freeing a failing address

To manually mark a specific address as no longer failing, use `RoutePlanner.free_address()`

```py
await route_planner.free_address(ip="<the failing address here>")
```

### Freeing all failing addresses

To clear every address currently marked as failing, use `RoutePlanner.free_all_addresses()`

```py
await route_planner.free_all_addresses()
```
