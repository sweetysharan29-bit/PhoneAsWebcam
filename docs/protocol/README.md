# Wire protocol

The contract between the iPhone app and the Windows app. Both sides implement it:
- Swift: `iphone/PhoneAsWebcam/Protocol/`
- C#: `windows/Connection/`

To be written here: message framing, handshake and version check, camera format advertisement, control messages (lens, zoom, focus / exposure / white-balance locks), status messages (thermal warnings, battery), and video packet transport.
