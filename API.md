# AllayClaims API

A public API for third-party plugins to integrate with AllayClaims — read claim
data, create/delete claims, listen to cancellable claim events, and run tasks
through a scheduler abstraction that works on both Bukkit/Paper and Folia.

## Modules

| Module | Artifact | Purpose |
|--------|----------|---------|
| API | `com.allayclaims:allayclaims-api:1.0.0` | Interfaces, events, scheduler contract. Shade or depend on this. |
| Plugin | `com.allayclaims:allayclaims-plugin:1.0.0` | The runnable Spigot/Folia plugin jar. |

The plugin jar **shades the API in** at the same package
(`com.allayclaims.api.*`), so end users only install one jar and third-party
plugins compile against the small API artifact.

## Maven

```xml
<dependency>
    <groupId>com.allayclaims</groupId>
    <artifactId>allayclaims-api</artifactId>
    <version>1.0.0</version>
    <scope>provided</scope>
</dependency>
```

Add the JitPack repository if consuming from source:

```xml
<repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
</repository>
```

In your `plugin.yml`, add AllayClaims as a `depend` or `softdepend` so it loads
before yours.

## Getting the API

Two equivalent ways:

```java
// 1. Services Manager (recommended — works across plugin boundaries)
AllayClaimsAPI api = getServer().getServicesManager().load(AllayClaimsAPI.class);

// 2. Static accessor (populated on enable, cleared on disable)
AllayClaimsAPI api = AllayClaimsAPI.get();
```

Always null-check — the API is `null` if AllayClaims is not installed or has not
finished enabling.

## Core interfaces

### `AllayClaimsAPI`
- `version()` — plugin version
- `isFolia()` — true when running on Folia
- `getClaimManager()` — `ClaimManager`
- `hasEconomy()` — true when Vault economy is hooked
- `getScheduler()` — `AllayScheduler`

### `ClaimManager`
- `mode()` — `ClaimMode.SHOVEL` or `ClaimMode.CHUNK`
- `chunkCost()` — claim-block cost of one chunk claim
- `isWorldClaimable(String)`
- `getClaimAt(String world, int x, int z)` — `Claim` or `null`
- `getClaimsOf(UUID owner)` — `List<Claim>`
- `allClaims()` — `Collection<Claim>`
- `usedBlocks(UUID)` / `availableBlocks(UUID)`
- `createClaim(owner, world, x1, z1, x2, z2)` — `CreateResult`
- `createChunkClaim(owner, world, chunkX, chunkZ)` — `CreateResult`
- `deleteClaim(Claim)` / `deleteAllClaims(UUID)` — fire `ClaimDeleteEvent`
- `accrueBlocks(int)` — grant blocks to every known player

### `Claim`
Read-only coordinates plus live mutation:
`trust(UUID)`, `untrust(UUID)`, `canAccess(UUID)`, `setExplosionsEnabled(boolean)`.

### `PlayerData`
`claimBlocks()`, `addBlocks(int)`, `claimIds()`.

## Events

All events are in `com.allayclaims.api.events`, extend Bukkit's `Event`,
implement `Cancellable`, and fire on the main thread (Bukkit) or the owning
region thread (Folia).

| Event | When | Cancellable |
|-------|------|-------------|
| `ClaimCreateEvent` | Before a claim is committed. | Yes — cancels creation. |
| `ClaimDeleteEvent` | Before a claim is deleted. Carries a `DeleteCause` (`UNCLAIM`, `UNALLCLAIM`, `ADMIN`). | Yes — cancels deletion. |
| `ClaimTrustEvent` | Before a player is trusted. | Yes. |
| `ClaimUntrustEvent` | Before a player is untrusted. | Yes. |

### Example: prevent claims in a specific world

```java
public final class MyListener implements Listener {
    @EventHandler
    public void onCreate(ClaimCreateEvent e) {
        if (e.claim().world().equalsIgnoreCase("event_world")) {
            e.setCancelled(true);
        }
    }
}
```

### Example: log every deletion

```java
@EventHandler
public void onDelete(ClaimDeleteEvent e) {
    getLogger().info("Claim " + e.claim().id() + " deleted by " + e.cause());
}
```

## Scheduler (Folia compatibility)

`AllayScheduler` abstracts over the Bukkit and Folia schedulers. Use it instead
of `BukkitScheduler` so your integration code runs on both platforms without
changes.

```java
AllayScheduler s = api.getScheduler();

// Run on the main thread (Bukkit) or the global region thread (Folia)
s.runGlobal(() -> player.sendMessage("hi"));

// Async
s.runAsyncLater(() -> loadDataAsync(), 20L);

// Region-owned task — runs on the thread owning the location
s.runRegion(RegionKey.of(player.getLocation()), () -> {
    player.getWorld().spawnEntity(...);
});
```

| Method | Bukkit | Folia |
|--------|--------|-------|
| `runGlobal` | main thread | global region thread |
| `runGlobalLater` | main thread, delayed | global region, delayed |
| `runGlobalTimer` | main thread, repeating | global region, repeating |
| `runAsync` | async pool | async scheduler |
| `runAsyncLater` | async, delayed | async, delayed |
| `runAsyncTimer` | async, repeating | async, repeating |
| `runRegion` | main thread | region owning the `RegionKey` location |
| `runRegionLater` | main thread, delayed | region, delayed |

### Detecting Folia at runtime

```java
if (api.isFolia()) {
    // Folia-specific logic
}
```

## Example integration plugin

```java
public final class MyPlugin extends JavaPlugin {
    private AllayClaimsAPI api;

    @Override
    public void onEnable() {
        api = AllayClaimsAPI.get();
        if (api == null) {
            getLogger().warning("AllayClaims not found — disabling.");
            getServer().getPluginManager().disablePlugin(this);
            return;
        }
        getServer().getPluginManager().registerEvents(new MyListener(api), this);
    }
}

public final class MyListener implements Listener {
    private final AllayClaimsAPI api;
    public MyListener(AllayClaimsAPI api) { this.api = api; }

    @EventHandler
    public void onPlayerJoin(PlayerJoinEvent e) {
        int available = api.getClaimManager().availableBlocks(e.getPlayer().getUniqueId());
        e.getPlayer().sendMessage("You have " + available + " claim blocks available.");
    }
}
```

## Versioning

The API follows the plugin version. Breaking changes to any interface in
`com.allayclaims.api` bump the major version. Implementations
(`ClaimImpl`, `ClaimManagerImpl`, `PlayerDataImpl`) are internal and not part
of the stable ABI — program to the interfaces.

## License

MIT, Copyright (c) 2026 AllayMC.
