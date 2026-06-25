# CR_Master — Centralized Robotics Stack

> One command to run your entire home robotics stack.

CR_Master is the master orchestration layer for a home robotics system. It ties together two projects — [CRCS](https://github.com/siddhesh1008/CRCS) for robot control and [CRML](https://github.com/siddhesh1008/CRML) for machine learning — alongside the supporting infrastructure (MQTT, local AI, Home Assistant) into a single Docker Compose stack.

Built in public as a learning project. Not yet stable for general use.

## Architecture

```
                        CR_Master
┌──────────────────────────────────────────────────────┐
│                                                      │
│   Natural language         ML models + training      │
│   command → robot          data pipeline             │
│                                                      │
│   ┌──────────────┐         ┌──────────────┐          │
│   │     CRCS     │         │     CRML     │          │
│   │ orchestrator │         │   port 8100  │          │
│   │  port 8000   │         └──────┬───────┘          │
│   └──────┬───────┘                │                  │
│          │                        │                  │
│          └──────────┬─────────────┘                  │
│                     │                                │
│              ┌──────▼──────┐                         │
│              │  Mosquitto  │  ← message bus          │
│              │  port 1883  │                         │
│              └─────────────┘                         │
│                                                      │
│   ┌──────────────┐   ┌──────────────────────────┐    │
│   │    Ollama    │   │     Home Assistant       │    │
│   │  port 11434  │   │      host network        │    │
│   └──────────────┘   └──────────────────────────┘    │
└──────────────────────────────────────────────────────┘
                          │
              ────────────┴────────────
              Robot fleet (MQTT topics)
```

## Projects

| Project | Role | Port |
|---|---|---|
| [CRCS](https://github.com/siddhesh1008/CRCS) | Parses natural language commands, routes them to robots via MQTT | 8000 |
| [CRML](https://github.com/siddhesh1008/CRML) | Trains and serves ML models (perception, motion planning, task management) for the robot fleet | 8100 |

## Full Service Map

| Service | Image / Source | Port | Notes |
|---|---|---|---|
| mosquitto | `eclipse-mosquitto:2` | 1883 | MQTT broker, shared message bus |
| orchestrator | `./CRCS/orchestrator` | 8000 | CRCS command processor |
| crml | `./CRML` | 8100 | ML inference + data pipeline |
| ollama | `ollama/ollama` | 11434 | Local LLM (`llama3.1:8b`) |
| homeassistant | `ghcr.io/home-assistant/home-assistant` | host | Home automation |
| homebox | `ghcr.io/sysadminsmedia/homebox` | 7745 | Home inventory |
| ros2 | `osrf/ros:humble-desktop` | host | ROS2 Humble, `ROS_DOMAIN_ID=42` |

## Getting Started

**Clone with submodules:**
```bash
git clone --recurse-submodules git@github.com:siddhesh1008/CR_Master.git
cd CR_Master
```

**Start the full stack:**
```bash
docker-compose up -d
```

**Start a specific service:**
```bash
docker-compose up -d crml
```

**View logs:**
```bash
docker-compose logs -f crml
docker-compose logs -f orchestrator
```

**Stop everything:**
```bash
docker-compose down
```

**Rebuild after code changes:**
```bash
docker-compose build crml
docker-compose up -d crml
```

## MQTT Topics

| Direction | Topic | Publisher |
|---|---|---|
| Master → Robot | `robots/{name}/command` | CRCS orchestrator |
| Robot → Master | `robots/{name}/status` | Robot / fake robot |
| Robot → Master | `robots/{name}/sensors/{type}` | Robot sensors |
| CRML → Robot | `crml/{name}/inference` | CRML inference results |

Monitor all traffic:
```bash
docker exec mosquitto mosquitto_sub -h localhost -t "#" -v
```

## Submodule Workflow

Each project (CRCS, CRML) has its own GitHub repo. CR_Master tracks a specific commit of each.

```bash
# Pull latest from both submodules
git submodule update --remote

# After updating submodules, record the new pointers
git add CRCS CRML
git commit -m "Update submodule refs"
git push
```

Make changes inside `CRCS/` or `CRML/` as normal git repos — commit and push from within those directories.

## Status

- [x] Master docker-compose with all services
- [x] CRCS and CRML as git submodules
- [x] Shared MQTT network (cr_network)
- [x] CRML collecting robot data via MQTT
- [x] CRML REST API (health, models, inference, data stats)
- [x] Model registry with JSON manifest
- [ ] Pytest suite for CRML
- [ ] Migration from individual docker-compose files to CR_Master
- [ ] LLM task management module (Ollama integration)
- [ ] First real ML model (perception)
- [ ] Physical robots connected

## License

MIT
