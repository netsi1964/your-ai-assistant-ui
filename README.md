# your-ai-assistant-ui

An AI assistant UI that helps you get your jobs done on any website — by understanding the website's structure for you.

## The idea

The core idea is to provide an AI-powered assistant that can analyse any website and generate a structured JSON overview of its pages and functions. For each page or interactive element (e.g. a button, a link, a form), the assistant produces a human-readable description of what it contains and what it can do.

This is driven by a prompt such as:

> "Jeg har brug for at du hjælper med at detektere et JSON overblik over sider og funktioner på denne hjemmeside, for hver fundet side eller funktion (f.eks. en knap) skal der være en tekst beskrivelse af hvad den indeholder, hvad den kan."
>
> *(English: "I need you to help detect a JSON overview of pages and functions on this website. For each found page or function (e.g. a button) there should be a text description of what it contains and what it can do.")*

## Example output

The assistant produces a structured JSON document with two top-level sections:

- **`pages`** — a list of pages discovered on the site, each with a `title`, `url`, `type`, `description`, a list of things it `contains`, and a list of things it `can` do.
- **`functions`** — a list of interactive UI elements (buttons, links, search fields, etc.), each described with a `label`, `type`, `location`, `description`, and a list of things it `can` do.

```json
{
  "site": {
    "name": "Example Site",
    "baseUrl": "https://www.example.com",
    "sourcePage": "https://www.example.com/some-page/",
    "scope": "seeded-from-current-page-and-main-site-sections"
  },
  "pages": [
    {
      "title": "Home",
      "url": "https://www.example.com/",
      "type": "hub",
      "description": "Main entry point for visitors. Brings together key content areas.",
      "contains": [
        "Navigation to main sections",
        "Featured content"
      ],
      "can": [
        "Guide visitors to relevant content",
        "Provide quick access to key tools"
      ]
    }
  ],
  "functions": [
    {
      "label": "Log in",
      "type": "button",
      "location": "header",
      "description": "Opens the login flow for registered users.",
      "can": [
        "Start the login flow",
        "Give access to personalised content"
      ]
    }
  ]
}
```

## Why this is useful

- **Automated site mapping** — instead of manually documenting a website, the AI assistant generates a structured overview in seconds.
- **AI context injection** — the JSON output can be fed back into an AI assistant as context, so it can answer user questions about the website accurately.
- **Accessibility and onboarding** — the descriptions make it easy for new users or developers to understand what a website offers without having to browse every page.
- **Integration-ready** — the structured JSON format makes it easy to integrate with other tools, chatbots, documentation systems, or testing pipelines.

## Technical foundation — WebMCP, Android and iOS

The JSON sitemap produced by this tool is the *description layer*. To make a website or app fully agent-ready, it pairs with a platform-level *execution layer* so that AI agents can not only understand the site but also act on it.

### Web — WebMCP

[WebMCP](https://github.com/webmcp/webmcp) lets websites expose functions (tools) directly to AI agents in the browser. Instead of scraping HTML or simulating clicks, agents call these tools in a structured and reliable way — exactly as described by the JSON output.

### Android — AppFunctions (available now)

**AppFunctions** is the official Android equivalent of WebMCP. Built into Android 16 (with a Jetpack backport), it lets apps expose self-describing tools using the `@AppFunction` annotation. A schema is generated automatically at build time, and AI agents like Gemini can call the functions directly via natural language.

```kotlin
@AppFunction(isDescribedByKDoc = true)
suspend fun checkPensionScore(
    appFunctionContext: AppFunctionContext,
    userId: String? = null
): PensionScoreResult { /* … */ }
```

### iOS — App Intents + MCP (iOS 26.1+)

Apple is integrating MCP support directly into the existing **App Intents** framework (spotted in iOS 26.1 beta). Once stable, apps can expose their functions via App Intents and MCP-compatible agents (ChatGPT, Claude, etc.) will call them without any extra code.

```swift
struct CheckPensionScoreIntent: AppIntent {
    static var title: LocalizedStringResource = "Check Pension Score"
    func perform() async throws -> some IntentResult { /* … */ }
}
```

### Any app today — mobile-mcp

[mobile-mcp](https://github.com/mobile-next/mobile-mcp) is an open-source bridge that lets AI agents automate *any* iOS or Android app today — no changes to the target app required. It uses accessibility APIs and screen coordinates as a bridge until native MCP support is universal.

### Platform overview

| Platform | Technology | Status |
|---|---|---|
| Web | WebMCP / MCP | Available now |
| Android | AppFunctions (`@AppFunction`) | Available now (Android 16 + Jetpack) |
| iOS | App Intents + MCP | In beta (iOS 26.1) |
| iOS & Android (any app) | mobile-mcp (bridge) | Available now |

For detailed documentation and links, see [`research/ios-android.md`](research/ios-android.md).

## Implementation examples

Below is a quick snapshot of how the sitemap concept maps to real platform integrations. Full examples with setup instructions are in [`research/code-examples.md`](research/code-examples.md).

### Android — `@AppFunction`

```kotlin
@AppFunction(isDescribedByKDoc = true)
suspend fun fetchPensionsInfo(
    appFunctionContext: AppFunctionContext,
    externalProvider: String
): PensionsInfoResult {
    val data = pensionsInfoService.fetchFromProvider(externalProvider)
    return PensionsInfoResult(data)
}
```

### iOS — `AppIntent`

```swift
struct FetchPensionsInfoIntent: AppIntent {
    static var title: LocalizedStringResource = "Fetch PensionsInfo"
    @Parameter(title: "Provider") var provider: String

    func perform() async throws -> some IntentResult & ReturnsValue<PensionsInfoResult> {
        let data = await PensionsInfoService.shared.fetch(from: provider)
        return .result(value: PensionsInfoResult(data: data))
    }
}
```

### mobile-mcp — MCP server config

```json
{
  "mcpServers": {
    "mobile-mcp": {
      "command": "npx",
      "args": ["-y", "@mobilenext/mobile-mcp@latest"]
    }
  }
}
```

### How the pieces fit together

```
LLM Sitemap JSON (this project)
        │
        ├── Web:      WebMCP tools exposed on the website
        ├── Android:  AppFunctions (@AppFunction annotations)
        ├── iOS:      App Intents + MCP (iOS 26.1+)
        └── Any app:  mobile-mcp bridge (works today)
```

The sitemap tells the agent *what exists* and *what it can do*.
The platform integration tells the agent *how to do it*.
