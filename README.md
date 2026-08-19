<div align="center">

# NOVA for Windows

**Neural Operative Virtual Assistant**

An AI assistant with hands. It sees your screen, controls your mouse and
keyboard, manages your files and drives your browser — by voice or by text.

</div>

---

This repository is the **Windows** build — an `.msi` and an NSIS installer. Other platforms live in their own repositories, each stripped to
one target.

## Install

Run the installer from a release, or build it yourself:

```powershell
git clone https://github.com/Ace-Haxxed/Nova-Windows
cd Nova-Windows
powershell -ExecutionPolicy Bypass -File scripts\install-windows.ps1
npm install
npm run desktop:build
```

The script installs system dependencies. The build writes an `.msi` and an NSIS
`.exe` to `src-tauri\target\release\bundle\`.

Needs **Windows 10** or newer.

```bash
nova              # start it
nova --keys       # open straight to the API key settings
nova --demo       # run three real prompts through the real agent loop
nova --reset      # forget every stored key
nova --version
```

---

## Give it a model

NOVA needs a language model. Pick one in **Settings → Keys**; you can switch
whenever you like.

| Backend | Key | Notes |
|---|---|---|
| **Built-in** | no | Runs inside NOVA. No server, no account, no internet after a one-time model download. |
| **Ollama** | no | Runs locally in its own server. Nothing leaves the machine. |
| **Groq** | free | Fastest cloud option by a wide margin. |
| **OpenRouter** | free tier | One key, every model. The `:free` models need no credits. |
| **NVIDIA** | free tier | Free credits, no card. Key starts `nvapi-`. |
| **Bytez** | yes | Serverless HuggingFace models. Type any model id. |
| **OpenAI** · **Anthropic** · **Gemini** | yes | The usual. |
| **Custom** | optional | Any OpenAI-compatible endpoint: vLLM, LM Studio, llama.cpp. |

Get a key from `openrouter.ai/keys`, `console.groq.com/keys`,
`build.nvidia.com` or your provider's console, then paste it in. Pasting a key
validates it against the live API in the same gesture — a green tick means the
key really worked, not that it looked plausible.

**OpenRouter models are read live.** NOVA fetches the catalogue, keeps the free
models that support tool calling, and picks the largest context window. No model
id is hardcoded anywhere, because every id that ever was hardcoded got withdrawn.

### Fully offline

Install [Ollama](https://ollama.com) for Windows, then:

```powershell
ollama pull llama3.1:8b
ollama pull llava
```

Then choose **Ollama**. No key, no account, no network.

---

## What it can do

See and describe your screen · find things on screen by description · click,
type, drag and scroll · manage windows and applications · read, write, organise,
zip and search files · drive a browser · run Python, Node and shell scripts ·
control volume, brightness and power · monitor processes · web search · timers
and reminders · long-term memory of your preferences

---

## What it does without asking

**Everything.** NOVA runs each action the model decides on immediately. There is
no confirmation dialog and no per-capability switch — both existed once and both
made it worse: a disabled tool looked to the model like a broken one, so it
looped trying to find another way.

What you get instead:

- **Every action is logged** as it happens, with arguments, result and timing.
  The log exports as JSON.
- **Deletes go to the trash**, never a hard delete. The log offers one-click
  restore.
- **Refusals are real.** Anything NOVA reports as denied came from the operating
  system, not from a prompt we added.

It can delete files and run shell commands. Read what it is doing.


---

## Privacy

No analytics, no telemetry, no crash reporting, no phoning home. The only
traffic NOVA makes is to the model backend you chose and to pages you ask it to
read. Choose Ollama and there is none.

API keys live in `%APPDATA%\\nova\\keys.json`, owner-readable only. That is
weaker than a system keychain — anything running as you can read it — and it is
deliberate: it keeps the keyring daemon off the startup path, so your first
message never waits on it. Keys are never written to the settings file or the
database, so exporting your action log cannot leak them.

Conversation history and memory live in a local SQLite database. Both can be
switched off or wiped from **Settings → Privacy**.

---

## Build from source

**Needs:** Node 18+, Rust 1.82+, and the system dependencies the install script
handles.

```bash
npm install
npm run desktop:dev      # development
npm run desktop:build    # build the installer
```

Before opening a pull request:

```bash
npm run typecheck
cd src-tauri && cargo clippy --all-targets -- -D warnings && cargo fmt --check
cd src-tauri && cargo test --lib
```

---

## How it fits together

```
src/
  core/          agent loop, LLM clients, memory, tools
  platform/      index.ts picks the tool set at startup
  components/    shared/ · desktop/ · Settings/ · onboarding/ · ui/
  hooks/         useAgent, useVoice, useHotkeys, useWakeWord
  store/         zustand: conversation, settings, keys, actions
src-tauri/
  src/commands/  screen, mouse, keyboard, windows, files, apps, system,
                 browser, voice, wakeword, keys, db
  src/platform/  detect.rs, windows.rs
```

**The agent loop** (`src/core/agent.ts`) is think → act → observe → repeat. The
model streams a reply, tool calls run, results feed back, and it continues until
it answers without calling a tool.

**Input goes through `enigo`** (SendInput), and window management and system
control through PowerShell helpers rather than FFI — which keeps the crate free
of hand-written `unsafe`.

**Hardware never goes through the webview.** The microphone is captured in Rust
with `cpal` (WASAPI), not `getUserMedia`. The frontend receives audio as
`mic-chunk` and `mic-level` events.

### Deliberate choices

- **CDP, not Playwright**, for browser control. Playwright is a Node library and
  cannot be driven from a Tauri binary without shipping a Node runtime. Firefox
  removed its CDP implementation in version 129, so there `open_url` works and
  the DOM tools say what is missing rather than failing silently.
- **Subprocess helpers, not FFI**, for capture and window management. No
  hand-written `unsafe`.
- **Models are downloaded, not bundled.** 140 MB in every installer for a
  feature most people replace with a cloud key is a bad trade.

### Known limits

- **SmartScreen will warn on first run.** The installer is unsigned; code
  signing needs a certificate that cannot live in a public repository.
- **Some system control needs an elevated prompt**, which Windows shows itself.
- **The wake word must be trained before it works.** Record a sample in
  **Settings → Voice**. Until then it listens for nothing and says nothing.

---

NOVA can control your device. Read what it asks before you approve it.

---

## Other platforms

| Platform | Repository | Install |
|---|---|---|
| **Android** | [Nova-Android](https://github.com/Ace-Haxxed/Nova-Android) | Open [Releases](https://github.com/Ace-Haxxed/Nova-Android/releases/latest) on the phone and tap the APK |
| **iOS** | [Nova-Ios](https://github.com/Ace-Haxxed/Nova-Ios) | Xcode with a free Apple ID (7-day cert) |
| **Arch Linux** | [Nova](https://github.com/Ace-Haxxed/Nova) | `scripts/install-arch.sh`, then `scripts/install.sh` |
| **Debian / Ubuntu** | [Nova-Debian](https://github.com/Ace-Haxxed/Nova-Debian) | `scripts/install-debian.sh`, then `scripts/install.sh` |
| **Fedora** | [Nova-Fedora](https://github.com/Ace-Haxxed/Nova-Fedora) | `scripts/install-fedora.sh`, then `scripts/install.sh` |
| **macOS** | [Nova-Mac](https://github.com/Ace-Haxxed/Nova-Macos) | `scripts/install-mac.sh`, then build the `.dmg` |
