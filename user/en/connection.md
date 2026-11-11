---
title: When the server cannot be reached
summary: Check connection status and path, then pair again if needed
---

## Check the connection

On the Apps page, tap “连接” (Connection) in the upper-right corner. The page shows “状态” (Status). When a connection is established it also shows “路径” (Path), for example “直连 · … ms” (Direct · … ms) or “中继 · … ms” (Relay · … ms).

Status meanings:

- “还没有配对” (Not paired): follow the Pair page steps.
- “连接中” (Connecting): wait for the connection.
- “已连接” (Connected): app web pages and chat can use the server.
- “家里的服务器不认这台设备，请重新扫码配对” (The home server does not recognize this device; scan again): open “重新配对” (Pair again) and use a new code.
- Any other disconnected state: the page shows the exact reason returned by the core. Follow that reason and check Status again later.

## Try these steps first

1. Confirm that the server is running; the phone and server are either on the same local network or can both reach the internet.
2. Reopen Wonton.
3. If the status is still not “已连接”, open “连接” and read the specific reason.
4. If the device is not recognized, tap “重新配对” and use a new pairing code instead of continuing with the old device identity.

When the connection is down, opening an app web page shows a dialog titled “打不开” (Cannot open) with “还没有连上家里的服务器：…” (Not connected to the home server: …). Restore the connection, return to the app details, and tap “打开” (Open).

## Outbound proxy

“出网代理” (Outbound proxy) configures the proxy used by the server or outbound apps to reach external services. When needed:

1. Tap “出网代理”.
2. Enter an address under “代理地址” (Proxy address), for example `http://192.0.2.10:7890`.
3. Enter networks that bypass the proxy under “直连网段” (Direct networks), for example `100.64.0.0/10`.
4. Tap “保存” (Save).
5. If the dialog says “配上代理要重启这些应用” (These apps must restart to use the proxy), review the list and tap “重启” (Restart). Tap “取消” (Cancel) if you do not want to restart them. Removing a proxy also lists the apps that need restarting.

The page also shows the outbound-proxy probe as “探测中” (Probing), “通” (Reachable), “不确定” (Uncertain), or “不通” (Unreachable). For “不确定” or “不通”, read the adjacent reason, fix the proxy address or direct networks, and tap “保存” again. When a proxy is configured, tap “取消代理” (Remove proxy) to remove it; the listed apps may also need to restart.

If the address is set by the installation configuration, the page says “地址由 WONTON_EGRESS_PROXY 指定。要改的话，改安装目录的设置，再运行一次安装脚本。” (The address is set by WONTON_EGRESS_PROXY. Change the installation settings and run the installer again.) The app cannot override it.
