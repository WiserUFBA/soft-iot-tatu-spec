# TATU Protocol Specification

**Version:** 1.0  
**Status:** Stable

---

## Overview

TATU is a lightweight IoT protocol built on top of MQTT. It allows a gateway, broker client, or application to:

- Read sensor values on demand (GET)
- Stream periodic sensor readings (FLOW)
- Subscribe to value-change notifications (EVENT)
- Write to actuators (POST)
- Stop ongoing streaming operations (STOP)

All communication is JSON over MQTT. The device subscribes to a request topic and publishes responses and errors to separate topics.

---

## Transport

### Topic structure

| Direction | Topic | Purpose |
|---|---|---|
| → device | `{prefix}{device}{reqSuffix}/#` | Requests (device subscribes) |
| device → | `{prefix}{device}{resSuffix}` | Responses |
| device → | `{prefix}{device}{errSuffix}` | Errors |

Default suffixes: `topicReq = /REQ`, `topicRes = /RES`, `topicErr = /ERR`, `topicPrefix = dev/`.

**Example** for device `esp8266-01`:
- Subscribe: `dev/esp8266-01/REQ/#`
- Publish responses: `dev/esp8266-01/RES`
- Publish errors: `dev/esp8266-01/ERR`

### QoS

All messages use QoS 0. The protocol does not depend on delivery guarantees.

---

## Message format

All messages are UTF-8 encoded JSON objects.

### Request

```json
{
  "method": "<METHOD>",
  "sensor": "<sensorName>",
  "time": { "collect": <seconds>, "publish": <seconds> },
  "target": "<METHOD>",
  "value": <any>
}
```

Fields vary by method — see per-method sections below.

### Response

```json
{
  "header": {
    "method": "<METHOD>",
    "device": "<deviceName>",
    "sensor": "<sensorName>",
    "time": { "collect": <seconds>, "publish": <seconds> }
  },
  "payload": {
    "sensors": [
      { "<sensorName>": [<value>, ...] }
    ]
  }
}
```

The `time` field in the header is present only for FLOW and EVENT. GET and POST use a simplified header (see per-method sections).

---

## Methods

### GET — one-shot read

Reads the current value of one sensor (or all sensors on the device) and returns it immediately.

**Request:**
```json
{"method": "GET", "sensor": "temperatureSensor"}
```

- `sensor`: sensor function name. Use the device name to read **all** sensors at once.

**Response:**
```json
{
  "header": {"method": "GET", "device": "esp8266-01", "sensor": "temperatureSensor"},
  "payload": {"sensors": [{"temperatureSensor": [24.2]}]}
}
```

Each sensor value is wrapped in a single-element array for consistency with FLOW and EVENT responses.

---

### FLOW — periodic streaming

Collects sensor values every `collect` seconds and publishes accumulated batches every `publish` seconds. Runs until a STOP command is received.

**Constraints:**
- `collect` must be a positive integer (seconds).
- `publish` must be ≥ `collect`.
- Violating either constraint produces an `INVALID_PARAMS` error and the task is not created.

**Request:**
```json
{"method": "FLOW", "sensor": "temperatureSensor", "time": {"collect": 5, "publish": 30}}
```

- `sensor`: sensor name or device name (all sensors).
- `time.collect`: collection interval in seconds.
- `time.publish`: publish interval in seconds. Defaults to `collect` if omitted.

**Response** (published every `publish` seconds):
```json
{
  "header": {
    "method": "FLOW", "device": "esp8266-01", "sensor": "temperatureSensor",
    "time": {"collect": 5, "publish": 30}
  },
  "payload": {"sensors": [{"temperatureSensor": [24.2, 24.5, 24.5, 24.8, 24.5, 24.5]}]}
}
```

All values collected in the window are published, even if repeated. The array length equals the number of successful collections in the window.

---

### EVENT — change detection

Polls the sensor every `collect` seconds and publishes only when the value changes. The current value is published immediately on creation as a reference baseline for the consumer.

**Constraints:**
- `collect` must be a positive integer (seconds).
- `publish` (if provided) must be ≥ `collect`.
- Violating either constraint produces an `INVALID_PARAMS` error and the task is not created.

#### Immediate mode (`publish` omitted or `0`)

Publishes a single-value response as soon as any change is detected.

**Request:**
```json
{"method": "EVENT", "sensor": "lightSensor", "time": {"collect": 1}}
```

