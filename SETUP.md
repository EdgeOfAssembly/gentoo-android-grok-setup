# Gentoo Android Development Environment for Grok Build + Local MCP

**Target:** Gentoo Linux x86_64, X11 only (no Wayland), for autonomous Grok Build agents using local tools and optional local MCP servers compiled/run from source.

This setup allows Grok Build agents to create, build, test, debug, and iterate Android apps (Kotlin + Jetpack Compose recommended) from start to finish with minimal human intervention.

## 1. Prerequisites

- Sufficient disk space: 15–30 GB free (SDK + system images)
- RAM: preferably 16 GB+ system (emulator needs several GB free)
- Kernel with KVM support (`lsmod | grep kvm`, `/dev/kvm` exists)
- Internet access
- X11 running

```bash
# Groups
emerge --ask acct-group/android
usermod -aG android,kvm $USER
# Log out and back in after group changes
```

## 2. Java

```bash
emerge --ask dev-java/openjdk-bin   # or openjdk
emerge --ask app-eselect/eselect-java
eselect java-vm list
eselect java-vm set user openjdk-bin-17   # 17 recommended; 21 also fine
```

## 3. Core System Packages

```bash
emerge --ask dev-util/android-tools          # adb, fastboot (USE=udev recommended)
# Optional but useful:
emerge --ask app-mobilephone/scrcpy
emerge --ask app-emulation/qemu              # KVM acceleration for emulator
# Audio for emulator:
emerge --ask media-sound/pulseaudio          # or media-sound/apulse
```

Also ensure basic tools: `unzip`, `wget`/`curl`, `git`.

## 4. Android SDK (Command-Line Tools — preferred for autonomy)

```bash
mkdir -p ~/Android/Sdk
cd /tmp
wget https://dl.google.com/android/repository/commandlinetools-linux-15859902_latest.zip
unzip commandlinetools-linux-*.zip
mkdir -p ~/Android/Sdk/cmdline-tools
mv cmdline-tools ~/Android/Sdk/cmdline-tools/latest
```

**Permanent environment variables** (add to `~/.bashrc` or better `/etc/env.d/99android` then `env-update`):

```bash
export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_SDK_ROOT=$ANDROID_HOME
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
```

Source the profile or open a new terminal.

## 5. Install SDK Packages

```bash
sdkmanager --sdk_root=$ANDROID_HOME --list
yes | sdkmanager --sdk_root=$ANDROID_HOME --licenses

# Core packages (API 36 required for new Play Store apps from 31 Aug 2026)
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

# Optional native support
sdkmanager --sdk_root=$ANDROID_HOME "ndk;latest" "cmake;latest"
```

Adjust exact build-tools and NDK versions after running `--list`.

## 6. Create an AVD (Android Virtual Device)

```bash
avdmanager create avd -n pixel_api36 -k "system-images;android-36;google_apis;x86_64" -d pixel_6
# or pixel_7, etc.
emulator -list-avds
```

Test under X11:
```bash
emulator -avd pixel_api36 &
adb devices
```

**Gentoo-specific emulator tips:**
- If GL/`libstdc++` issues: move `$ANDROID_HOME/emulator/lib64/libstdc++/libstdc++.so*` aside so system libraries are used.
- Ensure user is in `kvm` group and `/dev/kvm` permissions are correct.

## 7. Optional Local MCP Servers (from source only)

Grok Build’s built-in shell + filesystem tools are already sufficient for full autonomy. Extra MCP servers add higher-level device control (screenshots, UI tree, taps, etc.).

All recommended servers run locally (stdio) and are built/run from source.

### Example: Node/TypeScript ones
```bash
emerge --ask net-libs/nodejs   # Node 18+/20+
mkdir -p ~/mcp-servers && cd ~/mcp-servers

# Option A: martingeidobler/android-mcp-server
git clone https://github.com/martingeidobler/android-mcp-server.git
cd android-mcp-server
npm install && npm run build
# Run: node dist/index.js

# Option B: mobile-next/mobile-mcp (richer features)
git clone https://github.com/mobile-next/mobile-mcp.git
cd mobile-mcp
npm install && npm run build
```

### Example: Python ones
```bash
# Ensure recent Python + uv
git clone https://github.com/minhalvp/android-mcp-server.git
cd android-mcp-server
uv sync
# Run via: uv run server.py
```

Configure in Grok Build (example for `~/.grok/config.toml` or equivalent MCP config):

```toml
[mcp.servers.android]
command = "node"
args = ["/home/YOURUSER/mcp-servers/android-mcp-server/dist/index.js"]
```

or for Python:
```toml
[mcp.servers.android]
command = "uv"
args = ["--directory", "/home/YOURUSER/mcp-servers/android-mcp-server", "run", "server.py"]
```

## 8. Verification

```bash
java -version
echo $ANDROID_HOME
sdkmanager --list_installed
adb version
emulator -list-avds
```

## 9. Using with Grok Build

1. Install Grok Build (`curl -fsSL https://x.ai/cli/install.sh | bash` or equivalent).
2. Open a terminal that has the Android environment variables sourced.
3. `cd` into a project directory (or let the agent create one).
4. Run `grok` (TUI) or `grok -p "..."` (headless).
5. Agents can now autonomously scaffold Gradle projects, write Compose UI, build with `./gradlew`, launch the emulator, `adb install`, read logcat, fix bugs, etc.

---

See also `ANDROID_DEV_SKILL.md` for a reusable skill/prompt template that teaches Grok Build agents the preferred Android workflow on this setup.
