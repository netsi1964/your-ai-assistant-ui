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
