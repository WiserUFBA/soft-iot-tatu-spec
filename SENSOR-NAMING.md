# TATU Sensor Naming Convention

All sensor and actuator names in TATU follow a consistent `camelCase` naming convention. The name used in the `sensor` field of a TATU request must exactly match the function name registered on the device.

This taxonomy serves as a reference vocabulary for TATU implementations, data models, APIs, and documentation.

---

## Environmental Sensors

- `temperatureSensor`
- `humiditySensor`
- `pressureSensor`
- `lightSensor`
- `uvSensor`
- `windSpeedSensor`
- `rainfallSensor`
- `soilMoistureSensor`

## Gas and Air Quality Sensors

- `co2Sensor`
- `coSensor`
- `methaneSensor`
- `smokeSensor`
- `airQualitySensor`
- `ozoneSensor`
- `vocSensor` *(Volatile Organic Compounds)*
- `noxSensor` *(Nitrogen Oxides)*

## Motion and Position Sensors

- `motionSensor`
- `accelerometerSensor`
- `gyroscopeSensor`
- `magnetometerSensor`
- `tiltSensor`
- `pirSensor` *(Passive Infrared)*
- `ultrasonicSensor`
- `proximitySensor`
- `vibrationSensor`

## Audio and Imaging Sensors

- `soundSensor`
- `microphoneSensor`
- `cameraSensor`
- `thermalCameraSensor`

## Electrical Sensors

- `voltageSensor`
- `currentSensor`
- `powerSensor`
- `energyConsumptionSensor`

## Biometric and Health Sensors

- `heartRateSensor`
- `bloodPressureSensor`
- `bloodOxygenSensor`
- `emgSensor` *(Electromyography)*
- `ecgSensor` *(Electrocardiogram)*
- `temperatureBodySensor`

## Location and Navigation Sensors

- `gpsSensor`
- `geoLocationSensor`
- `compassSensor`
- `altitudeSensor`

## Other / Specialized Sensors

- `waterLeakSensor`
- `soilPhSensor`
- `flameSensor`
- `rfidSensor`
- `nfcSensor`
- `touchSensor`
- `weightSensor`
- `loadCellSensor`

---

> This taxonomy is not an official standard but is based on common naming practices across IoT platforms and ontologies such as SOSA/SSN, SAREF, and QUDT. Extend it for your specific domain as needed.
