---
title: Pair and add devices
summary: Connect a phone to your home server and pair another device
---

## Pair this device

Open Wonton. Before a device is paired, the app opens on the “配对” (Pair) page.

1. On the same local network as the server, open the server’s IP address in a browser. The installer prints this address in the terminal. The page contains a pairing QR code.
2. Scan it with the app, or paste the code into “粘贴配对码” (Paste pairing code).
3. Optionally change “这台设备的名字” (This device’s name).
4. Tap “配对” (Pair) and wait while “正在配对…” (Pairing…) is shown.

After pairing, the app opens the “应用” (Apps) page. The connection information is saved, so later launches do not ask for the code again.

### When the camera cannot be used

- “等待相机权限…” (Waiting for camera permission…): wait for the system permission request.
- “云吞没有相机权限，扫不了码。可以在系统设置里打开，或者粘贴配对码。” (Wonton has no camera permission, so it cannot scan. You can enable it in system settings or paste the pairing code.): tap “打开系统设置” (Open system settings), or paste the code. Android may show this notice with “允许使用相机” (Allow camera use); tap it to ask again.
- “这台设备不能扫码，请粘贴配对码。” or “这台设备没有相机，请粘贴配对码。” (This device cannot scan / has no camera; paste the code): use the text code.
- If the scanner stops: tap “重新打开相机” (Reopen camera), or paste the code.

### Pairing failures

- “服务器不认这个配对码” (The server does not recognize this pairing code): check that the code came from the server page and follow the server’s message.
- “配对码过期了” (The pairing code expired) or “这个配对码已经停用” (This pairing code has been disabled): get a new code from the server page.
- “连不上家里的服务器” (Cannot connect to the home server): make sure the phone and server are on the same local network, or that the server can reach the internet, then tap “用这个码再试一次” (Try this code again).
- “家里的服务器出错了” (The home server returned an error): wait and tap “用这个码再试一次”.
- “App 和服务器的版本对不上” (The app and server versions do not match): update both the app and the server.

Invalid text is rejected on the pairing page with the server’s reason. Do not keep submitting the same invalid code.

## Pair another device

On a device that is already paired:

1. Open the “应用” (Apps) page.
2. Tap “连接” (Connection) in the upper-right corner.
3. Tap “添加设备” (Add device).
4. Wait for the QR code, then scan it with Wonton on the new device.

The page shows the remaining validity time and the code as selectable text. If the new device cannot scan, paste that text into its Pair page. A pairing code is single-use; after it expires, tap “换一个码” (Get another code).

If the page says “没要到配对码” (Could not get a pairing code), tap “再试一次” (Try again).

## Pair again

Open “应用” → “连接” → “重新配对” (Re-pair), then scan or paste a new code as above. The app returns to “连接” after success.

If “连接” shows “家里的服务器不认这台设备，请重新扫码配对” (The home server does not recognize this device; scan again), use the same re-pair path.
