# Sentinel AI Automation

**Version:** 1.0.2

An AI-powered browser automation extension using the Sentinel library via a Node.js bridge. Describe actions in plain English and let AI figure out selectors, clicks, and data extraction.

## Permissions

| Permission | Why |
|---|---|
| `network` | Talks to your LLM provider and loads the target page |
| `shell` | Runs the Sentinel bridge script as a child process |
| `secrets` | Stores your LLM API key in the OS keychain |
| `desktopInput` | Needs exclusive mouse/keyboard focus while the browser is driven |

## Features

- **Natural Language Automation**: Describe browser actions in plain English
- **AI-Powered Selectors**: Sentinel automatically figures out the right selectors
- **Multi-LLM Support**: Works with OpenAI, Anthropic, Gemini, and Ollama
- **Self-Healing**: Automatically adapts to page changes
- **Token Efficient**: Uses 10× fewer LLM tokens than alternatives
- **Browser Automation**: Built on Playwright for reliable browser control

## Installation

1. Sidebar → **Extensions** → **Install from file...** → pick `sentinel-ai-automation.dsext`
2. Requires Node.js/npm on PATH so npm packages can install
3. Configure an LLM provider in the extension settings (see below)

No example workflow ships with this extension — every path needs an LLM API key (or a local Ollama setup) plus a browser install before it can run.

### LLM Provider Setup

You need to configure an LLM provider in the extension settings:

1. **OpenAI**: Get an API key from https://platform.openai.com/api-keys
2. **Anthropic**: Get an API key from https://console.anthropic.com/
3. **Gemini**: Get an API key from https://aistudio.google.com/app/apikey
4. **Ollama**: Run Ollama locally (no API key needed)

Configure these in the extension settings:
- **LLM Provider**: Choose your provider (openai, anthropic, gemini, ollama)
- **LLM API Key**: Enter your API key (not needed for Ollama)
- **LLM Model**: Optional model name (e.g., gpt-4, claude-3-opus, gemini-pro)

## Usage

### AI Automate Node

Executes browser automation using natural language instructions.

**Parameters:**
- **url**: The URL to navigate to (required)
- **instruction**: Plain English description of what to do (required)
- **llmProvider** / **llmApiKey** / **llmModel**: Optional overrides of extension settings

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/sentinel-ai-automation
python tools/deskstride_ext_cli.py scan extensions/sentinel-ai-automation
python tools/deskstride_ext_cli.py pack extensions/sentinel-ai-automation -o sentinel-ai-automation.dsext
```
