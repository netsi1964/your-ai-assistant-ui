# WebMCP and Mobile Equivalents

This document covers WebMCP for the web and the equivalent technologies for iOS and Android, which form part of the technical foundation for the AI assistant concept.

## WebMCP (Web)

WebMCP lets websites expose functions (tools) directly to AI agents running in the browser. Instead of scraping HTML or simulating clicks, AI agents can call these functions in a structured and reliable way — exactly as described by the JSON site-map output in this project.

- **How it works**: A website annotates its own capabilities using the Model Context Protocol (MCP). An AI agent in the browser discovers those capabilities and calls them directly.
- **Why it matters here**: The LLM sitemap JSON produced by this tool is the *description layer*. WebMCP is the *execution layer* — together they give an AI agent both the map of what exists and the ability to act on it.

## Android — AppFunctions (available now)

**AppFunctions** is the official mobile equivalent of WebMCP/MCP on Android.

- Built into **Android 16**, with a Jetpack library (`androidx.appfunctions`) for older versions.
- Apps expose specific functions (e.g. "create note", "add calendar event", "check pension score") as self-describing tools.
- AI agents like Gemini can discover and call them directly via natural language — exactly like WebMCP on the web.
- Developers annotate functions with `@AppFunction` and a schema is generated automatically at build time.

This is Google/Android's direct answer to WebMCP, making apps "agent-ready" at the OS level.

### Relevant links

| Resource | URL |
|---|---|
| Official documentation | https://developer.android.com/ai/appfunctions |
| Jetpack library releases | https://developer.android.com/jetpack/androidx/releases/appfunctions |
| Platform API reference | https://developer.android.com/reference/android/app/appfunctions/package-summary |
| Example app on GitHub | https://github.com/FilipFan/AppFunctionsPilot |

## iOS — MCP via App Intents (coming soon)

Apple is actively building **MCP support** on top of their existing **App Intents** framework.

- Spotted in **iOS 26.1 beta** (and macOS Tahoe 26.1).
- Once released, apps will expose their functions via App Intents, and MCP-compatible agents (ChatGPT, Claude, etc.) will be able to call them directly — without any extra MCP code from developers.
- Still in early beta as of March 2026, so not yet available to regular users.

### Relevant links

| Resource | URL |
|---|---|
| 9to5Mac article (most detailed on the implementation) | https://9to5mac.com/2025/09/22/macos-tahoe-26-1-beta-1-mcp-integration/ |
| Apple App Intents documentation (the foundation MCP builds on) | https://developer.apple.com/documentation/appintents |

## mobile-mcp — Open-source bridge (works today on all devices)

If you want agent control over any app *today*, regardless of whether the app natively supports MCP, **mobile-mcp** is an open-source bridge that works on both iOS and Android (physical devices, simulators, and emulators).

- AI agents control apps via accessibility APIs, screenshots, or screen coordinates.
- Think of it as a "bridge" rather than a native integration — powerful, but less clean than AppFunctions or App Intents.
- Works right now, with no changes needed to the target app.

### Relevant links

| Resource | URL |
|---|---|
| Main repository | https://github.com/mobile-next/mobile-mcp |
| Installation wiki | https://github.com/mobile-next/mobile-mcp/wiki |

## Summary

| Platform | Technology | Status |
|---|---|---|
| Web | WebMCP / MCP | Available now |
| Android | AppFunctions (`@AppFunction`) | Available now (Android 16 + Jetpack) |
| iOS | App Intents + MCP | In beta (iOS 26.1) |
| iOS & Android (any app) | mobile-mcp (bridge) | Available now (open source) |
