# Windows

Windows 11 side: a desktop app that receives and decodes the phone's video, and a virtual camera that other apps (Teams, Zoom, OBS, browsers) can select.

Build on a Windows 11 PC with Visual Studio. All projects go in one solution, `windows/PhoneAsWebcam.sln`.

| Folder | Language | Responsibility |
|---|---|---|
| `App/` | C# (.NET 8, WinUI 3) | Desktop app: device list, preview, settings, latency readout, thermal warnings from the phone |
| `Connection/` | C# library | Finding and connecting to the phone over USB (Apple's usbmuxd service) and Wi-Fi; C# implementation of the wire protocol in `docs/protocol/` |
| `Decoder/` | C# library | Hardware HEVC / H.264 decoding with Media Foundation and Direct3D 11 |
| `FrameBridge/` | C++ header + C# library | Shared-memory frame buffer that passes decoded frames from the app to the virtual camera; one shared layout definition used by both sides |
| `VirtualCamera/` | C++ (C++/WinRT) | Media Foundation custom media source (COM DLL), registered with `MFCreateVirtualCamera`; advertises the phone's current formats |
| `Installer/` | MSIX packaging | Packaging, COM registration of the virtual camera, Store submission |
| `Tests/` | C# / C++ | Unit and integration tests |

Data flow: phone → `Connection` → `Decoder` → `FrameBridge` → `VirtualCamera` → Teams / Zoom / OBS.
