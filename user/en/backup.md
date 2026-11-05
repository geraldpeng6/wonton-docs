---
title: Back up chat history
summary: Save a recovery key so a new device can retrieve earlier messages
---

## Save your recovery key

Chat backup is enabled by default. After the first device is paired and verified, the app creates the backup automatically; there is no manual enable step. An unverified device will not create a backup. Adding a device to an account that already has a backup does not generate a new key.

1. When the chat list reminds you to save your recovery key, tap the reminder to open “聊天记录备份” (Chat backup). You can also open it from “设置” (Settings).
2. Store the key in a password manager or another safe place. Tap “复制” (Copy) if needed. Until you confirm, the app temporarily keeps the key on this device, so you can still view it after closing and reopening the app.
3. Once you have stored it safely, tap “已保存” (Saved). The reminder disappears and the device deletes this displayed recovery key. You cannot view it here again afterward.

You are responsible for keeping your recovery key; the server does not keep it for you. Never send it in chat.

## Retrieve messages on a new device

After pairing a new device, the chat list reminds you to enter your recovery key. Tap the reminder, or open “设置” → “聊天记录备份”, paste your saved key into “恢复密钥” (Recovery key), and tap “取回以前的消息” (Retrieve earlier messages). The button stays disabled while the field is empty. Pairing alone does not decrypt earlier messages; you need the correct recovery key to retrieve them.

The page shows “正在处理…” (Processing…) while it works. Return to the chat list when it completes.

## If you lose your recovery key

Choose to reset the backup on the Chat backup page and read the confirmation carefully: resetting invalidates the old recovery key, and messages in the old backup cannot be retrieved. After you confirm the reset, the app provides a new recovery key and the chat list reminds you to save it again. Store the new key safely, then tap “已保存” (Saved). Other devices need the new key.

## Status and errors

- “聊天记录已备份” (Chat history backed up): backup is enabled; use the recovery key when changing phones or adding a device.
- “保存恢复密钥，换手机时要用” (Save your recovery key; you will need it when changing phones): save the key shown on the page, then tap “已保存”.
- “需要恢复密钥” (Recovery key required): enter the key on the backup page.
- “还不知道备份的情况” (Backup status is unknown) or “连上家里的服务器后才能确认” (Connect to the home server to check): restore the connection first.
- “恢复密钥不对，检查后再输一次” (The recovery key is incorrect; check it and try again): check your saved key and retry. Never send it in chat.
- For other failures, follow the reason shown in the error. If the result is uncertain, reopen the backup page and check the status.
