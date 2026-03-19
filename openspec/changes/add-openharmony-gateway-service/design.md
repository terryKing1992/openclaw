## Context

OpenClaw's daemon management system uses a platform-adapter pattern:

```
src/daemon/service.ts
├── darwin  → launchd.ts   (uses launchctl CLI)
├── linux   → systemd.ts   (uses systemctl CLI)
└── win32   → schtasks.ts  (uses schtasks.exe CLI)
```

Each adapter implements the `GatewayService` interface:
- `install` / `uninstall` - Service registration
- `start` / `stop` / `restart` - Lifecycle control
- `isLoaded` - Status check
- `readCommand` / `readRuntime` - Configuration introspection

OpenHarmony presents a unique challenge:
- No CLI tool like `launchctl`/`systemctl` for service management
- Node.js port exists with `process.platform === 'openharmony'`
- Linux-like `/proc` filesystem available
- Signal handling (SIGTERM/SIGUSR1) expected to work
- Native BackgroundTaskManager API exists but requires NAPI bindings

## Goals / Non-Goals

**Goals:**
- Enable `openclaw daemon install/start/stop/restart/status` on OpenHarmony PC
- Support multiple gateway instances (port-based)
- Maintain consistency with existing daemon CLI interface
- Pure TypeScript implementation (no native modules in Phase 1)

**Non-Goals:**
- Auto-start on boot (Phase 2: requires native app wrapper with `ohos.permission.STARTUP_APPLICATION`)
- Native BackgroundTaskManager integration (Phase 2)
- GUI integration with HarmonyOS system settings

## Decisions

### D1: PID-Based Service Management (Phase 1)

**Decision:** Implement service management using PID files and process signals, without system-level service registration.

**Rationale:**
- No CLI tool available for service management on OpenHarmony
- Existing codebase already has PID-based patterns for "unmanaged" gateways
- Allows testing and validation on real HarmonyOS PC hardware
- Provides migration path to native integration later

**Alternatives considered:**
- Native BackgroundTaskManager wrapper (requires NAPI bindings, higher complexity)
- Shell script wrapper (less portable, harder to test)

### D2: Port-Based Instance Naming

**Decision:** Use port number in file naming: `gateway-<port>.pid`, `gateway-<port>.env`

**Rationale:**
- Consistent with how multiple instances are distinguished elsewhere
- Allows parallel instances on different ports
- Simplifies cleanup and status checks

### D3: Service Config Storage

**Decision:** Store service configuration in `~/.openclaw/gateway-<port>.env` as key-value pairs.

**Rationale:**
- Simple to read/write from TypeScript
- Compatible with existing config patterns
- Easy to debug (plain text)

**Alternatives considered:**
- JSON file (more structured but unnecessary for simple config)
- Integrated into main OpenClaw config (mixes concerns)

### D4: Platform Detection Strategy

**Decision:** Treat `openharmony` platform as Linux-like for process operations, but as distinct platform for service management.

**Rationale:**
- `/proc` filesystem available for process discovery
- `ss`/`lsof` commands expected to work (same as Linux)
- But service management is fundamentally different (no systemd)

### D5: Signal-Based Restart

**Decision:** Use SIGUSR1 for in-process restart (same as unmanaged Linux gateways).

**Rationale:**
- Existing pattern in `run-loop.ts`
- Avoids full process respawn complexity
- Preserves TCC permissions on macOS-like systems

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GatewayService Interface                          │
├─────────────────────────────────────────────────────────────────────┤
│  label, loadedText, notLoadedText                                   │
│  install, uninstall, stop, restart, isLoaded                        │
│  readCommand, readRuntime                                            │
└─────────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    ohos-service.ts                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  State Files:                                                       │
│  ─────────────                                                      │
│  ~/.openclaw/gateway-<port>.env  # port, bind, args                │
│  ~/.openclaw/gateway-<port>.pid  # running process ID              │
│  ~/.openclaw/logs/gateway-<port>.log  # stdout                     │
│  ~/.openclaw/logs/gateway-<port>.err  # stderr                     │
│                                                                     │
│  Operations:                                                        │
│  ───────────                                                        │
│  install()  → Write .env file, create directories                  │
│  uninstall() → Remove .env and .pid files                          │
│  start()    → Read config, spawn process, write .pid               │
│  stop()     → Read .pid, verify gateway process, send SIGTERM     │
│  restart()  → Read .pid, send SIGUSR1 (in-process restart)        │
│  isLoaded() → Check .pid file exists and process is alive          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    gateway-processes.ts (extended)                  │
├─────────────────────────────────────────────────────────────────────┤
│  readGatewayProcessArgsSync(pid):                                   │
│    if (platform === 'openharmony') → /proc/${pid}/cmdline          │
│                                                                     │
│  findUnixGatewayPidsOnPortSync(port):                               │
│    Works for both 'linux' and 'openharmony'                        │
└─────────────────────────────────────────────────────────────────────┘
```

## Risks / Trade-offs

### Risk: No Auto-Restart on Crash

**Impact:** If the gateway crashes, it won't automatically restart.

**Mitigation:** 
- Document that users should monitor the process
- Phase 2: Native app wrapper with BackgroundTaskManager provides auto-restart

### Risk: Signal Handling Compatibility

**Impact:** SIGTERM/SIGUSR1 may not work as expected on OpenHarmony Node.js.

**Mitigation:**
- Test on real HarmonyOS PC hardware early
- Fall back to full stop/start if SIGUSR1 fails

### Risk: Process Orphaning

**Impact:** If the CLI process dies unexpectedly, the gateway may become orphaned.

**Mitigation:**
- PID files allow discovery of orphaned processes
- `openclaw gateway status --deep` can detect and clean up

### Trade-off: No System Integration

**Impact:** Gateway won't appear in system task manager or settings.

**Mitigation:**
- Phase 2: Native integration provides system visibility
- Current approach maintains CLI consistency across platforms

## Migration Plan

### Deployment

1. No migration needed - new platform support
2. Users install OpenClaw on OpenHarmony PC
3. Run `openclaw daemon install` to set up service config
4. Run `openclaw daemon start` to start gateway

### Rollback

1. `openclaw daemon stop`
2. `openclaw daemon uninstall`
3. Remove `~/.openclaw/` directory if needed

## Open Questions

1. **Signal behavior verification:** Does Node.js on OpenHarmony fully support SIGTERM and SIGUSR1? (Needs testing on real hardware)

2. **Process spawning:** Does `child_process.spawn` with detached/nohup work reliably? (Needs testing)

3. **Log rotation:** Should we implement log rotation for `gateway-<port>.log` files? (Currently left to user)

4. **Phase 2 timeline:** When to implement native BackgroundTaskManager integration? (Depends on user feedback and demand)