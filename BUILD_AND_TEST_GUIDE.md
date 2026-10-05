# Building and Testing the Game

This guide explains how to compile the Unity project, create development builds, test multiplayer locally, and distribute test builds to other computers or mobile devices. The simplest reliable multiplayer test is one host and one client running the same desktop build and connecting through Unity Relay with a join code.

The project targets Unity `6000.3.5f2`. Install that editor version, or a compatible newer Unity 6 editor, through Unity Hub. When installing it, include the build-support module for every platform you intend to target.

## Important project-specific facts

- The product name is `U6-Multiplayer` and the current version is `0.1.0`.
- `Assets/Scenes/Multiplayer.unity` is the intended multiplayer scene.
- `Assets/Scenes/Singleplayer.unity` is the intended offline-style test scene.
- Multiplayer over the UI uses Unity Relay, including tests between two programs on the same computer.
- The local-host mode created when the multiplayer scene opens is for solo use. The UI does not expose direct-IP or LAN joining.
- Relay and Vivox require the project to be linked to a Unity Cloud project with those services available.
- The project supports at most four connected players because `RelayManager` and the spawn-point setup are designed for four.

## Fix the scene list before the first build

The repository's current build-scene configuration contains these entries:

1. `Assets/Scenes/Multiplayer.unity`
2. `Assets/Scenes/Singleplayer.unity`
3. `Assets/Samples/NGO_Minimal_Setup/NGO_Setup.unity`
4. `Assets/Scenes/New Multiplayer.unity`

The fourth path does not exist and can cause a build error. The NGO sample scene is present, but it is not part of the actual game.

In Unity 6:

1. Open **File > Build Profiles**.
2. Open the scene-list section for the active profile.
3. Remove `Assets/Scenes/New Multiplayer.unity`.
4. Remove or disable `Assets/Samples/NGO_Minimal_Setup/NGO_Setup.unity` unless you deliberately want the package sample in the build.
5. Keep `Assets/Scenes/Multiplayer.unity` first so it becomes the startup scene.
6. Keep `Assets/Scenes/Singleplayer.unity` only if the final game will provide a way to load it. Otherwise create a separate Singleplayer build profile.

Unity 6 uses Build Profiles to keep platform and build settings together. Unity's official workflow is documented in [Create a build profile](https://docs.unity3d.com/6000.0/Documentation/Manual/create-build-profile.html).

## Link your own Unity Cloud project

The repository contains another developer's Unity organization and cloud project identifiers. A clone does not give you permission to use that project.

Before testing Relay or Vivox:

1. Sign in to Unity Hub and open the project.
2. In Unity, open **Edit > Project Settings > Services**.
3. Unlink the inaccessible project if necessary, then link this Unity project to a Cloud project that you own.
4. In the Unity Dashboard, enable or configure Relay and Vivox for the same project and environment.
5. Return to Unity and wait for service configuration and package imports to finish.
6. Test anonymous authentication, hosting, joining, and voice in the Editor before distributing a build.

Every host and client build must contain configuration for the same Unity Cloud project and environment. The host creates a Relay allocation and join code; clients use that code to join the allocation. See Unity's [Relay allocation and joining overview](https://docs.unity.com/relay/allocating-binding-joining).

Do not commit personal service credentials, signing passwords, keystores, or provisioning profiles. Unity project identifiers are not secret credentials, but they should still point to the intended test or production environment.

## How C# compilation works in Unity

You normally do not compile this project by building a `.csproj` in Visual Studio or Rider. Unity owns the compilation pipeline.

1. Open the project in Unity.
2. Unity imports assets and compiles scripts automatically.
3. Open **Window > General > Console**.
4. Resolve every red compiler error before entering Play Mode or building.
5. Save modified scenes and assets.

The IDE-generated solution is useful for editing, navigation, and static analysis, but Unity remains the source of truth for assemblies, serialized references, scenes, packages, and the final player build.

When Unity creates a player build, it compiles scripts for the selected platform, processes assets and shaders, and packages everything into that platform's executable format. A successful script reload in the Editor is necessary but does not replace testing a real build.

## First test the game in the Editor

### Single-player smoke test

1. Open `Assets/Scenes/Singleplayer.unity`.
2. Enter Play Mode.
3. Click the start button.
4. Verify movement, camera look, sprint, jump, ground slam, item pickup, rotation, and drop.
5. Check the Console for exceptions and missing references.

