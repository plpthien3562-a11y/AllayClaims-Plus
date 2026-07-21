# AllayClaims

A lightweight gold-shovel claim plugin for Minecraft servers (Bukkit/Spigot/Paper/Folia). Create, resize, trust, and protect your claims with an intuitive two-click system. Includes an optional maintenance tax system and a claim management GUI.

## Features

- **Two-click claim creation** — Right-click two corners with a golden shovel to claim an area.
- **Claim resizing** — Right-click a corner block of your claim, then right-click the new position to resize.
- **Trust system** — Allow other players to build in your claim with `/trust <player>`.
- **Explosion toggle** — Enable or disable explosions per claim with `/claimexplosion`.
- **Claim visualization** — Claim boundaries are shown with glowing blocks for a configurable duration.
- **Economy integration** — Buy and sell claim blocks using Vault economy (`/buyclaimblocks`, `/sellclaimblocks`).
- **Maintenance tax system** — Optional periodic tax charged per claim. Claims are locked if the owner cannot pay, and can be auto-deleted after the overdue payment deadline.
- **Claim management menu** — Open `/cmenu` to buy blocks, sell blocks, and pay tax debt from a GUI.
- **PlaceholderAPI support** — Exposes claim block placeholders.
- **Folia-compatible** — Uses reflection-based scheduler abstraction to support both Bukkit and Folia.

## Requirements

- Java 21+
- Bukkit/Spigot/Paper/Folia 1.20+
- [Vault](https://www.spigotmc.org/resources/vault.34315/) (required for economy features)
- [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.62483/) (optional, for placeholders)

## Installation

1. Download `AllayClaims.jar` from the [Modrinth page](https://modrinth.com/user/AllayMC).
2. Place the jar in your server's `plugins/` folder.
3. Restart the server.
4. Edit `plugins/AllayClaims/config.yml` to configure the plugin.
5. Run `/reload` or restart again.

## Commands

| Command | Description | Permission |
|---|---|---|
| `/trust <player>` | Trust a player in the claim you're standing in | `allayclaims.player` |
| `/untrust <player>` | Revoke trust from a player | `allayclaims.player` |
| `/unclaim` | Delete the claim you're standing in | `allayclaims.player` |
| `/unallclaim` | Delete all of your claims | `allayclaims.player` |
| `/claimexplosion` | Toggle explosions in the current claim | `allayclaims.player` |
| `/buyclaimblocks <amount>` | Buy claim blocks with money | `allayclaims.player` |
| `/sellclaimblocks <amount>` | Sell claim blocks for money | `allayclaims.player` |
| `/cmenu` | Open the claim management menu | `allayclaims.player` |
| `/allayclaims` | Show plugin info and credits | `allayclaims.player` |
| `/bbclaim <sub>` | Admin command (op only) | OP only |

## Configuration

See [`config.yml`](src/main/resources/config.yml) for all options. Key settings:

### Claim Settings

```yaml
max-claim-blocks: 10000       # Maximum claim blocks a player can hold
starting-claim-blocks: 1000   # Blocks given to new players
min-claim-width: 5             # Minimum claim width
min-claim-height: 5            # Minimum claim height
claim-tool: GOLDEN_SHOVEL      # Tool used to create/resize claims
visualization-block: GLOWSTONE # Block shown for claim borders
visualization-duration: 30     # Seconds the visualization stays visible
```

### Economy

```yaml
economy:
  enabled: true
  buy-price: 1.0    # Cost per claim block when buying
  sell-price: 0.5   # Money per claim block when selling
```

### Maintenance Tax

```yaml
tax:
  enabled: false                          # Master toggle (requires Vault)
  per-claim: 1.0                           # Tax per claim per cycle
  overdue-payment-deadline-hours: 72       # Max time a claim can stay locked before deletion
  period-ticks: 1728000                    # Billing cycle (1728000 = 24 hours)
  auto-delete-enabled: true               # Auto-delete locked claims after deadline
```

**How the maintenance tax works:**

1. Once per billing cycle (default: 24 hours), the plugin calculates each player's tax as `(number_of_claims) x per-claim`.
2. The amount is withdrawn from the player's Vault balance.
3. If the player can pay: all claims stay unlocked, debt is cleared.
4. If the player cannot pay: all their claims are **locked** — they cannot build, break, or interact with blocks in the claim.
5. If `auto-delete-enabled` is `true`: claims that have been locked longer than `overdue-payment-deadline-hours` are automatically deleted.
6. If `auto-delete-enabled` is `false`: locked claims stay locked indefinitely until the owner pays the debt via `/cmenu`.

### World Configuration

```yaml
claimable-worlds:       # Worlds where claiming is allowed (empty = all)
  - world
  - world_nether
  - world_the_end
unclaimable-worlds: []  # Worlds where claiming is explicitly blocked
```

## Claim Management Menu (`/cmenu`)

The `/cmenu` command opens a GUI with four options:

- **Buy Claim Blocks** — Purchase additional claim blocks using economy.
- **Sell Claim Blocks** — Sell unused claim blocks for money.
- **Pay Maintenance Tax** — Pay off all accumulated tax debt and unlock claims.
- **Claim Info** — View your claim count, available blocks, and used blocks.

## Admin Commands (`/bbclaim`)

| Subcommand | Description |
|---|---|
| `/bbclaim delete <player>` | Delete all claims owned by a player |
| `/bbclaim give <player> <blocks>` | Give claim blocks to a player |
| `/bbclaim take <player> <blocks>` | Remove claim blocks from a player |
| `/bbclaim ignore` | Toggle admin bypass (operators always bypass) |

## PlaceholderAPI Placeholders

| Placeholder | Description |
|---|---|
| `%allayclaims_blocks%` | Player's available claim blocks |
| `%allayclaims_used%` | Player's used claim blocks |
| `%allayclaims_total%` | Player's total claim blocks |
| `%allayclaims_claims%` | Number of claims owned |

## Building from Source

```bash
git clone https://github.com/plpthien3562-a11y/AllayClaims.git
cd AllayClaims
mvn clean package
```

The compiled jar will be in `target/AllayClaims.jar`.

## License

MIT License. See [LICENSE](LICENSE) for details.

Copyright (c) 2026 AllayMC
