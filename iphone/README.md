# iPhone app

Swift / SwiftUI app that captures from the selected camera, hardware-encodes the video and streams it to the Windows app over USB (primary) or Wi-Fi (fallback).

Build on a Mac with Xcode. Create the Xcode project in this folder (`iphone/PhoneAsWebcam.xcodeproj`). The source folders below already exist, so Xcode picks them up as groups.

| Folder | Responsibility |
|---|---|
| `PhoneAsWebcam/App/` | App entry point, app lifecycle, dependency wiring |
| `PhoneAsWebcam/Camera/` | Capture session, lens selection, format discovery (`AVCaptureDevice.formats`), focus / exposure / white-balance locks, zoom, Center Stage, portrait blur |
| `PhoneAsWebcam/Encoder/` | Hardware HEVC / H.264 encoding with VideoToolbox; bitrate scaled to the chosen format |
| `PhoneAsWebcam/Streaming/` | Network.framework server; USB (usbmuxd) and Wi-Fi listeners; adaptive bitrate on Wi-Fi |
| `PhoneAsWebcam/Protocol/` | Swift implementation of the wire protocol in `docs/protocol/` |
| `PhoneAsWebcam/Device/` | Thermal monitoring and warnings, screen dimming, keep-awake |
| `PhoneAsWebcam/UI/` | SwiftUI screens: camera picker, format picker, controls, status |
| `PhoneAsWebcam/Resources/` | `Info.plist`, asset catalog, permission strings |
| `PhoneAsWebcamTests/` | Unit tests |