This isolates movement and interaction problems from Relay, Netcode, authentication, and Vivox.

### Multiplayer smoke test

1. Open `Assets/Scenes/Multiplayer.unity`.
2. Enter Play Mode.
3. Verify that the local player spawns and the menu works.
4. Click **Host**. The project shuts down its temporary local host and starts a Relay host.
5. Confirm that a join code appears.
6. Click **Start** and verify that the host enters gameplay.

This proves only the host path. A real multiplayer test requires at least one separate client process.

## Local multiplayer testing methods

### Method 1: Editor host plus one desktop build

This is the recommended everyday workflow because it is simple and includes a real standalone player.

1. Create a Development Build for your desktop operating system.
2. Open `Multiplayer.unity` in the Editor and enter Play Mode.
3. In the Editor instance, click **Host** and copy its Relay join code.
4. Launch the standalone build.
5. In the build, enter the code and click **Join**.
6. Start the match from the host.
7. Test movement, spawning, colors, item authority, leaving, reconnecting, transport failure, and voice.

You can reverse the roles and host from the build while the Editor joins. Test both directions because lifecycle and timing bugs can be role-dependent.

### Method 2: Two or more standalone builds

This is closer to the player's real environment.

1. Build once.
2. Launch the executable multiple times. The project currently allows multiple instances because the single-instance setting is disabled.
3. Make one instance a Relay host.
4. Join from the other instances using the displayed code.
5. Arrange the windows side by side and test all supported player slots.

The project uses an anonymous Unity Authentication account. Multiple processes should receive separate player sessions, while Unity's Editor code also switches authentication profiles by editor process ID.

Use headphones when testing Vivox on one computer. Otherwise each instance can capture the other instance's audio and create feedback. Muting other local test participants is also useful when testing gameplay rather than voice.

### Method 3: Multiplayer Play Mode virtual players

The repository includes Multiplayer Play Mode `2.0.2`. It can run additional player instances from one project without creating a new build for every code change.

1. Open **Window > Multiplayer Play Mode**.
2. Activate one or more additional players before entering Play Mode.
3. Open `Multiplayer.unity` and press Play.
4. Host in one player window and join from another using the Relay code.
5. Watch the Console for each instance.

