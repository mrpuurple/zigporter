# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2025-02-26

### Added

- **Initial release of `zigporter`** — CLI tool for migrating Zigbee devices from ZHA to Zigbee2MQTT in Home Assistant
  - `export` command to export ZHA device data from Home Assistant
  - `list-z2m` command to list devices currently in Zigbee2MQTT
  - `migrate` command with interactive wizard for device-by-device migration
  - `setup` command to create and configure `~/.config/zigporter/.env`
- **Persistent migration state tracking** — JSON-based state with `PENDING → IN_PROGRESS → MIGRATED / FAILED` lifecycle; migrations can be paused and resumed
- **HA WebSocket integration** — ZHA device registry queries via WebSocket (compatible with HA 2025+ which dropped the REST ZHA endpoint)
- **Three-tier Z2M auth fallback** — Bearer token, ingress session cookie, and HA-native `mqtt.publish` via `call_service`
- SSL verification support via `HA_VERIFY_SSL` config option

[Unreleased]: https://github.com/nordstad/zigporter/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/nordstad/zigporter/releases/tag/v0.1.0
