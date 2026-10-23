# Project memory — 2026-10-20

## User intent

Build MechaMate Robotics Lab Autopilot for the Amazon Alexa+ hackathon. Deliver a realistic testable MCP prototype, exact hardware/software architecture, source and run instructions. Start without hardware; target Uno USB telemetry; require confirmation for hardware actions. No public deployment or publication without consent.

The user confirmed the applicable event is Build, Ship, Shape: Amazon Developer Hackathon with no extra constraints, and specifically requested the APIs, SDKs and tools used.

## Decisions

- Node/TypeScript and the official MCP SDK; requested protocol revision 2025-11-25.
- React voice/text simulator; no unsupported claim of live Alexa+ or Alexa exclusivity.
- Deterministic multi-tool agent for reproducibility and no paid API key requirement.
- Uno R3 A000066, USB Type B, A0 grounded for a known-input first demo.
- No motors, driver boards or external power supplies in the hardware BOM.
- Serial open is gated because it can reset the board. Empty port allowlist by default.
- No upload endpoint; compiler accepts only the bundled sketch.
- Separate operator token and MCP token; approval not exposed to the agent.
- Local bounded JSON memory; single-user/single-process scope.
- GitHub-ready package only; repository creation, invitations, video upload and submission remain undone.

## Runtime memory

`data/memory.json` is created on launch/use and excluded from source control. Notes persist across restarts; proposals and serial connections do not. Audit history records proposals, confirmations, serial outcomes and memory changes. Deleting the data directory manually resets project notes; the application has no silent reset action.

## Remaining external validation

Actual Uno USB behavior and real microphone recognition require the user's devices. Official Alexa+ connection requires access to the applicable Amazon integration program. Final hackathon eligibility, submission text and Product Feedback should be reviewed by the entrant.
