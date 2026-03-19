## 1. Platform Detection Updates

- [x] 1.1 Add `openharmony` case to `readGatewayProcessArgsSync()` in `src/infra/gateway-processes.ts`
- [x] 1.2 Verify `findUnixGatewayPidsOnPortSync()` works for OpenHarmony (uses same Linux `/proc` and `ss` patterns)
- [x] 1.3 Add `ohos` to `RespawnSupervisor` type in `src/infra/supervisor-markers.ts`
- [x] 1.4 Add `SUPERVISOR_HINTS.ohos` with `OPENCLAW_OHOS_SERVICE_NAME` hint in `src/infra/supervisor-markers.ts`
- [x] 1.5 Add OpenHarmony case to `detectRespawnSupervisor()` in `src/infra/supervisor-markers.ts`

## 2. OpenHarmony Service Adapter

- [x] 2.1 Create `src/daemon/ohos-service.ts` with file path constants (`OHOS_SERVICE_DIR`, `OHOS_PID_FILE`, `OHOS_ENV_FILE`)
- [x] 2.2 Implement `resolveOhosServicePaths()` for port-based file naming (`gateway-<port>.env`, `gateway-<port>.pid`)
- [x] 2.3 Implement `writeOhosServiceConfig()` to write environment file with port, bind, and program arguments
- [x] 2.4 Implement `readOhosServiceConfig()` to parse environment file and return `GatewayServiceCommandConfig`
- [x] 2.5 Implement `installOhosService()` - write config, create directories
- [x] 2.6 Implement `uninstallOhosService()` - stop service, remove config and PID files
- [x] 2.7 Implement `startOhosService()` - spawn gateway process, write PID, redirect logs
- [x] 2.8 Implement `stopOhosService()` - read PID, verify process, send SIGTERM, wait for exit
- [x] 2.9 Implement `restartOhosService()` - read PID, send SIGUSR1 for in-process restart
- [x] 2.10 Implement `isOhosServiceLoaded()` - check PID file exists and process is alive
- [x] 2.11 Implement `readOhosServiceCommand()` - delegate to `readOhosServiceConfig()`
- [x] 2.12 Implement `readOhosServiceRuntime()` - return process status and port information

## 3. Service Registry Integration

- [x] 3.1 Add `openharmony` entry to `GATEWAY_SERVICE_REGISTRY` in `src/daemon/service.ts`
- [x] 3.2 Add `isSupportedGatewayServicePlatform()` check for `openharmony`
- [x] 3.3 Add OpenHarmony restart path in `src/infra/restart.ts` (signal-based restart)

## 4. CLI Integration

- [x] 4.1 Update `src/cli/daemon-cli/shared.ts` to handle `openharmony` platform for port parsing
- [x] 4.2 Verify `runDaemonStart/Stop/Restart/Uninstall` work with OpenHarmony adapter via registry

## 5. Tests

- [x] 5.1 Create `src/daemon/ohos-service.test.ts` with unit tests for:
  - [x] 5.1.1 Config file write/read round-trip
  - [x] 5.1.2 PID file management
  - [x] 5.1.3 Process spawning and signaling (mocked)
  - [x] 5.1.4 Multiple instance handling
  - [x] 5.1.5 Service status detection
- [x] 5.2 Add test for `openharmony` platform detection in `src/infra/gateway-processes.test.ts`
- [x] 5.3 Add test for `ohos` supervisor detection in `src/infra/supervisor-markers.test.ts`

## 6. Documentation

- [x] 6.1 Add OpenHarmony-specific notes to gateway service documentation (if applicable)
- [x] 6.2 Document OpenHarmony limitations (no auto-start in Phase 1)

## 7. Verification

- [ ] 7.1 Run `pnpm check` (lint, format, typecheck)
- [ ] 7.2 Run `pnpm test` to verify no regressions
- [ ] 7.3 Manual verification on HarmonyOS PC hardware (if available)