<!-- Context: project-intelligence/patterns | Priority: critical | Version: 1.0 | Updated: 2026-09-12 -->

# Technical Patterns (Advanced)

**Purpose**: Real-time, async, build, and security patterns layered on top of the core stack. Read after `technical-domain.md`.
**Last Updated**: 2026-09-12

## Quick Reference
**Update Triggers**: New realtime feature | Build tooling change | New security rule
**Audience**: Developers, AI agents building firmware + web features

## 1. Realtime & Async Patterns

**Concept**: Two live paths — ESP32 pushes (REST/WebSocket), browser subscribes (Supabase Realtime). REST = telemetry ingest; WebSocket = direct device control; Realtime = dashboard push.

### Firmware — WebSocket server (target, adopt when adding live control)
```cpp
#include <ESPAsyncWebServer.h>
AsyncWebServer server(80);
AsyncWebSocket ws("/ws");
void onWsEvent(AsyncWebSocket* s, AsyncWebSocketClient* c, AwsEventType t, void* arg, uint8_t* data, size_t len) {
  if (t == WS_EVT_DATA) { /* JSON command → relay/HEMS control */ }
}
void setupServer() { ws.onEvent(onWsEvent); server.addHandler(&ws); server.begin(); }
```

### Firmware — REST upload (existing, keep)
`SupabaseLogger.h` — HTTPClient POST to `/rest/v1/telemetry` with `apikey` + `Authorization: Bearer`.

### Web — Supabase Realtime (target)
```javascript
const channel = supabase.channel('telemetry-live')
channel.on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'telemetry' },
  (payload) => appendPoint(payload.new)).subscribe()
```
Live dashboard only. History stays on REST `.select()` (existing pattern).

### Web/Edge — RPC aggregate (target)
```javascript
const { data, error } = await supabase.rpc('telemetry_daily_aggregate', { device_id });
```
Server-side aggregation feeding `telemetry_hourly` intent.

## 2. Firmware Build & Debug

- **PlatformIO** (adopting) — `platformio.ini` per env when library deps grow; keeps `.ino`-in-same-name-folder rule.
- **WebSerial** (adopting) — browser serial console:
```cpp
#include <WebSerial.h>
WebSerial.begin(&server); WebSerial.onMessage(onWsMsg);
WebSerial.println("boot ok");
```

## 3. UI Component Patterns

- **Setup/Config page** — standalone `.html` (dark Tailwind CDN, no build): SSID, pool, calibration sliders; sends via REST or WebSocket to device.
- **History dashboard** — Chart.js time-series from `telemetry_hourly`, day/week grouping; reuse chart config style from `index.html`.

## 4. Naming — Additions

| Type | Convention | Example |
|------|-----------|---------|
| WS/class | `PascalCase` | `WsControlHandler` |
| Realtime channel | `snake_case` + `-live` | `telemetry-live` |
| RPC functions | `snake_case` verb-noun | `telemetry_daily_aggregate` |
| PlatformIO envs | `kebab-case` | `esp32-dev`, `esp32-prod` |

## 5. Standards — Additions

1. **Watchdog** — `esp_task_wdt_add()` on sensor task; feed each loop; reboot on stall.
2. **Error handling** — POST retry w/ exponential backoff (1→2→4 s, max 3); buffer telemetry during failure, flush on reconnect; log via WebSerial, never drop silently.
3. **Power/task mgmt** — duty-cycled sampling on Core 1; cap WiFi reconnects; use `vTaskDelay`, never blocking `delay()` in tasks.

## 6. Security — Additions

1. **Supabase RLS** — enable RLS on `telemetry`; anon `INSERT` from device; restricted `SELECT`; policies added in the same migration as table DDL. Never `service_role` client-side.
2. **Credential rotation** — SUPABASE_KEY / WiFi creds per-device rotatable; `.env.local` (`VERCEL_OIDC_TOKEN`) treated as replaceable secret.
3. **Rate limiting** — RPC guard: `select count(*) from telemetry where created_at > now() - interval '1 minute'`; reject over-limit inserts per device.
4. Existing rules still apply: anon-key-only, public dashboards, `setInsecure()` TLS (no cert fingerprints).

## 📂 Codebase References

**Existing**
- `firmware/Project_Voltrabloom_Unified/Project_Voltrabloom_Unified.ino` — dual-core sketch (extend with WS server)
- `firmware/Project_Voltrabloom_Unified/SupabaseLogger.h` — REST upload helper
- `firmware/Project_Voltrabloom_Unified/SensorReader.h` — ADC1 sensor sampling
- `index.html` — main live dashboard (add Realtime + history here)
- `documents/schemas/supabase_telemetry_schema.sql` — tables + functions (add RLS here)

**Planned (not yet implemented — apply per patterns above)**
- WebSocket handler + WebSerial console (firmware) · `platformio.ini`
- Realtime channel + RPC subscription (web)

## Related Files

- `technical-domain.md` — core stack & base patterns
- `decisions-log.md` — decision history