Unity documents virtual-player activation under [Activate a Virtual Player](https://docs-multiplayer.unity3d.com/mppm/current/virtual-players/virtual-players-enable/). Multiplayer Play Mode is best for small local tests; use standalone builds and physical devices for final validation.

If service login or voice behaves differently in virtual players, fall back to an Editor-plus-build test. External SDKs sometimes persist state differently from ordinary gameplay assets.

## Create a Windows development build

Windows is the easiest initial distribution target for this repository.

1. In Unity Hub, confirm that the editor installation includes **Windows Build Support**.
2. In Unity, open **File > Build Profiles**.
3. Choose or add a Windows profile and make it active.
4. Clean up the scene list as described earlier.
5. Enable **Development Build**.
6. Enable **Script Debugging** only when you need to attach an IDE debugger.
7. Optionally enable **Autoconnect Profiler** when investigating performance on your own test machine.
8. Select **Build** or **Build and Run**.
9. Choose a separate output directory such as `Builds/Windows-Development`.

Keep generated builds outside `Assets`. The repository should normally ignore `Builds/` so large generated binaries are not committed.

A Windows build is a folder, not only an `.exe`. Distribute the entire folder, including:

- `U6-Multiplayer.exe`
- `U6-Multiplayer_Data`
- `UnityPlayer.dll`
- Any other files and directories Unity generates beside the executable

Zip the complete folder before sending it. If you send only the `.exe`, the game will not run.

## Development versus release builds

Use a Development Build while implementing and testing because it provides better logging and profiling support. Disable development-only options for a release candidate.

| Build type | Use it for | Recommended options |
| --- | --- | --- |
| Development | Daily multiplayer tests and bug reports | Development Build on; profiler/debugging only when needed |
| Release candidate | Performance, compatibility, and distribution rehearsal | Development Build off; final scene list and player settings |
| Store release | Public distribution | Platform signing, final identifiers, icons, versioning, privacy text, and store packaging |

Do not judge final frame rate or loading time from a deeply profiled development build. Instrumentation changes performance.

## Send a desktop build to another computer

### Same operating system

1. Zip the entire build folder.
2. Upload it to a file-sharing service or transfer it over your local network or USB storage.
3. On the other computer, extract the archive to a normal writable directory.
4. Launch the executable from the extracted folder.
5. Allow the game through the operating system firewall if prompted.
6. Grant microphone access when testing Vivox.
7. On the host device, click **Host** and send the join code to the tester.
8. On the client device, enter the code and click **Join**.
9. Keep the host running for the whole session; this project does not implement host migration.

Relay allows the players to be on different networks, so router port forwarding is not required. Both devices need internet access to Unity Authentication, Relay, and Vivox services.

### Different desktop operating systems

Build a separate player for each operating system. A Windows `.exe` does not run natively on macOS, and a macOS `.app` is not a Windows executable.

- Build Windows and Linux players from an editor installation with the appropriate platform modules.
- Build and sign macOS applications on macOS when preparing realistic distribution.
- Compress a macOS `.app` before transfer so its application-bundle structure and permissions are preserved.
- Unsigned or unnotarized development builds may trigger Windows SmartScreen or macOS Gatekeeper. For a small trusted test, the tester can explicitly approve the application; public distribution should use proper code signing and notarization.

All testers should use builds from the same commit and compatible content version. Netcode prefab or scene mismatches can create connection failures or incorrect synchronized behavior even when the application launches.

## Build and test on Android

### Current limitation

The project has keyboard, mouse, and gamepad bindings, but it does not include an on-screen mobile movement and camera UI. An Android build may launch, yet it will not be comfortably playable on a touchscreen. For meaningful Android tests, first add touch controls or connect a supported physical gamepad.

### Setup and build

1. In Unity Hub, add **Android Build Support**, **Android SDK and NDK Tools**, and **OpenJDK** to Unity `6000.3.5f2`.
2. Enable Developer Options and USB debugging on the Android device.
3. Connect the device and approve its debugging prompt.
4. In Unity, open **File > Build Profiles** and add an Android profile.
5. Switch to the Android profile and wait for asset reimport.
6. Set a unique package identifier in **Edit > Project Settings > Player**. Replace the template identifier with something you control, such as `com.yourstudio.u6multiplayer`.
7. Add appropriate microphone permission/purpose configuration for Vivox.
8. For direct testing, create an APK rather than a Google Play App Bundle.
9. Select the connected device under **Run Device**.
10. Choose **Build and Run**.

Unity can install and launch an APK directly on the selected device. The official workflow is described in [Build your application for Android](https://docs.unity3d.com/6000.0/Documentation/Manual/android-BuildProcess.html).

For another tester, send the APK through a trusted file-transfer method. The tester must allow installation from that source. Use debug signing for private development tests; use your own protected keystore for release builds. Never commit the keystore or its passwords.

When debugging, inspect device logs with Android Logcat. Unity documents USB and wireless device logging in [Debugging on an Android device](https://docs.unity3d.com/6000.0/Documentation/Manual/android-debugging-on-an-android-device.html).

## Build and test on iPhone or iPad

### Current limitation

As on Android, the project has no touch movement or look controls. Add mobile controls or use a supported physical controller. The current microphone usage description is blank, so configure a clear user-facing reason before testing Vivox.

### Required workflow

1. Install the iOS Build Support module for the Unity editor.
2. Use a Mac with Xcode for the final native build and device installation.
3. Create an iOS Build Profile and switch to it.
4. Set a unique Bundle Identifier, version, build number, microphone purpose text, and signing team.
5. Build from Unity to generate an Xcode project.
6. Open the generated project in Xcode.
7. Select the signing team and connected device.
8. Build and run from Xcode.

Unity generates the Xcode project; Xcode compiles, signs, installs, and launches the iOS application. See [Build an iOS application](https://docs.unity3d.com/6000.0/Documentation/Manual/iphone-BuildProcess.html).

For remote testers, distribute through TestFlight or an appropriately provisioned ad hoc/development build. Sending an unsigned application folder is not sufficient on iOS.

## Recommended cross-device multiplayer test

For the first test with another person, use two Windows computers:

1. Link the project to your Unity Cloud project and verify Relay in the Editor.
2. Fix the build scene list.
3. Make a Windows Development Build.
4. Zip and send the complete build folder.
5. Both people extract and run the same build.
6. Player A clicks **Host** and sends the displayed code to Player B.
7. Player B enters the code and clicks **Join**.
8. Player A clicks **Start**.
9. Test movement and items first, then enable microphones and test positional voice.
10. Swap roles so the previous client becomes host.

Record the build's Git commit, platform, Unity version, device specifications, and reproduction steps with every bug report.

## What to test in every multiplayer build

### Session flow

- Start as a local solo host.
- Switch to Relay hosting and receive a join code.
- Join from another instance and reject invalid codes cleanly.
- Join up to the four-player limit.
- Leave and return to the lobby.
- Disconnect the host and observe client behavior.
- Temporarily interrupt a client's network and verify recovery or error handling.

### Gameplay and synchronization

- Each player spawns once at a distinct spawn point.
- Local input controls only the owning player.
- Remote movement is visible and reasonably smooth.
- Player colors agree on every device.
- Only one player can hold an item at a time.
- Pickup, rotation, collision avoidance, drop, and throw agree on all clients.
- The game-start transition and HUD happen for every player.
- Late joins and reconnects behave as intended.

### Voice

- Each player logs into Vivox and joins the same positional channel.
- Speech indicators appear on remote players.
- Volume changes with distance.
- Local mute works.
- Leaving a session leaves the voice channel.
- Denying microphone permission fails gracefully.

### Device and build quality

- Correct resolution, fullscreen/window behavior, and input devices.
- Acceptable CPU, GPU, memory, and network usage.
- No recurring warnings or exceptions in player logs.
- Firewall, microphone, and mobile permission prompts are understandable.
- The build starts from a clean installation without Editor-generated local state.

## Find player logs

When a standalone build fails, ask the tester to close the game and send the player log along with exact reproduction steps.

Typical locations for this project's current `DefaultCompany/U6-Multiplayer` identity are:

- Windows: `%USERPROFILE%\AppData\LocalLow\DefaultCompany\U6-Multiplayer\Player.log`
- macOS: `~/Library/Logs/DefaultCompany/U6-Multiplayer/Player.log`
- Linux: `~/.config/unity3d/DefaultCompany/U6-Multiplayer/Player.log`
- Android: use Android Logcat while reproducing the problem
- iOS: use the Xcode device console

Change the placeholder company name and package identifiers before broader distribution; doing so also changes some log and save-data paths.

## Common failures

| Symptom | Likely cause | What to check |
| --- | --- | --- |
| Build reports a missing scene | Stale `New Multiplayer.unity` entry | Clean the Build Profiles scene list |
| Build opens the wrong scene | Wrong first enabled scene | Put `Multiplayer.unity` first |
| Host or Join fails immediately | Cloud project unavailable, services disabled, or no internet | Services link, Dashboard environment, Authentication, Relay, and player log |
| Players cannot join by local IP | Direct-IP UI is not implemented | Use Relay, or add a separate LAN/direct-IP connection flow |
| Voice does not work | Vivox setup, microphone permission, device selection, or channel join failed | Dashboard, OS permissions, logs, and headphones |
| Second local instance behaves strangely | Shared state, focus, audio feedback, or insufficient resources | Separate processes/profiles, window focus, mute, and system usage |
| Android/iOS player cannot move | No touch controller exists | Connect a gamepad or implement touch input and UI |
| Sent Windows build will not launch | Only the `.exe` was sent | Zip and send the complete generated folder |
| Cross-platform client cannot connect correctly | Different game versions or network prefab layouts | Rebuild every platform from the same commit |

## Before sending a test build

- [ ] Unity Console has no compiler errors.
- [ ] `Multiplayer.unity` is the first enabled scene.
- [ ] Missing and sample scenes are removed from the profile.
- [ ] The project is linked to a Cloud project you control.
- [ ] Relay host and join work locally.
- [ ] The version number or build label identifies this build.
- [ ] Desktop builds include the entire generated folder.
- [ ] Mobile identifiers, permissions, and signing are configured.
- [ ] The tester receives controls, installation steps, join instructions, and the player-log location.
- [ ] Everyone tests the same commit and build version.

## Suggested next improvement

Create two committed Build Profiles:

- `Desktop Development`: multiplayer scene first, Development Build enabled, intended for Windows/macOS/Linux testing.
- `Desktop Release`: the same game scenes, development instrumentation disabled, and final player settings.

Later, add separate Android and iOS profiles after implementing touch controls and mobile permission handling. This prevents platform-specific settings from being changed manually before every build and makes test results easier to reproduce.
