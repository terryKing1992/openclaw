## ADDED Requirements

### Requirement: Service adapter registration
The system SHALL register an OpenHarmony service adapter in `GATEWAY_SERVICE_REGISTRY` when `process.platform === 'openharmony'`.

#### Scenario: Resolve gateway service on OpenHarmony
- **WHEN** calling `resolveGatewayService()` on OpenHarmony
- **THEN** system SHALL return the OpenHarmony service adapter

### Requirement: Service installation
The system SHALL support installing the gateway service on OpenHarmony via `openclaw daemon install`.

#### Scenario: Install service with configuration
- **WHEN** user runs `openclaw daemon install` with port 18789 and bind mode `lan`
- **THEN** system SHALL create `~/.openclaw/gateway-18789.env` with configuration
- **AND** system SHALL create `~/.openclaw/logs/` directory

#### Scenario: Install multiple instances
- **WHEN** user installs services for ports 18789 and 18790
- **THEN** system SHALL create separate `gateway-18789.env` and `gateway-18790.env` files

### Requirement: Service uninstallation
The system SHALL support uninstalling the gateway service on OpenHarmony via `openclaw daemon uninstall`.

#### Scenario: Uninstall service
- **WHEN** user runs `openclaw daemon uninstall`
- **THEN** system SHALL stop any running gateway for the configured port
- **AND** system SHALL remove the `.env` and `.pid` files

### Requirement: Service start
The system SHALL support starting the gateway service on OpenHarmony via `openclaw daemon start`.

#### Scenario: Start service from installed configuration
- **WHEN** user runs `openclaw daemon start` with installed service configuration
- **THEN** system SHALL spawn `openclaw gateway run` with configured arguments
- **AND** system SHALL write the process PID to `gateway-<port>.pid`
- **AND** system SHALL redirect stdout/stderr to log files

#### Scenario: Start fails if already running
- **WHEN** user runs `openclaw daemon start` but gateway is already running on that port
- **THEN** system SHALL report service is already loaded

### Requirement: Service stop
The system SHALL support stopping the gateway service on OpenHarmony via `openclaw daemon stop`.

#### Scenario: Stop running service
- **WHEN** user runs `openclaw daemon stop` with a running gateway
- **THEN** system SHALL read PID from `gateway-<port>.pid`
- **AND** system SHALL verify the PID is a gateway process
- **AND** system SHALL send SIGTERM to the process
- **AND** system SHALL wait for process to exit
- **AND** system SHALL remove the `.pid` file

#### Scenario: Stop when not running
- **WHEN** user runs `openclaw daemon stop` but no gateway is running
- **THEN** system SHALL attempt to find gateway on the port via process discovery
- **AND** if found, system SHALL signal that process

### Requirement: Service restart
The system SHALL support restarting the gateway service on OpenHarmony via `openclaw daemon restart`.

#### Scenario: In-process restart via SIGUSR1
- **WHEN** user runs `openclaw daemon restart` with a running gateway
- **THEN** system SHALL send SIGUSR1 to trigger in-process restart
- **AND** system SHALL return `{ outcome: "completed" }`

#### Scenario: Restart when not running
- **WHEN** user runs `openclaw daemon restart` but no gateway is running
- **THEN** system SHALL start the gateway from installed configuration

### Requirement: Service status check
The system SHALL support checking gateway service status on OpenHarmony via `openclaw daemon status`.

#### Scenario: Check running service
- **WHEN** user runs `openclaw daemon status` with a running gateway
- **THEN** system SHALL report `loadedText` status ("running")

#### Scenario: Check stopped service
- **WHEN** user runs `openclaw daemon status` with no running gateway
- **THEN** system SHALL report `notLoadedText` status ("stopped")

### Requirement: Multiple instance support
The system SHALL support multiple gateway instances on different ports.

#### Scenario: Operations target specific port
- **WHEN** user runs `openclaw daemon start --port 18790`
- **THEN** system SHALL operate on `gateway-18790.env` and `gateway-18790.pid`

#### Scenario: Status shows all instances
- **WHEN** user runs `openclaw daemon status --all`
- **THEN** system SHALL report status for all installed gateway instances

### Requirement: Configuration persistence
The system SHALL persist service configuration across restarts.

#### Scenario: Read installed configuration
- **WHEN** calling `readCommand()` on OpenHarmony service
- **THEN** system SHALL read and parse `gateway-<port>.env` file
- **AND** return program arguments and environment

#### Scenario: Runtime information
- **WHEN** calling `readRuntime()` on OpenHarmony service
- **THEN** system SHALL return current process status and port information