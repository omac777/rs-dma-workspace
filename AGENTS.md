# AGENTS.md

You are an expert Rust and Slint developer working on the `rs-dma-workspace`.

## Toolchain & Build Rules
- Always use `cargo +nightly` for building and testing.
- The workspace contains `rs-dma-lib` (core logic) and `rs-dma-uml-collab-diagram` (Slint GUI).
- Never use `mitts`. Use standard `cargo`.

## UI Testing & MCP Driving
- The Slint app has a built-in MCP server for UI automation.
- To launch the app for testing, run:
  `SLINT_EMIT_DEBUG_INFO=1 SLINT_MCP_PORT=8080 cargo +nightly run -p rs-dma-uml-collab-diagram --features slint/mcp`
- Do NOT set `SLINT_BACKEND=headless`. The `slint/mcp` feature pulls in `renderer-software`, and
  Slint's software renderer has `rotate()`/`scale()` as no-ops (`// TODO (#6068)`). Headless would
  silently flatten every rotated `AssociationLine`, `MessageLabel` and `YellowDebugRect`, so the
  diagram would render as horizontal bars instead of the A-B-C triangle. Let Slint pick its
  default backend (Skia/FemtoVG), which honours `transform-rotation`.
- Use the `slint-uml` MCP server tools to interact with the running app:
  - `list_windows` to find the active window.
  - `take_screenshot` to capture the current UI state.
  - `click_element` or `type_text` to simulate user input.
  - `get_element_properties` to verify `@testable` state.
- After making UI changes, always rebuild, relaunch the app, and verify the change via MCP before declaring a task complete.
- Verify rendering from the captured pixels, never from the element tree alone: the tree reports the
  unrotated geometry, so a broken transform is invisible in it but obvious in a screenshot.
