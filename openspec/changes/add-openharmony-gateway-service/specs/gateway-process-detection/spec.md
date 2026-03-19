## ADDED Requirements

### Requirement: OpenHarmony platform detection
The system SHALL detect OpenHarmony platform via `process.platform === 'openharmony'` and route gateway process operations accordingly.

#### Scenario: Process args reading on OpenHarmony
- **WHEN** reading gateway process arguments on OpenHarmony
- **THEN** system SHALL use `/proc/${pid}/cmdline` (same as Linux)

#### Scenario: Port listener discovery on OpenHarmony
- **WHEN** discovering gateway processes listening on a port
- **THEN** system SHALL use Unix socket scanning methods (ss/lsof) same as Linux

### Requirement: Gateway process verification
The system SHALL verify that a PID belongs to a gateway process before signaling it.

#### Scenario: Verify gateway process via command line
- **WHEN** checking if PID 12345 is a gateway process
- **THEN** system SHALL read `/proc/12345/cmdline` and verify it contains gateway-related arguments

#### Scenario: Reject signaling non-gateway process
- **WHEN** attempting to signal a PID that is not a gateway process
- **THEN** system SHALL throw an error and refuse to send the signal