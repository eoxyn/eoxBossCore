<img width="2138" height="736" alt="eoxBossCore banner" src="https://github.com/user-attachments/assets/d2719246-e09f-41f8-8acf-7c5e50acb010" />
<a href="https://bstats.org/plugin/bukkit/eoxBossCore/31069">
  <img src="https://bstats.org/signatures/bukkit/eoxBossCore.svg" width="1200" alt="eoxBossCore bStats statistics — server and player count over time">
</a>

---

A powerful, lightweight, and fully asynchronous custom boss plugin for Folia and Paper 1.20+ servers.  
Designed for high-performance environments, including servers with 1000+ concurrent players.  
Test Server: [play.imibiyum.com](http://play.imibiyum.com)

---

⚠️ **Attention!** Don't forget to increase the `maxHealth` value in `spigot.yml`.  
Spigot does not allow you to exceed the maximum health limit, so if you set a value higher than this in the plugin, you will get an error.

### spigot.yml
```yml
attribute:
  maxHealth:
    max: 1024.0
```

## Features

- Custom boss entities with any EntityType
- Skill system: COMMAND, LIGHTNING, SUMMON_MINION (extensible)
- Skill triggers: ON_SPAWN, ON_DEATH, ON_DAMAGE, ON_TIMER
- Boss bar with range-based show/hide and color control
- AI controller: SLEEP / IDLE / COMBAT state machine
- Minion spawning with configurable max count and safe spawn locations
- Scheduled spawns by day-of-week and time (spawn_schedule.yml)
- Kill leaderboard with SQLite / MySQL / MariaDB support
- PlaceholderAPI expansion with per-player kill and rank placeholders
- Boss persistence across chunk unloads and server restarts
- Full Folia thread-safety (no main-thread blocking)
- `/eoxbc reload` — live config reload, no restart needed

---

## Requirements

| Requirement | Version |
|---|---|
| Paper or Folia | 1.20+ (including 26.x) |
| Java | 17+ |
| PlaceholderAPI | 2.11+ (optional) |

---

## Installation

1. Drop `eoxBossCore.jar` into your `/plugins` folder.
2. Start the server once to generate config files.
3. Add your boss files to `plugins/eoxBossCore/bosses/`.
4. Run `/eoxbc reload` or restart.

---

## Commands & Permissions

| Command | Description | Permission |
|---|---|---|
| `/eoxbc spawn <boss> <world> <x> <y> <z>` | Spawn a boss | `eoxbc.admin` |
| `/eoxbc despawn <instanceId\|all>` | Remove boss(es) | `eoxbc.admin` |
| `/eoxbc list` | List active bosses | `eoxbc.admin` |
| `/eoxbc reload` | Reload all configs | `eoxbc.admin` |
| `/eoxbc debug` | Toggle debug/performance mode | `eoxbc.admin` |

### Aliases

`/ebc` and `/eoxboss` → shorthand for `/eoxbc`

---

## Boss Configuration

Each boss lives in its own file inside `plugins/eoxBossCore/bosses/`.  
You can name the files anything — `soul_guardian.yml`, `ruhani_boss.yml`, etc.  
Every file must have a `bosses:` key at the top level.

```yaml
# plugins/eoxBossCore/bosses/soul_guardian.yml

bosses:
  soul_guardian:
    Type: WITHER_SKELETON
    Name: "&4&lSoul Guardian"
    Health: 500.0
    Damage: 12.0
    Scale: 1.8
    MovementSpeed: 0.28

    Equipment:
      MainHand:   DIAMOND_SWORD
      OffHand:    SHIELD
      Helmet:     NETHERITE_HELMET
      Chestplate: NETHERITE_CHESTPLATE
      Leggings:   NETHERITE_LEGGINGS
      Boots:      NETHERITE_BOOTS

    Options:
      NaturalDespawn: false         # false = persists through chunk unloads
      PreventOtherDrops: true       # suppresses natural mob drops (arrows, flesh, etc.)
      MaxMinions: 6
      BossBar: true
      BossBarColor: RED
      BossBarSegments: NOTCHED_20
      BossBarRange: 80.0
      DespawnTimeout: 3600          # seconds of inactivity before auto-despawn (0 = never)
      CountKillsInLeaderboard: true # false = kills not counted (useful for minion bosses)

    Minions:
      BossId: soul_sentinel
      Count: 3

    Skills:
      death_announce:
        Trigger: ON_DEATH
        Type: COMMAND
        Target: "@self"
        Commands:
          - "say &4[Boss] &cSoul Guardian slain by &e{killer}&c! Top damager: &e{top_damager}"
          - "give {killer} diamond 1"
          - "give {top_damager} diamond 1"

      timer_lightning:
        Trigger: ON_TIMER
        Type: LIGHTNING
        Target: "@LivingInRadius(15)"
        Interval: 100

      timer_summon:
        Trigger: ON_TIMER
        Type: SUMMON_MINION
        Target: "@self"
        Interval: 200
        Parameters:
          boss_id: "soul_sentinel"
          count: "2"

      spawn_announce:
        Trigger: ON_SPAWN
        Type: COMMAND
        Target: "@self"
        Commands:
          - "say &4[Boss] &cA Soul Guardian has appeared at &e{boss_world} {boss_x} {boss_y} {boss_z}&c!"
```

---

## Skill Reference

### Triggers

| Trigger | When it fires |
|---|---|
| `ON_SPAWN` | When the boss spawns |
| `ON_DEATH` | When the boss dies |
| `ON_DAMAGE` | When the boss receives damage |
| `ON_TIMER` | Repeatedly, every `Interval` ticks |

### Skill Types

**COMMAND** — Runs console commands. Supports `&c` and `&#RRGGBB` color codes.

```yaml
Type: COMMAND
Target: "@self"
Commands:
  - "say &c{target} has been struck!"
  - "effect give {target} slowness 5 2"
```

Available placeholders: `{target}` `{target_x}` `{target_y}` `{target_z}`  
`{boss_x}` `{boss_y}` `{boss_z}` `{boss_world}`  
`{killer}` `{killer_uuid}` `{top_damager}` `{top_damager_uuid}`

**LIGHTNING** — Strikes lightning at targets.

```yaml
Type: LIGHTNING
Target: "@LivingInRadius(10)"   # @self | @LivingInRadius(N)
Parameters:
  effect_only: "false"           # true = visual only, no damage
```

**SUMMON_MINION** — Spawns minion bosses around the boss.

```yaml
Type: SUMMON_MINION
Target: "@self"
Parameters:
  boss_id: "soul_sentinel"
  count: "3"
```

> Max active minions is controlled by `MaxMinions` in the boss's `Options` block.

---

## Scheduled Spawns

Edit `plugins/eoxBossCore/spawn_schedule.yml`.  
Changes apply on `/eoxbc reload` — no restart needed.

```yaml
enabled: true   # master switch — false disables ALL schedules instantly

schedules:
  soul_guardian_weekly:
    enabled: true
    boss-id: soul_guardian
    world: world
    x: 1178.5
    y: 79.0
    z: -714.5
    days:
      - MONDAY
      - WEDNESDAY
      - FRIDAY
    times:
      - "18:00"
      - "22:00"

  daily_boss:
    enabled: true
    boss-id: forest_troll
    world: world
    x: 0.5
    y: 64.0
    z: 0.5
    days:
      - EVERY_DAY
    times:
      - "12:00"
```

**Valid days:** `MONDAY` `TUESDAY` `WEDNESDAY` `THURSDAY` `FRIDAY` `SATURDAY` `SUNDAY` `EVERY_DAY`  
**Time format:** `HH:mm` (24-hour, server timezone)

---

## Database & Leaderboard

Edit `plugins/eoxBossCore/database.yml`.  
⚠️ Requires a **full server restart** to apply (not `/eoxbc reload`).

```yaml
database:
  enabled: true
  type: sqlite          # sqlite | mysql | mariadb

  sqlite:
    file: database/database.db

  host: localhost
  port: 3306
  name: eoxbosscore
  username: root
  password: ''
  table-prefix: eoxbc_  # change only before first startup

  pool:
    maximum-pool-size: 5
    minimum-idle: 1

leaderboard:
  refresh-interval: 60  # seconds between leaderboard refreshes
  top-size: 10          # number of top players tracked (1–10)
```

---

## PlaceholderAPI

PlaceholderAPI expansion registers automatically when PlaceholderAPI is installed.  
All placeholders use the `%eoxbc_...%` prefix.

| Placeholder | Returns |
|---|---|
| `%eoxbc_top_1_name%` | Name of #1 killer |
| `%eoxbc_top_1_kills%` | Kill count of #1 |
| `%eoxbc_top_N_name%` | Name of rank N (1–10) |
| `%eoxbc_top_N_kills%` | Kills of rank N |
| `%eoxbc_player_kills%` | Kills of the viewing player |
| `%eoxbc_player_rank%` | Rank of the viewing player |

---

## Messages

All player-facing messages are in `plugins/eoxBossCore/messages.yml`.  
Supports both **MiniMessage** tags and **& color codes** (including `&#RRGGBB` hex).

```yaml
prefix: "&#d41c1ceoxBossCore &8»&r "

command:
  spawned: "{prefix}&#b8f56eBoss '&#eaf56e{boss}&#b8f56e' spawned. &7(instance: &f{instance}&7)"
```

---

## Config

`plugins/eoxBossCore/config.yml` controls AI and boss bar tick rates:

```yaml
settings:
  ai-tick-interval: 5        # ticks between AI updates per boss
  bossbar-tick-interval: 10  # ticks between boss bar range checks
  sleep-range: 80            # blocks — boss sleeps when no player within range
```

---

## License

All rights reserved. Do not redistribute or modify without permission.
