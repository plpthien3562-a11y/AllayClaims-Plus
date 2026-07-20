# AllayClaims

**Lightweight land-claim plugin for Spigot, Paper, and Folia.**

Golden-shovel rectangle claims, whole-chunk claims, a clean public API for
third-party developers, and first-class Folia support — all in one jar.

---

## Features

- **Two claim modes**
  - **Shovel mode** — select two corners with a golden shovel to create a rectangular claim (GriefPrevention style).
  - **Chunk mode** — claim the entire 16×16 chunk you're standing in with `/chunkclaim`.
- **Claim blocks economy** — every player starts with a configurable block budget. Blocks are spent when claiming and refunded when unclaiming. Accrue more over time (`blocks-per-hour`) or buy/sell them with Vault.
- **Trust system** — `/trust` and `/untrust` grant or revoke per-claim access to other players.
- **Explosion toggle** — `/claimexplosion` enables or disables explosions inside a claim.
- **Admin tools** — `/bbclaim delete|give|take|ignore` for server operators.
- **PlaceholderAPI** — `%allayclaims_claim_block%` shows a player's available claim blocks.
- **Vault economy** — optional buy/sell of claim blocks.
- **100% translatable** — every message lives in `messages.yml`.
- **Folia compatible** — runs natively on Folia's region/async/global schedulers with no extra configuration.
- **Public API** — a stable, shaded API jar lets other plugins read claim data, create/delete claims, listen to cancellable events, and schedule tasks safely on both Bukkit and Folia.

---

## Requirements

- Java 21+
- Minecraft 1.20+
- Spigot, Paper, or Folia

**Optional:**
- [Vault](https://www.spigotmc.org/resources/vault.34315/) — economy hook for buying/selling claim blocks.
- [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) — placeholder expansion.

---

## Installation

1. Drop `AllayClaims.jar` into your `plugins/` folder.
2. Restart the server.
3. Edit `config.yml` and `messages.yml` to taste, then `/reload` or restart.

That's it — no database, no external services.

---

## Commands

| Command | Description |
|--------|-------------|
| `/trust <player>` | Trust a player in the claim you're standing in. |
| `/untrust <player>` | Revoke a player's trust. |
| `/unclaim` | Delete the claim you're standing in. |
| `/unallclaim` | Delete all of your claims. |
| `/claimexplosion` | Toggle explosions in the current claim. |
| `/chunkclaim` | Claim the chunk you're standing in (chunk mode only). |
| `/chunkunclaim` | Unclaim the current chunk (chunk mode only). |
| `/buyclaimblocks <amount>` | Buy claim blocks with money (requires Vault). |
| `/sellclaimblocks <amount>` | Sell claim blocks for money (requires Vault). |
| `/bbclaim <sub>` | Admin: `delete`, `give`, `take`, `ignore` (op only). |
| `/allayclaims` | Show plugin info and credits. |

**Permission:** `allayclaims.player` (default: true) grants basic claim access.

---

## Configuration

`config.yml`:

```yaml
# Claim mode: "shovel" or "chunk"
claim-mode: shovel

max-claim-blocks: 10000
starting-claim-blocks: 1000
blocks-per-hour: 100

economy:
  enabled: true
  buy-price: 1.0
  sell-price: 0.5

min-claim-width: 5
min-claim-height: 5

# Cost (in claim blocks) of one chunk claim in chunk mode.
chunk-cost: 256

claim-tool: GOLDEN_SHOVEL
visualization-block: GLOWSTONE
visualization-duration: 30

claimable-worlds:
  - world
  - world_nether
  - world_the_end
unclaimable-worlds: []

explosions-enabled-by-default: false
block-explosions-in-unclaimed: true
allow-pistons-unclaimed: true
allow-redstone-unclaimed: true
```

All player-facing messages are in `messages.yml` and fully translatable.

---

## Folia Support

AllayClaims auto-detects Folia at startup and routes scheduled tasks to the
correct scheduler (global, region, or async). No config change needed — install
the same jar on Spigot, Paper, or Folia.

---

## Developer API

AllayClaims ships a public API (`com.allayclaims.api`) so other plugins can
read and manipulate claims, react to cancellable events, and run tasks
safely on both Bukkit and Folia.

### Maven

```xml
<dependency>
    <groupId>com.allayclaims</groupId>
    <artifactId>allayclaims-api</artifactId>
    <version>1.0.0</version>
    <scope>provided</scope>
</dependency>
```

### Getting the API

```java
AllayClaimsAPI api = getServer().getServicesManager().load(AllayClaimsAPI.class);
// or
AllayClaimsAPI api = AllayClaimsAPI.get();
```

### Events

| Event | Fires | Cancellable |
|-------|-------|-------------|
| `ClaimCreateEvent` | Before a claim is created. | Yes |
| `ClaimDeleteEvent` | Before a claim is deleted. | Yes |
| `ClaimTrustEvent` | Before a player is trusted. | Yes |
| `ClaimUntrustEvent` | Before a player is untrusted. | Yes |

```java
@EventHandler
public void onCreate(ClaimCreateEvent e) {
    if (e.claim().world().equalsIgnoreCase("event_world")) {
        e.setCancelled(true);
    }
}
```

### Scheduler (Folia-safe)

```java
AllayScheduler s = api.getScheduler();
s.runGlobal(() -> player.sendMessage("hi"));
s.runAsyncLater(() -> loadData(), 20L);
s.runRegion(RegionKey.of(player.getLocation()), () -> { /* region thread */ });
```

Full interface reference and examples: see `API.md`.

---

## Building

```bash
mvn package
```

Outputs:
- `plugin/target/AllayClaims-1.0.0.jar` — the runnable plugin (API shaded in).
- `api/target/allayclaims-api-1.0.0.jar` — standalone API for downstream developers.

---

## License

MIT, Copyright (c) 2026 AllayMC.
