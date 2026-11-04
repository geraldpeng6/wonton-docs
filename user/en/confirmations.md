---
title: Confirmation cards
summary: Review what an agent will do, then allow or reject it
---

## Find the card

A confirmation card appears in the private room “Wonton” for the core account. If it is not in the room you are viewing, the app shows a banner such as “标题，等你确认” (Title, waiting for your confirmation) or “几张卡片等你确认” (Several cards waiting for your confirmation); tap it to open the room. Android may also show a system notification. The Apps status area shows “等确认” (Waiting for confirmation).

## Read it before choosing

The card shows a title, what the operation will do, parameters, and the choices supplied by the core. Emphasized lines call out details that need care, such as deleting data or files supplied by a Bot without review. Button labels come from the requesting plugin; common choices include “允许” (Allow), “拒绝” (Reject), “安装” (Install), “不装” (Do not install), “卸载” (Uninstall), and “取消” (Cancel).

After you tap a choice, the buttons are disabled and the selected button shows progress. Do not tap repeatedly. The card keeps the result:

- “正在执行” (Running): the core is processing it.
- A success message: the operation finished; the core supplies the exact text.
- “没有成功：…” (Not successful: …): the operation failed; follow the reason.
- “结果不确定：…” (Result uncertain: …): the core cannot confirm the result. Check the relevant page before retrying an operation that may already have completed.
- “已拒绝，没有执行” (Rejected; nothing was done): the operation did not run.
- “已过期，没有执行” (Expired; nothing was done): the confirmation deadline passed.

If a problem line appears above the choices, follow it. When no choices remain, the card cannot be answered again.

## When a card is expected

A data-changing tool used by an agent or plugin, installing or uninstalling an app, and saving files into a shared folder can require a confirmation card. A read-only operation should not be treated as completed merely because a card is visible. High-permission app installs must be confirmed in Wonton.

If you are unsure, choose “拒绝” (Reject) and ask the agent what it will change and which data it will affect.
