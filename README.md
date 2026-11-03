# PhoneAsWebcam

Turn an iPhone into a high-quality webcam for a Windows 11 PC. It uses the full capability of the selected iPhone camera, with no artificial caps on resolution or frame rate.

- **iPhone app** (Swift / SwiftUI): captures and hardware-encodes video (HEVC or H.264) and streams it over USB, or over Wi-Fi as a fallback.
- **Windows desktop app** (C# / .NET 8 / WinUI 3): connects to the phone, decodes the stream, and provides preview and settings.
- **Windows virtual camera** (C++ / Media Foundation): a system-wide camera that Teams, Zoom, OBS, browsers and the Windows Camera app can select.

Status: early setup. No app code yet.

## Layout

| Folder | Contents |
|---|---|
| `ios/` | Xcode project (Swift / SwiftUI) |
| `windows/App/` | C# WinUI 3 desktop app |
| `windows/VirtualCamera/` | C++ Media Foundation media source DLL |
| `protocol/` | Shared wire-protocol spec (framing, handshake, formats) |
| `docs/` | Project documentation |

## Building

- iOS app: a Mac with Xcode.
- Windows apps: a Windows 11 PC with Visual Studio. The virtual camera needs Windows 11 (`MFCreateVirtualCamera`).
- USB connection on Windows: needs the Apple Devices app or iTunes installed.

For full requirements, decisions and scope, see [docs/PROJECT_CONTEXT.md](docs/PROJECT_CONTEXT.md).
