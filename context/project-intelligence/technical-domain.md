<!-- Context: project-intelligence/technical | Priority: critical | Version: 1.1 | Updated: 2026-09-12 -->

# Technical Domain

**Purpose**: Tech stack, architecture, and development patterns for VoltraBloom (ESP32 IoT + Web Dashboards).
**Last Updated**: 2026-09-11

## Quick Reference
**Update Triggers**: Tech stack changes | New patterns | Architecture decisions
**Audience**: Developers, AI agents

## Primary Stack

| Layer | Technology | Version | Rationale |
|-------|-----------|---------|-----------|
| Firmware | C++ Arduino + FreeRTOS | ESP32 core | Dual-core IoT with real-time sensing |
| Web Dashboards | Vanilla HTML + Tailwind (CDN) | Tailwind CDN | No build system, plain `.html` files |
| 3D Visualization | Three.js | r128 | GLB model loading + procedural geometry |
| Charts | Chart.js | 4.4.0 | Telemetry time-series display |
| Animation | GSAP | 3.12.5 | Smooth UI transitions |
| Backend | Supabase (PostgreSQL + REST) | — | Cloud telemetry storage + realtime |
| Deployment | Vercel (static hosting) | — | `npx vercel --prod`, no build step |
| WebSocket server | ESPAsyncWebServer | — | Live device control (adopting, see `technical-patterns.md`) |
| Firmware build | PlatformIO | — | Env-based builds when deps grow (adopting) |
| Serial debug | WebSerial | — | Browser console for firmware (adopting) |

## Architecture Pattern

```
Type: Hybrid (IoT + Web)
Pattern: ESP32 sensors → Supabase cloud → HTML dashboards
- ESP32 reads ADC1 sensors at 100 Hz on Core 1
- WiFi + REST + Supabase upload on Core 0
- HTML pages pull from Supabase via anon key (client-side)
```

## Project Structure

```
VOLTRA/
├── firmware/
│   ├── Project_Voltrabloom_Unified/   ← Recommended (dual-core, FreeRTOS)
│   ├── Project_Voltrabloom/           ← Older WiFi AP variant
│   ├── Project_Voltrabloom_Supabase/  ← Cloud-only logger
│   └── tester_pertama/                ← Sensor calibration, no WiFi
├── index.html                          ← Main live 3D dashboard
├── viewer_3d.html                      ← Standalone Three.js GLB viewer
├── box_akrilik_designer.html           ← GLB showcase + HEMS controls
├── main.js + style.css                ← Production build (needs HTTP server)
├── 3d_models/                          ← .glb / .stl / .svg hardware models
├── media/                              ← Logos, photos, screenshots
├── documents/                          ← Papers, proposals, SQL schemas
├── gallery/                            ← Optimized gallery assets
├── tools/Arduino IDE/                  ← Vendored portable Arduino IDE
└── vercel.json                         ← Vercel deploy config
```

## API Patterns

**Firmware → Supabase (ESP32 REST):**
```cpp
HTTPClient http;
http.begin(SUPABASE_URL "/rest/v1/telemetry");
http.addHeader("Content-Type", "application/json");
http.addHeader("apikey", SUPABASE_KEY);
http.addHeader("Authorization", "Bearer " + String(SUPABASE_KEY));
http.POST(jsonPayload);
```

**Web → Supabase (JS client):**
```javascript
const { data, error } = await supabase
  .from('telemetry')
  .select('*')
  .order('created_at', { ascending: false })
  .limit(100);
```

## Component Patterns

**HTML Dashboard Page:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0"></script>
  <script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5"></script>
</head>
<body class="bg-slate-900 text-white min-h-screen">
  <div id="three-container"></div>
  <canvas id="telemetry-chart"></canvas>
  <script>
    // Three.js scene + Chart.js config + Supabase realtime
  </script>
