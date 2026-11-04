---
title: Install and manage apps
summary: Get, open, update, and uninstall plugin apps from the app store
---

## Get an app

1. Open the “应用” (Apps) tab.
2. Tap “应用商店” (App Store), the last tile in the grid. If no app is installed, “去应用商店” (Go to App Store) opens it too.
3. Browse “已审核” (Reviewed) or “社区” (Community), or type a keyword in “搜索” (Search).
4. Filter with “全部” (All) and the available tags.
5. Tap an app’s name or description to inspect it; you can also tap “获取” (Get) on the row to open the install preview directly.
6. On the “安装” (Install) page, check the version, source, permissions, requirements, and notices. Tap “安装” (Install) when you agree.

While the install runs, the list shows its progress. On success it shows “已安装” (Installed), and the app appears on the Apps tab.

Pay attention to the “高权限” (High permissions) tag and the “权限” (Permissions) section. The confirmation page disables installation when the host blocks the app or the core does not support one of its permissions.

## Open and inspect an app

Tap an app icon on the Apps tab. An agent with a chat entry opens its direct chat; an app with a web entry opens its web page. An app with no directly openable entry, or one that is installing, updating, or uninstalling, opens its detail page.

In the App Store, tap an app’s name or description to open its details. The page can include “新内容” (What’s new), “权限” (Permissions), “要求” (Requirements), and “信息” (Info), plus “主页” (Homepage) and “代码仓库” (Repository) links.

If a web page cannot open, the dialog says “打不开” (Cannot open) and “还没有连上家里的服务器：…” (Not connected to the home server: …). Restore the connection under “应用” → “连接”, then tap “打开” (Open) again.

## Update an app

An installed app with an older version has an “更新” (Update) button in the App Store and on its detail page. Tap it, check the version change, release notes, and new permissions on the “升级” (Update) page, then tap “升级” (Update).

## Uninstall an app

Long-press the app icon on the Apps tab, or tap “卸载” (Uninstall) on the installed app’s detail page. The confirmation dialog offers:

- “卸载，保留数据” (Uninstall, keep data): remove the app but keep its data.
- “卸载并删除数据” (Uninstall and delete data): also delete its keys and Bot account; this cannot be undone.
- “取消” (Cancel): do not uninstall.

## Read failures

A task shows “正在安装” (Installing), “正在升级” (Updating), or “正在卸载” (Uninstalling). Success shows “已安装” (Installed), “已升级” (Updated), or “已卸载” (Uninstalled). A failure shows the matching failure label, the specific reason, and the next step.

Store-list failures:

- “连不上 GitHub，列表可能不是最新的” (Cannot reach GitHub; the list may be outdated): check the server’s outbound connection; the list may still be an older copy.
- “GitHub 太慢，没有及时答完，列表可能不是最新的” (GitHub was too slow; the list may be outdated): refresh later.
- “GitHub 限速了，过几分钟再试” (GitHub rate-limited the server; try again in a few minutes): wait, then refresh.
- “商店的列表读不懂，稍后再试” or “商店出了问题，稍后再试” (The store list/server has a problem; try again later): refresh later.
- “商店的列表比家里的服务器新，先升级家里的服务器” (The store list is newer than the home server; update the home server first): update the server core.
- “商店的来源出了问题，稍后再试” or “有的没读到，列表不完整” (The store source failed / the list is incomplete): refresh later and check the list.
- “社区应用已关闭” (Community apps are disabled): choose an app under “已审核” (Reviewed).

Install, update, and uninstall failures:

- “连上家里的服务器后再试。” (Reconnect to the home server, then retry.): restore the connection and retry.
- “应用在确认之后变了，请重新查看、确认。” (The app changed after confirmation; review and confirm again.): tap “重新查看” (Review), check the confirmation page again, and retry.
- “家里的服务器不认这台设备，请重新扫码配对。” (The home server does not recognize this device; scan again.): use “应用” → “连接” → “重新配对”.
- “家里的服务器连不上应用的来源，稍后再试。” or “来源里没有这个应用或版本，刷新商店后再试。” (The source is unavailable / the app or version is missing): refresh the store later.
- “家里的服务器不支持这个应用，先升级家里的服务器。” (The home server does not support this app; update the home server first.): update the server core.
- “它已经装了，或者正在安装、升级、卸载，等那边做完。” (It is already installed or another task is running; wait for it to finish.): wait for the task in progress.
- “应用包不合法，请联系应用的作者。” (The app package is invalid; contact its author.): stop and contact the app author.
- “家里的服务器正在升级，升级结束后再试。” (The home server is updating; try again when it finishes.): wait for the update to finish.
- “检查家里服务器上的代理，或者在出网代理设置里改地址后再试。” (Check the server proxy or change it under Outbound proxy, then retry.): fix the proxy under “应用” → “连接” → “出网代理”.
- “家里的服务器出了错，稍后再试。” (The home server returned an error; try again later.): retry later.
- “App 和家里的服务器版本对不上，请更新 App 或家里的服务器。” (The app and server versions do not match; update one or both.): update the app and server.

If the result says “结果不确定：…” (Result uncertain: …), do not guess whether the app changed. Refresh the Apps page and check the actual state.
