# Changelog

All notable changes to the TATU protocol are documented here.

Format: `[vMAJOR.MINOR] — YYYY-MM-DD`

---

## [v1.0] — 2026-09-15

First stable release. Consolidates behaviour validated across the MicroPython (ESP8266) and CPython (Raspberry Pi) implementations.

### Defined

- **Transport**: MQTT topics, QoS 0, JSON encoding.
- **GET**: one-shot sensor read; device name reads all sensors.
- **FLOW**: periodic collection with drift-free deadlines; `publish` ≥ `collect`.
- **EVENT immediate** (`publish=0`): publishes initial value on creation; subsequent publishes on each value change.
- **EVENT windowed** (`publish>0`): buffers only changed values; publishes batch every `publish` seconds; skips publish if buffer empty.
- **POST**: actuator write returning the applied value.
- **STOP**: requires `sensor` field; `INVALID_PARAMS` if absent; `STOP_NOT_FOUND` if task not running.
- **Error codes**: `SENSOR_NOT_FOUND`, `INVALID_PARAMS`, `UNKNOWN_METHOD`, `STOP_NOT_FOUND`, `SENSOR_READ_ERROR`.
- **Concurrency rule**: same (method, device, sensor) triple replaces the existing task.
- **Buffer limit**: 30 samples, drop-newest.
- **Timing model**: drift-free deadline advancement.
