
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/71e1c133-034c-44c1-a8d7-4ee8b08e1068" />

<a href="https://bstats.org/plugin/bukkit/eoxBossCore/31069">
  <img src="https://bstats.org/signatures/bukkit/eoxBossCore.svg" width="1200">
</a>

---
A powerful, lightweight, and fully asynchronous custom boss plugin for Folia and Paper 1.21.4+ servers. 
Designed for high-performance environments, including servers with 1000+ concurrent players.

Test Server: play.imibiyum.com

---

## Features

- Custom boss entities with any EntityType
- Skill system: COMMAND, LIGHTNING, SUMMON_MINION (extensible)
- Skill triggers: ON_SPAWN, ON_DEATH, ON_HIT, TIMER
- Boss bar with range-based show/hide and color control
- AI controller: SLEEP / IDLE / COMBAT state machine
- Minion spawning with configurable max count
- Scheduled spawns by day-of-week and time (spawn_schedule.yml)
- Kill leaderboard with SQLite / MySQL / MariaDB support
- PlaceholderAPI expansion
- Boss persistence across chunk unloads and server restarts
- Full Folia thread-safety (no main-thread blocking)
- `/eoxbc reload` — live config reload, no restart needed

---

## Requirements

| Requirement | Version |
|---|---|
| Paper or Folia | 1.21.4+ |
| Java | 17+ |
| PlaceholderAPI | 2.11+ (optional) |

---

## Installation

1. Drop `eoxBossCore.jar` into your `/plugins` folder.
2. Start the server once to generate config files.
3. Edit `plugins/eoxBossCore/bosses/` to define your bosses.
4. Run `/eoxbc reload` or restart.

---

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
- `/ebc` → shorthand for `/eoxbc`

---

## Boss Configuration

Bosses are defined in `plugins/eoxBossCore/bosses/<name>.yml`.  
Each file can contain one or more boss templates under the `bosses:` key.

```yaml
bosses:
  soul_guardian:
    display-name: "<red><bold>Soul Guardian"
    entity-type: WITHER_SKELETON
    max-health: 500
    movement-speed: 0.3
    scale: 1.5

    options:
      boss-bar: true
      boss-bar-color: RED
      boss-bar-overlay: PROGRESS
      boss-bar-range: 60
      despawn: false
      prevent-other-drops: true

    equipment:
      main-hand:
        material: IRON_SWORD
        name: "<dark_red>Soul Blade"
        enchantments:
          SHARPNESS: 3

    skills:
      - trigger: ON_SPAWN
        type: COMMAND
        commands:
          - "say &4[Boss] &cThe Soul Guardian awakens!"

      - trigger: ON_DEATH
        type: COMMAND
        commands:
          - "say &2[Boss] &aThe Soul Guardian has been slain by {killer}!"

      - trigger: TIMER
        interval: 200
        type: LIGHTNING
        target: CURRENT_TARGET

      - trigger: TIMER
        interval: 600
        type: SUMMON_MINION
        minion-boss-id: skeleton_minion
        minion-count: 3
        max-minions: 6
```

---

## Skill Reference

### Triggers

| Trigger | When it fires |
|---|---|
| `ON_SPAWN` | When the boss spawns |
| `ON_DEATH` | When the boss dies |
| `ON_HIT` | When the boss hits a player |
| `TIMER` | Repeatedly, every `interval` ticks |

### Skill Types

**COMMAND** Runs console commands. Supports color codes (`&c`, `&#RRGGBB`).

```yaml
type: COMMAND
commands:
  - "say &c{target} has been struck!"
  - "effect give {target} slowness 5 2"
```

Available placeholders: `{target}` `{target_x}` `{target_y}` `{target_z}`  
`{boss_x}` `{boss_y}` `{boss_z}` `{boss_world}`  
`{killer}` `{killer_uuid}` `{top_damager}` `{top_damager_uuid}`

**LIGHTNING** Strikes lightning at targets.

```yaml
type: LIGHTNING
target: CURRENT_TARGET   # CURRENT_TARGET | RADIUS | SELF
radius: 10               # only used when target: RADIUS
```

**SUMMON_MINION** Spawns minion bosses around the boss.

```yaml
type: SUMMON_MINION
minion-boss-id: skeleton_minion
minion-count: 3
max-minions: 6
```

---

## Scheduled Spawns

Edit `plugins/eoxBossCore/spawn_schedule.yml`.  
Changes apply on `/eoxbc reload` no restart needed.

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
**Time format:** `HH:mm` (24-hour)

---

## Database & Leaderboard

Edit `plugins/eoxBossCore/database.yml`.  
Requires a **full server restart** to apply (not `/eoxbc reload`).

```yaml
database:
  enabled: true
  type: sqlite        # sqlite | mysql | mariadb

  sqlite:
    file: database/database.db

  host: localhost
  port: 3306
  name: eoxbosscore
  username: root
  password: ''

leaderboard:
  refresh-interval: 60
  top-size: 10
```

---

## PlaceholderAPI

Register eoxBossCore with PlaceholderAPI by having it installed.  
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
prefix: "&8[&#fc4235eoxBossCore&8] "

command:
  spawned: "{prefix}<green>Boss '<yellow>{boss}</yellow>' spawned."
```

---

## Config

`plugins/eoxBossCore/config.yml` controls AI and boss bar tick rates:

```yaml
settings:
  ai-tick-interval: 5        # ticks between AI updates per boss
  bossbar-tick-interval: 10  # ticks between boss bar updates
  sleep-range: 80            # blocks — boss sleeps when no player is within range
```

---

## License

All rights reserved. Do not redistribute or modify without permission.
