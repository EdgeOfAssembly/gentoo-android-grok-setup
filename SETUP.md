# Gentoo Android Development Environment Setup for Grok Build + Local MCP

**Target:** Gentoo Linux x86_64, X11 only (no Wayland), for autonomous Android app development using Grok Build CLI agents and local MCP servers.

**Date reference:** July 2026 (API 36 required for new Play Store apps from 31 Aug 2026).

This guide sets up a pure CLI Android SDK so Grok Build agents can create, build, test, debug, and iterate Android apps (Kotlin + Jetpack Compose preferred) from start to finish with no further human intervention beyond the initial prompt.

## 1. Prerequisites

- Kernel with KVM support (`CONFIG_KVM`, `CONFIG_KVM_INTEL` or `CONFIG_KVM_AMD`). Verify: `lsmod | grep kvm` and existence of `/dev/kvm`.
- Sufficient disk space: 15–30 GB free for SDK + system images.
- Sufficient RAM: Emulator prefers 4–8+ GB free.
- Internet access for downloads and Gradle dependencies.
- X11 session already running (emulator uses X11 toolkit by default).

```bash
# Add user to required groups (log out / log in after)
emerge --ask acct-group/android
usermod -aG android,kvm $USER
```

## 2. Java

```bash
emerge --ask dev-java/openjdk-bin   # or openjdk
emerge --ask app-eselect/eselect-java
eselect java-vm list
eselect java-vm set user openjdk-bin-17   # 17 recommended; 21 also works
```

## 3. Core System Packages

```bash
emerge --ask dev-util/android-tools          # adb, fastboot (add USE="udev" if needed)
# Optional but recommended:
emerge --ask app-mobilephone/scrcpy          # device/emulator mirroring under X11
emerge --ask app-emulation/qemu              # KVM support (relevant USE flags: kvm, X, ...)
# Helpers
emerge --ask app-arch/unzip net-misc/wget dev-vcs/git
# Optional: Android Studio GUI (agents do not require it)
# emerge --ask dev-util/android-studio
```

For audio in emulator: PulseAudio or `media-sound/apulse`.

## 4. Android SDK (CLI preferred)

```bash
mkdir -p ~/Android/Sdk
cd /tmp
wget https://dl.google.com/android/repository/commandlinetools-linux-15859902_latest.zip
unzip commandlinetools-linux-*.zip
mkdir -p ~/Android/Sdk/cmdline-tools
mv cmdline-tools ~/Android/Sdk/cmdline-tools/latest
```

**Environment variables** (add permanently to `~/.bashrc`, `~/.profile` or `/etc/env.d/99android`):

```bash
export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_SDK_ROOT=$ANDROID_HOME
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
# JAVA_HOME already handled by eselect
```

Source the file or open a new terminal.

## 5. Install SDK Components

```bash
sdkmanager --sdk_root=$ANDROID_HOME --list
yes | sdkmanager --sdk_root=$ANDROID_HOME --licenses

sdkmanager --sdk_root=$ANDROID_HOME \
  "cmdline-tools;latest" \
  "platform-tools" \
  "build-tools;36.0.0" \
  "platforms;android-36" \
  "platforms;android-35" \
  "platforms;android-34" \
  "emulator" \
  "system-images;android-36;google_apis;x86_64" \
  "system-images;android-35;google_apis;x86_64"

# Optional
sdkmanager --sdk_root=$ANDROID_HOME "ndk;latest" "cmake;latest"
```

(Adjust exact build-tools / system-image versions after checking `sdkmanager --list`. Prioritize API 36.)

## 6. Create and Test AVD

```bash
avdmanager create avd -n pixel_api36 \
  -k "system-images;android-36;google_apis;x86_64" \
  -d pixel_6

emulator -avd pixel_api36 &     # opens under X11
adb devices
```

**Gentoo / X11 notes:**
- If GL / libstdc++ errors occur, move the bundled libraries out of `$ANDROID_HOME/emulator/lib64/` (or `lib/`) so the system libraries are used.
- Ensure Mesa / graphics libraries are present.
- For headless testing: `emulator -avd NAME -no-window -no-audio ...`

## 7. Local MCP Servers (from source only)

Grok Build’s built-in shell + filesystem tools already provide full autonomy. Optional local MCP servers give higher-level device control.

**Runtimes:**
```bash
emerge --ask net-libs/nodejs          # for TypeScript servers
# Python is usually already present; ensure recent version
```

**Example 1 – martingeidobler/android-mcp-server (Node/TS):**
```bash
mkdir -p ~/mcp-servers
cd ~/mcp-servers
git clone https://github.com/martingeidobler/android-mcp-server.git
cd android-mcp-server
npm install
npm run build
# Run: node dist/index.js
```

**Example 2 – mobile-next/mobile-mcp (Node/TS, richer features):**
```bash
cd ~/mcp-servers
git clone https://github.com/mobile-next/mobile-mcp.git
cd mobile-mcp
npm install
npm run build
```

**Example 3 – Python ADB servers (minhalvp/android-mcp-server or CursorTouch/Android-MCP):**
```bash
cd ~/mcp-servers
git clone <repo-url>
cd <repo>
# Install uv if needed, then:
uv sync
# Run via: uv run server.py (or equivalent)
```

**Configure in Grok Build** (example `~/.grok/config.toml` or equivalent `.mcp.json` style):

```toml
[mcp.servers.android]
command = "node"
args = ["/home/YOURUSER/mcp-servers/android-mcp-server/dist/index.js"]
```

Or for Python:
```toml
command = "uv"
args = ["--directory", "/home/YOURUSER/mcp-servers/android-mcp-server", "run", "server.py"]
```

All servers must stay local (stdio preferred). No remote/cloud MCP.

## 8. Verification

```bash
java -version
echo $ANDROID_HOME
sdkmanager --list_installed
adb version
emulator -list-avds
```

## 9. Using with Grok Build

1. Install/start Grok Build in a shell that has the environment variables sourced.
2. Optionally add the local MCP servers as shown.
3. Prompt the agents, e.g.:
   > Create a modern offline-first todo app using Jetpack Compose, Material 3, Room, and Kotlin. Build the APK, start the emulator, install it, and verify basic functionality.

Agents will use shell commands for Gradle, adb, emulator, etc., and any configured local MCP for richer interaction.

## Notes & Tips

- Gradle wrapper is generated per project; no system Gradle required.
- Signing: debug keystore is automatic; release uses `keytool` + `apksigner`.
- Physical devices work once USB debugging is enabled and the android group + udev rules are active.
- Keep the SDK updated with `sdkmanager` as needed; agents can do this too.

For the skill template that teaches agents best practices on this setup, see `ANDROID_DEV_SKILL.md`.