</body>
</html>
```

**Three.js 3D Viewer:**
```javascript
const scene = new THREE.Scene();
const loader = new THREE.GLTFLoader();
loader.load('3d_models/Frantic_Kasi_v1.glb', (gltf) => {
  scene.add(gltf.scene);
});
```

## Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| HTML files | `snake_case` / `kebab-case` | `box_akrilik_designer.html` |
| Firmware folders | `PascalCase` + prefix | `Project_Voltrabloom_Unified/` |
| Firmware `.ino` | Match folder name exactly | `Project_Voltrabloom_Unified.ino` |
| C++ headers | `PascalCase.h` | `SupabaseLogger.h`, `ArduinoCompat.h` |
| JS files | `camelCase.js` | `main.js` |
| CSS files | `kebab-case.css` | `style.css` |
| Pin constants | `camelCase` | `pinSolar`, `pinWind`, `pinSoil` |
| FreeRTOS tasks | `camelCase` | `vSensorTask`, `vWiFiTask` |
| SQL tables | `snake_case` | `telemetry`, `telemetry_hourly` |
| 3D models | `PascalCase` + `_snake_case` | `Frantic_Kasi_v1.glb` |

## Code Standards

1. **ADC1 only** — all analog sensors on GPIO 32, 33, 34, 35, 36/VP, 39/VN. ADC2 unusable with WiFi.
2. **Float division** — never `sum / count` as integers. Always `(((float)sum / count) / 4095.0) * VREF`.
3. **Arduino sketch layout** — `.ino` must live in folder of exact same name.
4. **Headers (.h)** — declare types only. Never put global object instances in headers.
5. **Supabase TLS** — use `client.setInsecure()`. Do not "fix" with cert fingerprint.
6. **Three.js CDN** — explicit `.js` endpoints. `.closePath()` on shapes. Builders in `window.onload`.
7. **HTML output** — clean HTML5, never wrapped in markdown code fences.
8. **FreeRTOS** — dual-core, mutex-protected `SystemTelemetry`, `xTaskCreatePinnedToCore`.
9. **Float math** — always `(float)` casts, never integer division.
10. **No build system** — plain `.ino` and `.html`, no package manager, no bundler.

## Security Requirements

1. **Supabase anon key only** — never paste `service_role` key in client-side code.
2. **Hard-coded WiFi creds** are default/editable per device — not production secrets.
3. **TLS `setInsecure()`** — acceptable for this project, do not add cert fingerprints.
4. **No dashboard auth** — `index.html` and viewers are public, no login required.
5. **Vercel static** — no server-side code, no secrets in deployment config.
6. **`.env.local`** — gitignored, contains `VERCEL_OIDC_TOKEN`.

## 📂 Codebase References

**Firmware**: `firmware/Project_Voltrabloom_Unified/Project_Voltrabloom_Unified.ino` — main dual-core sketch
**Headers**: `firmware/Project_Voltrabloom_Unified/SupabaseLogger.h` — cloud upload helper
**Compat**: `firmware/Project_Voltrabloom_Unified/ArduinoCompat.h` — IDE + clangd shim
**Dashboard**: `index.html` — main live 3D telemetry dashboard
**3D Viewer**: `viewer_3d.html` — standalone GLB viewer (loads `Frantic_Kasi_v1.glb`)
**Designer**: `box_akrilik_designer.html` — GLB showcase + animated HEMS controls
**Production Build**: `main.js` + `style.css` — ES module build (needs HTTP server)
**3D Models**: `3d_models/Frantic_Kasi.glb` (optimized) + `Frantic_Kasi_v1.glb` (original)
**SQL Schema**: `documents/schemas/supabase_telemetry_schema.sql` — telemetry tables + functions
**Deploy Config**: `vercel.json` — static hosting, immutable cache, no build step

## Key Technical Decisions

| Decision | Rationale | Impact |
|----------|-----------|--------|
| No build system | Plain `.html` opened directly in browser | Zero setup friction, CDN-only deps |
| FreeRTOS dual-core | 100 Hz ADC sampling needs dedicated core | Sensor reliability + WiFi coexistence |
| `setInsecure()` TLS | ESP32 resource constraints, project scope | Simpler code, acceptable risk |
| Supabase over custom backend | Managed PostgreSQL + REST + realtime | No server to maintain |
| GLB models over procedural | Realistic hardware visualization | Better visual fidelity |

See `decisions-log.md` for full decision history with alternatives.

## Related Files

- `business-domain.md` — Why this technical foundation exists
- `business-tech-bridge.md` — How business needs map to solutions
- `decisions-log.md` — Full decision history with context
- `technical-patterns.md` — Advanced patterns (realtime, WebSocket, RLS, build)
