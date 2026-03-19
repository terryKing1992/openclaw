## ADDED Requirements

### Requirement: OpenHarmony supervisor marker detection
The system SHALL detect when the gateway is running under OpenHarmony service supervision.

#### Scenario: Detect via environment variable
- **WHEN** environment variable `OPENCLAW_OHOS_SERVICE_NAME` is set
- **THEN** system SHALL identify supervisor type as `ohos`

#### Scenario: OpenHarmony platform supervisor detection
- **WHEN** running on `openharmony` platform with service marker environment variables
- **THEN** `detectRespawnSupervisor()` SHALL return `ohos`

### Requirement: Supervisor type extension
The `RespawnSupervisor` type SHALL include `ohos` as a valid supervisor type.

#### Scenario: Type checking for ohos supervisor
- **WHEN** TypeScript code references `RespawnSupervisor` type
- **THEN** `ohos` SHALL be a valid value alongside `launchd`, `systemd`, and `schtasks`