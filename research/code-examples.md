# Implementation Code Examples

This document shows concrete code examples for implementing the AI assistant concept across web, Android, and iOS, as well as the mobile-mcp bridge approach.

The examples follow the same pattern as the LLM sitemap JSON — exposing functions that AI agents can discover and call. The function names used here are illustrative and map to the kind of actions described in the sitemap (e.g. "Check pension score", "Fetch PensionsInfo").

---

## 1. Android — AppFunctions (native, available in Android 16+)

Use the `@AppFunction` annotation from the Jetpack library (`androidx.appfunctions`). A schema is generated automatically at build time, and AI agents (e.g. Gemini) can discover and call the functions directly via natural language.

### Gradle setup

```kotlin
// build.gradle.kts (app module)
dependencies {
    implementation("androidx.appfunctions:appfunctions:1.0.0-alpha08")
    implementation("androidx.appfunctions:appfunctions-service:1.0.0-alpha08")
    ksp("androidx.appfunctions:appfunctions-compiler:1.0.0-alpha08")
}
```

### Example 1: Expose a "Check pension score" function

```kotlin
import androidx.appfunctions.AppFunction
import androidx.appfunctions.AppFunctionContext
import androidx.appfunctions.AppFunctionSerializable

/**
 * Returns a score indicating the strength of the user's pension savings
 * relative to their current salary and expected retirement period.
 */
@AppFunction(isDescribedByKDoc = true)
suspend fun checkPensionScore(
    appFunctionContext: AppFunctionContext,
    userId: String? = null  // optional — resolved from context if not provided
): PensionScoreResult {
    val score = pensionRepository.getPensionScore(
        userId ?: appFunctionContext.callingPackage
    )
    return PensionScoreResult(
        score = score.value,
        explanation = score.explanation,
        recommendation = score.recommendation
    )
}

@AppFunctionSerializable(isDescribedByKDoc = true)
data class PensionScoreResult(
    /** Numeric score from 0–100 representing pension strength. */
    val score: Int,
    /** Plain-language explanation of the score. */
    val explanation: String,
    /** Optional recommendation for improvement. */
    val recommendation: String?
)
```

The KDoc comments become the natural-language descriptions visible to the AI agent — exactly like the `description` and `can` fields in the LLM sitemap JSON.

### Example 2: Expose a "Fetch external pension data" function

```kotlin
/**
 * Fetches pension data from an external provider (e.g. ATP) to create
 * a consolidated overview of the user's total pension savings.
 */
@AppFunction(isDescribedByKDoc = true)
suspend fun fetchPensionsInfo(
    appFunctionContext: AppFunctionContext,
    /** The external provider to fetch from, e.g. "ATP" or "Danica". */
    externalProvider: String
): PensionsInfoResult {
    val data = pensionsInfoService.fetchFromProvider(externalProvider)
    return PensionsInfoResult(
        provider = externalProvider,
        totalSavings = data.totalAmount,
        currency = data.currency
    )
}

@AppFunctionSerializable(isDescribedByKDoc = true)
data class PensionsInfoResult(
    val provider: String,
    val totalSavings: Double,
    val currency: String
)
```

---

## 2. iOS — App Intents + MCP (via iOS 26.1 beta and later)

Apple integrates MCP directly into the existing **App Intents** framework. You define intents as normal — MCP makes them automatically available to external agents (ChatGPT, Claude, etc.) without any additional code.

### Example 1: AppIntent for "Check pension score"

```swift
import AppIntents

struct CheckPensionScoreIntent: AppIntent {
    static var title: LocalizedStringResource = "Check Pension Score"
    static var description = IntentDescription(
        "Shows an overview of the user's pension strength based on salary and retirement period."
    )

    @Parameter(title: "User ID", requestValueDialog: "Which user should we check?")
    var userId: String?

    func perform() async throws -> some IntentResult & ReturnsValue<PensionScoreResult> {
        let score = await PensionService.shared.getPensionScore(userId: userId)
        return .result(value: PensionScoreResult(
            score: score.value,
            explanation: score.explanation
        ))
    }
}

struct PensionScoreResult: IntentResultValue {
    var localizedDescription: String { "\(score)/100 — \(explanation)" }
    let score: Int
    let explanation: String
}
```

### Example 2: AppIntent for "Fetch external pension data"

```swift
struct FetchPensionsInfoIntent: AppIntent {
    static var title: LocalizedStringResource = "Fetch PensionsInfo"
    static var description = IntentDescription(
        "Fetches pension data from external providers to create a consolidated pension overview."
    )

    @Parameter(title: "Provider")
    var provider: String  // e.g. "ATP"

    func perform() async throws -> some IntentResult & ReturnsValue<PensionsInfoResult> {
        let data = await PensionsInfoService.shared.fetch(from: provider)
        return .result(value: PensionsInfoResult(
            provider: provider,
            totalSavings: data.totalAmount,
            currency: data.currency
        ))
    }
}

struct PensionsInfoResult: IntentResultValue {
    var localizedDescription: String { "\(provider): \(totalSavings) \(currency)" }
    let provider: String
    let totalSavings: Double
    let currency: String
}
```

### Registering intents for MCP discovery

```swift
// In your @main App struct or AppDelegate
import AppIntents

// Donate the intents so the system (and MCP-compatible agents) can discover them
AppIntentDonationManager.shared.donate(intents: [
    CheckPensionScoreIntent.self,
    FetchPensionsInfoIntent.self
])
```

When iOS 26.1 MCP support is stable, agents will discover these intents automatically through the system — no extra code needed.

---

## 3. mobile-mcp — Open-source bridge (works today on all devices)

With mobile-mcp you don't write app code. You run an MCP server that can automate *any* app (including apps you don't own) via accessibility APIs or screen coordinates. The AI agent uses the built-in tools to navigate the app as a human would.

### Setup: MCP server configuration

Add the following to your MCP client config (e.g. Claude Desktop, Cursor, GitHub Copilot):

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

Start it manually with:

```bash
npx @mobilenext/mobile-mcp@latest
# The tool auto-detects connected devices, simulators, and emulators
```

### Example agent tool calls

The agent sends these tool calls automatically based on natural-language instructions. They illustrate how the LLM sitemap JSON maps to real actions:

```json
[
  {
    "tool": "mobile_launch_app",
    "parameters": {
      "packageName": "dk.example.myapp"
    }
  },
  {
    "tool": "mobile_list_elements_on_screen",
    "parameters": {}
  },
  {
    "tool": "mobile_click_on_screen_at_coordinates",
    "parameters": {
      "x": 320,
      "y": 640
    }
  },
  {
    "tool": "mobile_type_text",
    "parameters": {
      "text": "Check pension score"
    }
  }
]
```

With the LLM sitemap as context, the agent can reason: *"The sitemap says there is a 'Check pension score' button on the home screen — find it and tap it"* — rather than having to discover the UI from scratch every time.

---

## How the pieces fit together

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
Together they make any website or app fully agent-ready.
