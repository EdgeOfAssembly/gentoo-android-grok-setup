# Android Development Skill Template for Grok Build

This is a reusable skill / AGENTS.md style template for Grok Build agents working on Android projects on a Gentoo + X11 host with the CLI Android SDK installed.

Place this (or adapt it) as `AGENTS.md`, a skill file, or project-level instructions so agents automatically follow the conventions.

---

## Skill: Android App Development (Gentoo CLI + Local MCP)

**When to use:** Any request involving creating, modifying, building, testing, or debugging Android applications.

### Preferred Stack
- Language: Kotlin
- UI: Jetpack Compose + Material 3
- Architecture: recommended MVVM or simple unidirectional data flow
- Persistence: Room (or DataStore for simple prefs)
- Networking: Ktor or Retrofit + OkHttp (when needed)
- DI: optional Hilt or manual
- Min SDK: 26+ (or higher as required)
- Target / Compile SDK: 36 (Android 16) when possible; also support 34/35 for broader testing
- Build system: Gradle Kotlin DSL (`build.gradle.kts`)

### Environment Assumptions
- `ANDROID_HOME` and PATH already contain cmdline-tools, platform-tools, emulator, build-tools.
- Emulator AVDs exist (e.g. `pixel_api36`).
- Local MCP servers for ADB / device interaction may be available (screenshots, UI tree, taps).
- Host is Gentoo x86_64 running X11; emulator windows appear under X11.

### Autonomous Workflow
1. **Scaffold**  
   - Create project directory.  
   - Generate `settings.gradle.kts`, root + app `build.gradle.kts`, `AndroidManifest.xml`, `MainActivity.kt`, Compose theme, etc.  
   - Prefer modern templates (Compose Empty Activity style).

2. **Implement**  
   - Write clean, idiomatic Kotlin.  
   - Keep UI declarative with Compose.  
   - Handle configuration changes, dark theme, basic accessibility.

3. **Build**  
   ```bash
   ./gradlew assembleDebug          # or bundleRelease for AAB
   ```
   Fix any compilation / resource errors immediately.

4. **Test on Emulator**  
   - Start AVD if not running: `emulator -avd pixel_api36 &` (or use existing).  
   - Wait for boot: `adb wait-for-device`.  
   - Install: `adb install -r app/build/outputs/apk/debug/app-debug.apk`.  
   - Launch: `adb shell am start -n <package>/.MainActivity`.  
   - Capture logs: `adb logcat -d` or filtered.  
   - Use local MCP tools for screenshots / UI hierarchy / gestures when available.

5. **Iterate**  
   - Reproduce reported issues.  
   - Fix, rebuild, reinstall, re-test in a tight loop.  
   - Prefer small, verifiable changes.

6. **Release packaging (when requested)**  
   - Generate keystore with `keytool` if needed.  
   - Configure signing in Gradle.  
   - Produce signed APK/AAB with `apksigner` / Gradle tasks.  
   - Run `zipalign`.

### Conventions & Best Practices
- Always use the Gradle wrapper (`./gradlew`).  
- Never hard-code absolute paths outside `$ANDROID_HOME`.  
- Prefer `google_apis` system images; use Play Store images only when Google services are required.  
- Keep projects offline-first when possible.  
- Write meaningful commit messages if using git.  
- Document any new environment variables or one-time setup in the project README.  
- When using MCP device tools, prefer accessibility-tree based interactions over raw coordinates when available.

### Common Pitfalls on this Setup
- Emulator GL / libstdc++ conflicts → move bundled libs aside.  
- Missing licenses → `sdkmanager --licenses`.  
- AVD not found → recreate with `avdmanager`.  
- Group permissions → ensure user is in `android` and `kvm`.  
- X11 display → emulator expects a running X session.

### Example Agent Prompt Patterns
- “Create a minimal Compose calculator app, build it, run it on the pixel_api36 emulator, and take a screenshot.”  
- “Add offline Room storage to the existing todo app and verify persistence after force-stop.”  
- “Fix the crash on rotation and produce a signed release APK.”

### Success Criteria
- Compiles cleanly.  
- Installs and launches on emulator (or physical device).  
- Core user flows work.  
- Logs show no fatal errors for the tested paths.  
- Code is maintainable and follows the preferred stack.

---

**End of skill template.**  
Agents should load this context at the start of any Android-related task and follow the workflow strictly for maximum autonomy.
