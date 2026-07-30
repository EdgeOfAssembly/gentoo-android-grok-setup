# gentoo-android-grok-setup

Step-by-step Gentoo Linux (x86_64, X11-only) Android development environment for autonomous **Grok Build** agents + local MCP servers.

## Files

- **[SETUP.md](SETUP.md)** – Complete installation guide (Java, SDK CLI, emulator, KVM, local MCP from source).
- **[ANDROID_DEV_SKILL.md](ANDROID_DEV_SKILL.md)** – Reusable skill / AGENTS.md template that teaches Grok Build agents the preferred stack, workflow, and conventions for Android apps on this setup.

## Download

- Repository: https://github.com/EdgeOfAssembly/gentoo-android-grok-setup
- Zip of main branch: https://github.com/EdgeOfAssembly/gentoo-android-grok-setup/archive/refs/heads/main.zip

Clone or download the zip, follow SETUP.md, then place the skill template where Grok Build can pick it up (project root AGENTS.md, skills directory, etc.).

Once configured, agents can go from natural-language description to a running APK on the emulator fully autonomously.
