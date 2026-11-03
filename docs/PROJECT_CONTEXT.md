# PhoneAsWebcam — Project Context

## 1. Goal
Build an app that turns an **iPhone into a high-quality webcam for a Windows 11 PC**.
Quality is the top priority: the app must use **whatever the phone's selected camera supports**, with no artificial caps.

The project began as a broader idea for full iOS ↔ Windows integration. It was narrowed to **webcam only** (see section 8).

## 2. Repository and workflow
- **Repo:** `sweetysharan29-bit/PhoneAsWebcam` (GitHub), default branch `main`
- **Workflow rule:** commit and push **directly to `main`**. No pull requests.
- **Build environment:** code can be written in a Linux cloud session, but:
  - the iOS app must be built on a **Mac with Xcode**
  - the Windows apps must be built on a **Windows 11 PC** (Visual Studio)
  - the user builds and tests locally and reports errors back
- **App name:** placeholder `PhoneAsWebcam` (final name not decided)

## 3. Architecture
```
┌──────────── iPhone (Swift / SwiftUI) ────────────┐
│ Camera capture (AVFoundation)                    │
│ Hardware encode HEVC / H.264 (VideoToolbox)      │
│ Streaming server (Network.framework)             │
└──────────────────────────────────────────────────┘
        │  PRIMARY: USB cable (Lightning / USB-C)
        │           Apple's USB connection service (usbmuxd);
        │           needs the Apple Devices app or iTunes on Windows
        │  FALLBACK: Wi-Fi (local network)
        │  NOT USED: Bluetooth (< 2 Mbps, too slow for video)
        ▼
┌──────────── Windows 11 ──────────────────────────┐
│ Desktop app (C# / .NET 8 / WinUI 3)              │
│  - USB / Wi-Fi connection, discovery             │
│  - Hardware decode (Media Foundation / D3D11)    │
│  - Preview, settings, latency readout            │
│            │ shared memory / frame pipe          │
│            ▼                                     │
│ Virtual camera (C++ / C++/WinRT)                 │
│  - Media Foundation custom media source (COM DLL)│
│  - Registered with MFCreateVirtualCamera         │
│    (Windows 11 virtual camera API)               │
│  - Appears system-wide in Teams, Zoom, OBS,      │
│    Windows Camera app, browsers                  │
└──────────────────────────────────────────────────┘
```

## 4. Languages (decided)
| Component | Language | Reason |
|---|---|---|
| iPhone app | **Swift** (SwiftUI, AVFoundation, VideoToolbox) | Only practical way to get full camera quality on iOS |
| Windows virtual camera | **C++** | Loaded inside other apps; must be native code |
| Windows desktop app | **C#** (.NET 8, WinUI 3) | Fast to build a modern Windows 11 interface; syntax close to Java |

**Java was considered and rejected:** iOS has no Java runtime, a virtual camera can't be written in Java, and a Java desktop app would still need native glue code.

## 5. Quality requirements
### Camera and format selection (no caps)
- The user picks a camera: main (wide), ultra-wide, telephoto or front.
- The app **reads every format the selected camera actually supports** (`AVCaptureDevice.formats`): resolution, frame-rate ranges, HDR / 10-bit color.
- Only supported options are shown. **The best is chosen by default:** the highest resolution at the highest frame rate.
- Switching cameras refreshes the option list.
- Device-dependent extras appear only when supported: 10-bit HDR, Cinematic / portrait blur, Center Stage, high frame rates (e.g. 1080p120), 4K60.

### Encoding and transport
- Hardware HEVC (preferred) or H.264, low-latency settings.
- **Bitrate scales with the chosen format**, e.g. about 80–100 Mbps for 4K60 over USB.
- USB is the primary link; Wi-Fi is the fallback with adaptive bitrate.
- Target latency over USB: **about 50–100 ms**.

### Camera controls
- Lens selection and zoom
- Focus lock, exposure lock, white-balance lock
- Video stabilization **off** by default (lower latency)
- Portrait blur and Center Stage where supported

### Device behavior
- Screen dims while streaming, to save battery and reduce heat.
- **Thermal monitoring:** if iOS lowers the frame rate because of heat, show a **visible warning**. Never lower quality silently.
- The phone charges over the cable while streaming.

### Windows side
- Hardware decoding.
- The virtual camera **advertises the phone's current formats**, so OBS, Teams, Zoom etc. can choose.
- Note: Teams and Zoom usually cap their own output to 720p or 1080p. OBS and recording software can use the full 4K60.

## 6. Constraints and known limits
- On iPhone, the camera only works while the app is **in the foreground** (iOS restriction).
- USB on Windows depends on Apple's mobile device service (Apple Devices app or iTunes).
- Windows 11 is required for the native virtual camera API (`MFCreateVirtualCamera`).

## 7. Publishing notes
### iPhone (App Store)
- Apple Developer Program: **$99/year**. A company account needs a D-U-N-S number (about 1–2 weeks).
- A Mac with Xcode is required to sign and submit.
- Review takes about 1–3 days. Provide a **demo video and instructions** because reviewers need a PC to test.
- Permission explanations are required: camera, local network (and microphone, if added).
- A privacy policy URL and privacy labels are required.
- Beta testing through TestFlight.

### Windows
- Microsoft Store: free for individuals, about $99 one-time for companies. Certification takes hours to days.
- Outside the Store: a code-signing certificate avoids SmartScreen warnings (Azure Trusted Signing about $10/month, or a traditional certificate $200–400/year).
- **Expected timeline to both stores:** about 2–4 weeks, mostly account setup and review rounds.

## 8. Out of scope (from the original idea)
| Feature | Why it was dropped |
|---|---|
| Call audio through the PC | Windows doesn't expose Bluetooth call audio to third-party apps (restricted `phoneLineTransportManagement` capability) |
| SMS / iMessage | iOS gives apps no access; the Bluetooth message protocol is limited and fragile |
| AirPlay receiver for mirroring | Reverse-engineered protocol; legal and trademark risk |
| Controlling the iPhone from the PC | iOS doesn't allow touch injection |
| Remote PC lock, file transfer, clipboard, notifications | Feasible, but deferred; the scope is now webcam only |

## 9. Repo layout
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

## 10. Open items
- [ ] Final app name
- [ ] Confirm the user has a Mac (Xcode) and a Windows 11 PC for building
- [ ] Individual or company (UST) developer accounts
- [ ] Microphone passthrough (phone mic as a PC mic)? Not yet decided
