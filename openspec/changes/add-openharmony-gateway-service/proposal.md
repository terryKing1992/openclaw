## Why

OpenClaw currently supports gateway service management on macOS (launchd), Linux (systemd), and Windows (schtasks). With HarmonyOS PC gaining adoption, users need the ability to run OpenClaw gateway as a background service on this platform. HarmonyOS PC runs Node.js and uses `process.platform === 'openharmony'`, but lacks a systemd-equivalent CLI tool for service management, requiring a custom solution.

## What Changes

- Add OpenHarmony platform detection across the codebase
- Implement PID-based service management adapter for OpenHarmony (`ohos-service.ts`)
- Support `openclaw daemon install/start/stop/restart/status` commands on OpenHarmony
- Enable multiple gateway instances with port-based naming
- Add supervisor marker support for OpenHarmony processes

## Capabilities

### New Capabilities

- `ohos-gateway-service`: Process management for OpenClaw gateway on OpenHarmony PC, including install, start, stop, restart, and status operations with PID-based tracking and multiple instance support.

### Modified Capabilities

- `gateway-process-detection`: Extend process detection to support `openharmony` platform (shares Linux-like `/proc` filesystem patterns).
- `supervisor-detection`: Extend supervisor marker detection to include OpenHarmony service hints.

## Impact

### New Files

- `src/daemon/ohos-service.ts` - OpenHarmony service adapter implementation
- `src/daemon/ohos-service.test.ts` - Unit tests for the adapter

### Modified Files

- `src/infra/gateway-processes.ts` - Add `openharmony` platform case for process args reading
- `src/infra/supervisor-markers.ts` - Add `ohos` supervisor type and hints
- `src/daemon/service.ts` - Register `openharmony` in `GATEWAY_SERVICE_REGISTRY`
- `src/infra/restart.ts` - Add OpenHarmony restart path
- `src/cli/daemon-cli/shared.ts` - Platform detection utilities

### File Layout (OpenHarmony-specific)

```
~/.openclaw/
├── gateway-<port>.env       # Service config per instance
├── gateway-<port>.pid       # PID file per instance
└── logs/
    ├── gateway-<port>.log   # stdout
    └── gateway-<port>.err   # stderr
```

### Dependencies

- No new npm dependencies required (Phase 1: pure TypeScript, PID-based)
- Phase 2 (future): May require NAPI bindings for native BackgroundTaskManager integration