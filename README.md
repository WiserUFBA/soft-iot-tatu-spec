# soft-iot-tatu-spec

Canonical specification of the **TATU protocol** — a lightweight IoT protocol built on top of MQTT for reading sensors and writing actuators on embedded and single-board computer devices.

This repository is the single source of truth for the protocol. All implementations must conform to it.

## Current version

**v1.0** — see [PROTOCOL.md](PROTOCOL.md) for the full specification and [CHANGELOG.md](CHANGELOG.md) for version history.

## Implementations

| Platform | Repository |
|---|---|
| MicroPython (ESP8266) | [soft-iot-tatu-upython](https://github.com/WiserUFBA/soft-iot-tatu-upython) |
| CPython (Raspberry Pi, Linux SBC) | [soft-iot-tatu-python](https://github.com/WiserUFBA/soft-iot-tatu-python) |

## Contributing

Protocol changes require updating this spec **and** all active implementations simultaneously. Open an issue here to propose a change before modifying any implementation repository.