**Response on creation (initial value):**
```json
{
  "header": {"method": "EVENT", "device": "esp8266-01", "sensor": "lightSensor", "time": {"collect": 1, "publish": 0}},
  "payload": {"sensors": [{"lightSensor": [842]}]}
}
```

**Response on each subsequent change** (same header format, updated value).

#### Windowed mode (`publish` > 0)

Buffers only the values that changed within the window and publishes them as a batch every `publish` seconds. If no change occurred in the window, the publish is skipped.

**Request:**
```json
{"method": "EVENT", "sensor": "lightSensor", "time": {"collect": 1, "publish": 30}}
```

**Response every `publish` seconds (only if changes occurred):**
```json
{
  "header": {"method": "EVENT", "device": "esp8266-01", "sensor": "lightSensor", "time": {"collect": 1, "publish": 30}},
  "payload": {"sensors": [{"lightSensor": [842, 901, 876]}]}
}
```

**Comparison with FLOW:**

| | FLOW | EVENT (immediate) | EVENT (windowed) |
|---|---|---|---|
| Publishes repeated values | yes | no | no |
| Publish trigger | timer | value change | timer (if changed) |
| Initial value on creation | no | yes | yes |

---

### POST — actuator write

Writes a value to an actuator function and returns the result.

**Request:**
```json
{"method": "POST", "sensor": "ledActuator", "value": true}
```

- `sensor`: actuator function name.
- `value`: value passed to the actuator function. Type depends on the actuator.

**Response:**
```json
{
  "header": {"method": "POST", "device": "esp8266-01", "sensor": "ledActuator", "value": true},
  "payload": {"value": true}
}
```

`value` in the header and payload is the value returned by the actuator function after applying the write.

---

### STOP — stop an ongoing operation

Stops a running FLOW or EVENT task.

**Request:**
```json
{"method": "STOP", "sensor": "temperatureSensor", "target": "FLOW"}
```

- `sensor`: **required**. Must match the sensor name used when the task was created. Omitting it produces an `INVALID_PARAMS` error.
- `target`: method to stop — `"FLOW"` or `"EVENT"`. Defaults to `"FLOW"` if omitted.

If the target task is not currently running, produces a `STOP_NOT_FOUND` error.

STOP produces no response on success — silence confirms the task was stopped.

---

## Error responses

Errors are published to the `/ERR` topic as JSON:

```json
{"code": "<ERROR_CODE>", "message": "<human-readable detail>"}
```

| Code | Produced by | Cause |
|---|---|---|
| `SENSOR_NOT_FOUND` | GET, FLOW, EVENT, POST | Sensor name not registered on this device |
| `INVALID_PARAMS` | FLOW, EVENT, STOP | Missing or out-of-range `time` fields; STOP without `sensor` |
| `UNKNOWN_METHOD` | any | Unrecognised `method` value |
| `STOP_NOT_FOUND` | STOP | Target task is not currently running |
| `SENSOR_READ_ERROR` | GET, FLOW, EVENT, POST | Exception raised by the sensor/actuator function |

The `message` field is optional and carries implementation-specific detail. Consumers must not depend on its content — use `code` for programmatic handling.

---

## Validation rules summary

| Rule | Methods | Error |
|---|---|---|
| `sensor` field must match a registered sensor | GET, FLOW, EVENT, POST | `SENSOR_NOT_FOUND` |
| `time.collect` must be a positive integer | FLOW, EVENT | `INVALID_PARAMS` |
| `time.publish` must be ≥ `time.collect` | FLOW, EVENT (windowed) | `INVALID_PARAMS` |
| `sensor` field is required | STOP | `INVALID_PARAMS` |
| `method` must be one of GET, FLOW, EVENT, POST, STOP | any | `UNKNOWN_METHOD` |

---

## Concurrency and task identity

Each (method, device, sensor) triple identifies one task. Sending a new FLOW or EVENT request for the same triple replaces the existing task — the old one is stopped and the new one starts immediately.

---

## Buffer limits

Implementations may impose a maximum buffer size for FLOW and EVENT windowed mode. Values collected beyond the limit are dropped (drop-newest policy). The recommended limit is 30 samples per sensor per window.

---

## Timing model

All intervals are in seconds. Implementations must use drift-free deadline advancement: each deadline is advanced from its previous value, not from the current time, so accumulated delays do not shift the schedule. Missed deadlines are skipped in O(1).

---

## Protocol versioning

This document describes **v1.0**. Future versions increment the minor version for backwards-compatible additions and the major version for breaking changes. Breaking changes require coordinated updates across all implementations.
