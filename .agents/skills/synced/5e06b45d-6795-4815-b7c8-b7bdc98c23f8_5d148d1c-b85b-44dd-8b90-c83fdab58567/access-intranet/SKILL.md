---
name: "access-intranet"
description: "Open Epic for OCHIN in the native Mac Hyperspace app (what the user calls Hyperdrive), or open the practice intranet or Epic web remote in the browser, then stop at the sign-in screen for the user. Use whenever the user says \"open Epic\", \"get me into Epic\", \"Hyperdrive\", \"Hyperspace\", \"OCHIN\", \"access intranet\", \"open intranet\", \"entranet\", \"Epic remote\", or otherwise asks to get into the EHR or an internal work site, even if he does not name the app."
---

# Access Intranet

Gets the user to the sign-in screen of Epic (OCHIN) or an internal work site, then hands off. The default for anything Epic is the native Mac app, because that is how he wants to work. The web remote is only a fallback.

## Routing

| What he says | Route |
|---|---|
| Epic, OCHIN, Hyperdrive, Hyperspace, "the EHR", "chart" | **A. Native Hyperspace app** (default) |
| "Epic remote", "web Epic", "remote.ochin", or the native app fails | **B. Browser**: https://remote.ochin.info |
| Intranet, "entranet" | **B. Browser**: intranet URL (blank for now; ask once) |

"Hyperdrive" is the user's name for the app. On his Mac the app is **Hyperspace**, bundle ID `com.epic.hyperspace`. "Entranet" means intranet. Don't ask about either one.

Some apps on his Mac look relevant but are the wrong target. Don't use them unless he names them:
- **Epic** (`com.hiddenreflex.Epic`): a privacy web browser, not the EHR.
- **Epic Access** (`app.glaze.macos.14ggx99p-local`): a locally built wrapper of unknown purpose.

## A. Native Hyperspace app

1. **Load computer use.** If the `mcp__remote-devices__computer_*` tools are deferred, load all of them in one ToolSearch call (query `mcp__remote-devices__computer_`, max_results 100). If only `enable__mcp__remote-devices__computer` is present, call it first.
2. **Get access.** Call `computer_resolve_access` with `["com.epic.hyperspace"]`, then `computer_request_access` with the returned entries verbatim and a one-line reason ("Open Epic Hyperspace to the OCHIN sign-in screen"). If the bundle ID doesn't resolve, the app was removed or renamed. Run `computer_list_apps` with query `hyperspace`. If it isn't installed, say so and offer route B.
3. **Launch in the background.** Run `computer_open_application("com.epic.hyperspace")`. Work with the `computer_app_*` tools so he can keep using other apps.
4. **Wait for the picker.** The window is plain black for roughly 5 to 12 seconds while it starts. Wait about 8 seconds, then `computer_app_screenshot` at scale 0.6. Repeat once if it's still black. If it's still black after about 25 seconds, report that it's stuck and stop.
5. **Select the environment.** The first screen is **"Select An Environment"**. Under Recent Environments it shows tiles labeled Local Application / Ochin:
   - **OCHIN Production**: the default. Use it unless he asks for something else.
   - **remote**: use it only if he asks for "remote" by name.
   Click the tile by its `element_index` from the screenshot's AX summary. Positions shift, so match the title text, not coordinates. Then click **Connect**.
   If the environment he wants isn't listed, type its name in the Search field. If several matches look plausible, ask which one.
6. **If the Connect click is blocked** by a permission or safety check (this has happened before on the live production environment), don't retry or work around it. Leave the picker open and tell him: "Hyperspace is open on the environment picker. Click OCHIN Production, then Connect." Then stop.
7. **Hand off at sign-in.** When the Epic login screen appears, stop. Tell him it's ready and offer to continue once he's signed in. If Epic is already signed in (a session is still open), describe in a sentence what's on screen and ask what he wants to do next.
8. **Release the lock.** Call `computer_release_lock` at the end of the turn and leave the app open.

### When it fails

Report exactly what the screen shows, in one sentence. Common causes:
- "Unable to connect" or a timeout: OCHIN is down, or the network is blocking it. Offer route B.
- A license, version, or "not authorized for this device" message: the organization hasn't enabled the native Mac client for him. This needs Erie IT or OCHIN support, so say that directly. Don't troubleshoot further.
- An update prompt: tell him. Don't accept App Store or terms dialogs for him.

## B. Browser (web remote or intranet)

1. Use the session's preferred browser (the built-in browser when available, otherwise Codex in Chrome), following that browser's own skill. Open a new tab, and never reuse his existing tabs.
2. Navigate to the URL. Wait 2 to 3 seconds and take a screenshot. Confirm the page loaded and isn't a DNS error, a VPN block, or a certificate warning. If it failed, state what the page shows. For the intranet, an on-network or VPN requirement is the usual cause.
3. If a login form appears, stop and hand off. If he's already signed in, describe the page briefly and ask what he wants to do next. If the remote site wants to download a Citrix or launcher file, ask before downloading. Give the filename and source when you ask.
4. If he gives the intranet URL, offer to save it to this skill's routing table. Saving means proposing an updated skill, so the next run doesn't have to ask.

## Rules

- **Credentials are entered only by the user.** This includes usernames, passwords, MFA codes, badge taps, and PINs. Never type them, autofill them, or fetch them from a password manager, even if he asks. Epic sign-in carries prescribing and EPCS authority, so this is non-negotiable.
- **Don't act inside Epic after sign-in** unless he asks for a specific task in that turn. Opening the app does not authorize chart actions.
- **Keep the report short.** Use one or two sentences, plus a screenshot only when something failed or is ambiguous. Don't recap the steps.
- **Leave the app or tab open.** He asked to see it.