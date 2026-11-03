# PhoneAsWebcam

Turn an iPhone into a high-quality webcam for a Windows 11 PC. It uses the full capability of the selected iPhone camera, with no artificial caps on resolution or frame rate.

- **iPhone app** (Swift / SwiftUI): captures and hardware-encodes video (HEVC or H.264) and streams it over USB, or over Wi-Fi as a fallback.
- **Windows desktop app** (C# / .NET 8 / WinUI 3): connects to the phone, decodes the stream, and provides preview and settings.
- **Windows virtual camera** (C++ / Media Foundation): a system-wide camera that Teams, Zoom, OBS, browsers and the Windows Camera app can select.

Status: early setup. No app code yet.

## Layout

```
PhoneAsWebcam/
├── iphone/                    # iPhone app (Swift / SwiftUI), built on a Mac with Xcode
│   ├── PhoneAsWebcam/
│   │   ├── App/               # entry point, lifecycle
│   │   ├── Camera/            # capture, lens + format discovery, focus/exposure/WB, zoom
│   │   ├── Encoder/           # VideoToolbox HEVC / H.264
│   │   ├── Streaming/         # Network.framework server, USB + Wi-Fi
│   │   ├── Protocol/          # Swift side of the wire protocol
│   │   ├── Device/            # thermal warnings, screen dimming
│   │   ├── UI/                # SwiftUI screens
│   │   └── Resources/         # Info.plist, assets, permission strings
│   └── PhoneAsWebcamTests/
├── windows/                   # Windows 11 side, built with Visual Studio (one .sln)
│   ├── App/                   # C# WinUI 3 desktop app
│   ├── Connection/            # C#: USB (usbmuxd) + Wi-Fi, C# side of the wire protocol
│   ├── Decoder/               # C#: Media Foundation / D3D11 hardware decode
│   ├── FrameBridge/           # C++ header + C#: shared-memory frames, app -> virtual camera
│   ├── VirtualCamera/         # C++ Media Foundation media source DLL
│   ├── Installer/             # MSIX packaging, virtual camera registration
│   └── Tests/
└── docs/
    ├── PROJECT_CONTEXT.md     # goals, decisions, scope
    └── protocol/              # wire-protocol spec shared by iphone/ and windows/
```

Each of `iphone/` and `windows/` has its own README describing its modules. The wire-protocol spec in `docs/protocol/` is the contract both sides implement.

## Building

- iOS app: a Mac with Xcode.
- Windows apps: a Windows 11 PC with Visual Studio. The virtual camera needs Windows 11 (`MFCreateVirtualCamera`).
- USB connection on Windows: needs the Apple Devices app or iTunes installed.

For full requirements, decisions and scope, see [docs/PROJECT_CONTEXT.md](docs/PROJECT_CONTEXT.md).
