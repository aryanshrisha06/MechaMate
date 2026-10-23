# MechaMate — Robotics Lab Autopilot

A private, simulation-first robotics workbench for the Alexa+ hackathon track. A React voice/text interface uses a deterministic workflow agent that actually calls an MCP server over **MCP 2025-11-25 Streamable HTTP**. The agent reads project memory, looks up verified Uno specifications, checks telemetry and explains evidence. No paid API key or hardware is required.

**This is an Alexa+ experience simulator, not a live Alexa+ integration.** The agent is a transparent rule-based workflow, not an LLM. Nothing here is inherently exclusive to Alexa. No motor actuation, firmware upload, arbitrary code execution, or public deployment is implemented.

## Start the simulation

Install Node.js 22.12+ (24 LTS recommended) with npm. Open a terminal **in this folder**:

```sh
npm ci
npm run build
npm start
```

Open **http://127.0.0.1:4317** (use that exact host). Paste the operator token printed in the terminal into the page. Tokens are generated afresh each launch unless supplied through environment variables. Do not share terminal screenshots containing tokens.

Try:

1. `diagnose my lab` — four real MCP calls, with visible evidence and tool trace.
2. `remember A0 is grounded for the first demo`, then `memory`.
3. `simulate rail`, `simulate malformed`, `simulate stale`, then `simulate normal`.
4. `wiring` — manufacturer-sourced Uno R3 guidance.
5. `simulate motor` — inspect the proposal, type its exact confirmation phrase and confirm. This changes virtual state only.
6. `compile` — requires the optional Arduino CLI setup below; otherwise returns an honest compiler-unavailable error.

Dictate fills the message box for review; click Send to submit. Browser speech recognition is optional, browser-dependent and may send audio to the browser provider. Text always works. Reading replies aloud is opt-in. Neither dictation nor the agent can confirm an action.

## Tests

```sh
npm test
npm run build
npx playwright install chromium
npm run test:browser
```

Tests use temporary local directories under `work/`, ephemeral loopback ports and a fake serial adapter. They do not discover, open or write any physical port. Browser tests exercise the actual React → API → agent → MCP → lab path. See [validation results](docs/VALIDATION.md).

## Actual Uno firmware compilation

Install [Arduino CLI](https://arduino.github.io/arduino-cli/latest/installation/) and add it to PATH, or set `ARDUINO_CLI` to its absolute executable path.

```sh
arduino-cli core update-index
arduino-cli core install arduino:avr@1.8.6
npm run compile:firmware
```

The compiler always uses `firmware/telemetry` and `arduino:avr:uno`. It has a 60-second timeout and bounded output. It never passes an upload flag, port, user-supplied source or shell command. Compiler dependencies are installed by the operator, not by an MCP tool.

## Optional physical telemetry demo

Use an **Arduino UNO R3 (A000066)**, a USB data cable ending in USB Type B, and one jumper. This is not the Uno R4.

1. Disconnect USB and all external supplies. Remove every motor, driver, servo, relay and other load.
2. Inspect `firmware/telemetry/telemetry.ino`. Provision it manually using Arduino IDE only after explicitly choosing to replace the board’s existing firmware. Upload is intentionally outside this application.
3. Remove power again. Connect **A0 → GND**. Leave other headers unused.
4. Reconnect USB. Identify its port with your OS or Arduino IDE; close Serial Monitor.
5. Restart MechaMate with the exact port allowlisted:

```powershell
# Windows example; replace COM3 with the actual board port.
$env:SERIAL_ALLOWLIST = "COM3"
npm start
```

```sh
# Linux example; requires OS permission for this device.
SERIAL_ALLOWLIST=/dev/ttyACM0 npm start
```

6. In the UI enter `connect COM3` (or the Linux path). Review the exact port and reset warning, verify loads are disconnected, then type the displayed confirmation phrase and confirm within 60 seconds.
7. Expect schema-1 JSON lines at 115200 baud, approximately 5 samples/second, with A0 near zero. A rail finding is expected for this ground-reference test.
8. Use **Disconnect USB reader** to return to simulation.

Opening USB serial may toggle DTR and reset the Uno, running whatever firmware is already on it. “Read-only” means the application sends no serial payload bytes; it is not a claim of electrically passive operation. Serial opening is never automatic, including after restart or reconnect. No physical hardware was used during the delivered validation.

## MCP clients

The endpoint is `http://127.0.0.1:4317/mcp`. The separately printed MCP token authorizes MCP access; it **does not** authorize the operator confirmation API.

```powershell
$env:MCP_TOKEN = "<server MCP token>"
npm run agent -- "diagnose my lab"
```

POSIX: `MCP_TOKEN='<server MCP token>' npm run agent -- 'diagnose my lab'`.

The official SDK client performs initialization, initialized notification and tool calls. The server negotiates `2025-11-25`; subsequent requests must include `MCP-Protocol-Version: 2025-11-25`. It uses the spec’s stateless JSON-response option; GET/DELETE return 405. This is Streamable HTTP without optional SSE or resumability.

| Tool | Purpose |
|---|---|
| `read_telemetry` | Source, frame, age and invalid-frame count |
| `diagnose_lab` | Evidence-based stale/malformed/rail findings |
| `get_components` | Uno R3 specs, wiring and manufacturer references |
| `read_memory` / `save_note` | Persistent bounded project notes and audit history |
| `set_simulation` | Normal, rail, stale or malformed synthetic data |
| `propose_action` | Propose allowlisted serial open or virtual motor step |
| `compile_firmware` | Compile bundled sketch, never upload |

Resource: `mechamate://project/memory`. No confirmation tool exists.

## Configuration and limitations

| Variable | Default |
|---|---|
| `PORT` | `4317`, always loopback |
| `MCP_TOKEN` | Random per launch; ≥24 characters |
| `OPERATOR_TOKEN` | Different random token; ≥24 characters |
| `SERIAL_ALLOWLIST` | Empty; comma-separated exact port paths |
| `DATA_DIR` | `data` relative to project root |
| `ARDUINO_CLI` | `arduino-cli` on PATH |

Run one server process per data directory. JSON memory uses atomic replacement and keeps 100 notes and 500 audit events; it is not a multi-user database or tamper-proof audit log. Files and local bearer tokens assume a trusted operator account. Notes are treated as data. The service is intentionally local and does not implement public OAuth/TLS, tenant isolation or internet rate limiting.

## Documentation and submission

- [Exact hardware/software architecture](docs/ARCHITECTURE.md)
- [APIs, SDKs and tools actually used + Product Feedback](docs/TECHNOLOGY.md)
- [Safety and threat boundaries](docs/SAFETY.md)
- [Hackathon requirements and demo script](docs/HACKATHON.md)
- [Project memory and decisions](PROJECT_MEMORY.md)
- [Verification record](docs/VALIDATION.md)

All source and a dependency lockfile are included. No repository, website, video or submission has been published. The code includes an MIT license for a possible open-source release; publishing and granting reviewer access remain the owner’s choice.
