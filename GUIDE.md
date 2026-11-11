# ThumbPilot

Edition: public 1.5.0 | Nov 2026 | MIT

## READ THIS FIRST (the whole job in 10 lines)
STEP 0, every session: `git pull --ff-only` and note the version at the top of this file. Start every report with `[GUIDE vX.Y.Z]`. This file is the ONLY source of truth: if any zip, folder, chat or password-manager note disagrees, this file wins. Never work from a copy.

Default stance: ACT. the owner gave you the phone to use. Do the task, fix your own problems, keep going.
1. Send one `run` on `display:"main"`. It unlocks, switches profile (`switch_user:true`) and does the app task. No flags, no preflight needed.
2. Any hold: look it up in section 4, apply the fix, run it again. A hold is a puzzle, not a stop. If the same fix fails twice, try a different approach before you tell the owner.
3. the gateway not answering? Use USB on the host Mac (section 2). Both routes work; pick whichever is up.
4. Be quick. Do not re-check what you just checked. One doctor per session is plenty. Skip the paperwork.
5. Routine app work (open, search, browse, read, fill forms, compare, add to cart, draft messages) needs no permission. Just do it.
6. Ask the owner only before: spending money or paying, sending a message or email, changing security or account settings, entering a PIN/OTP/password (those screens are the owner's), or a phone reboot.
7. Never type a wrong PIN twice (Android lockout). If the helper says still_locked, guard_locked or rate_limited, stop that one thing and tell the owner.
8. Learned something? Add it to this file on your branch (section 10). Short is fine.
9. Report in one or two lines: what happened, what is next.
Everything below is reference. Use it when you need detail; you do not need to follow it step by step.

Update rule: this guide is the source of truth for NT2 operations. It supersedes earlier handoffs, ZIPs, and copy-paste exports. Record new holds, fixes, timings, and app procedures on a topic branch using the review workflow in section 10.

## 1. Device and profiles

- A rooted Android phone controlled over a restricted gateway or USB. Example: Nothing Phone (2), model A065, serial `<ADB_SERIAL>`, screen 1080x2412.
- Android users: `<ANDROID_USER_ID>` values such as 0 (admin), 10, 11, 12 depending on your setup. Use the profile the task names. Switching changes the whole phone.
- FIRERPA Lamda provides UI hierarchy and scoped taps. The controller validates and dispatches every action. A decision model (JEV or similar) picks actions from structured observations.
- Optional viewer: `<VIEWER_URL>` (keep it on a private network).

## 2. Two routes

Choose the gateway for the agent and normal wireless work. Choose USB-C for an authorized local session on the prepared host Mac. Use one phone operator across both routes, including legacy tools. Before changing routes after a failed mutating job, resolve its outcome and ownership. A clean pre-input failure can move to the other approved route when that route is available. The controller itself has no automatic fallback.

### Wireless gateway

- Restricted gateway `<GATEWAY_USER>@<GATEWAY_HOST>:22`. Verbs are `capabilities`, `status`, `doctor`, `observe`, `run`, and `act`. Send one UTF-8 JSON object on stdin and EOF within 10 seconds, up to 32 KiB.
- Use the actual caller's approved SSH identity and pinned host binding. the agent uses its existing approved identity. Use `ssh_config.template` and `known_hosts.sg` from your handoff folder to resolve an approved binding. Pinned server fingerprint `<SSH_HOST_KEY_FINGERPRINT>`.
- `capabilities {}` checks the interface and release without contacting the phone. `status` reads cached results. `doctor` is a live read-only check. Confirm `current_user`, `unlocked`, `phone_owner`, and `action_ready` before assuming the requested profile is usable.

```sh
printf '%s\n' '{}' |
  ssh -T -oBatchMode=yes -oForwardAgent=no -oStrictHostKeyChecking=yes \
    -oConnectTimeout=12 <GATEWAY_USER>@<GATEWAY_HOST> capabilities

printf '%s\n' '{"user":10}' |
  ssh -T -oBatchMode=yes -oForwardAgent=no -oStrictHostKeyChecking=yes \
    -oConnectTimeout=12 <GATEWAY_USER>@<GATEWAY_HOST> doctor

ssh -T -oBatchMode=yes -oForwardAgent=no -oStrictHostKeyChecking=yes \
  -oConnectTimeout=12 <GATEWAY_USER>@<GATEWAY_HOST> run < /absolute/path/to/request.json
```

Substitute the approved alias if the caller's identity and host pin are configured under one. Allow at least 300 seconds for the full SSH command with a 150-second job deadline. `ConnectTimeout` covers connection setup. A `no_private_route_available` result means the gateway reverse-route probe failed. `private_tunnel_start_failed` means the private tunnel failed to start. Use the read-only retries in section 4. Either result can occur while the phone is otherwise connected.

### Direct USB-C on the MacBook host Mac

Use a data-capable cable and the host Mac's existing ADB authorization, pinned Lamda certificate, and local Python environment. Verify the specific NT2 serial and the host Mac-owned forward `<ADB_SERIAL> tcp:<LOCAL_PORT> tcp:<DEVICE_PORT>`. If the forward is missing, the prepared host's operator can restore it after checking the phone and port are available, using `adb -s <ADB_SERIAL> forward --no-rebind tcp:<LOCAL_PORT> tcp:<DEVICE_PORT>`. The USB job CLI uses the forward without managing its lifecycle.

```sh
adb -s <ADB_SERIAL> get-state
adb forward --list
cd '<INSTALL_DIR>'

printf '%s\n' '{"user":10}' |
  <PYTHON> -m nt2_runtime.usb_job doctor

<PYTHON> -m nt2_runtime.usb_job run \
  < /absolute/path/to/request.json
```

The USB CLI has `doctor`, `observe`, `run`, and `act` with the shared request schema. It returns a compact observation by default and supports `--full` for authorized inspection when needed. Save its job ID and result locally for reconciliation. the gateway `status` covers the gateway jobs. USB carries phone traffic over the cable, while any JEV decisions still use the Mac's configured online route.

## 3. Unlock and profile switching

- Use one `run` with `display:"main"` for the requested app task. Add `switch_user:true` for an authorized change to a different foreground profile. The runtime verifies the new user and rereads keyguard after a three-second settle. Main-display `run` and `act` default to the installed automatic unlock helper. `unlock:false` opts out of the helper. `doctor` and `observe` leave the lock and profile alone.
- Use `run` to start on a locked or different profile. `act` depends on a fresh observation ID from the intended foreground surface. Virtual-display `run` requires ready profile storage and an owned per-job display. Use `main` for automatic unlock and for multi-job work on the same visible screen.
- Helper v3 waits for the correct profile's PIN pad and checks the pad and target user before entering digits. A recorded failed attempt triggers the 10-minute rate limit. Two consecutive failures trigger a separate guard lock. The gateway launches the helper detached, polls only recognized status words, and repeats only a known `NO_PAD` once internally. Keypad calibration and the protected PIN binding belong to the installed helper. Its source is a maintainer reference.
- The coordinated Settings example below can be saved as `request.json` and used on either route. Omit `switch_user` when the target profile is already in front. Inspect the Settings destination with fresh `observe` and confirm unlock and owner state with `doctor` during acceptance. A known completion contract can complete without calling JEV.

```json
{
  "user": 10,
  "package": "com.android.settings",
  "display": "main",
  "agent": "nt2-operator",
  "switch_user": true,
  "goal": "Open Android Settings and stop on its first visible Settings screen. Leave every setting unchanged.",
  "max_steps": 1,
  "deadline_seconds": 150
}
```

## 4. Recovery by result

| Result | Action |
| --- | --- |
| `unlock_helper_no_pad` | The helper did not enter a digit. After the internal repeat, wait 15 seconds before a caller retry only if the result confirms released ownership and no app input. The v3 handoff allows up to three caller retries. |
| `unlock_helper_wrong_user`, `profile_mismatch_explicit_switch_required`, `profile_switch_not_verified` | Check the actual user and owner. Set `switch_user:true` if the switch is still needed and authorized, and the previous outcome is known. |
| `unlock_helper_still_locked`, `unlock_helper_guard_locked`, `unlock_helper_rate_limited`, `unlock_helper_no_pin`, `unlock_helper_bad_digit` | Stop the helper attempts and report the exact reason. Use the physical first unlock after a reboot. the owner handles the protected binding if it needs correction. |
| `stale_observation_no_action` | The selected app input was withheld. Check any profile or lock preparation that occurred, then observe again and bind the intended control to the new IDs. |
| `unknown_request_fields_<names>` | Remove unsupported fields using the current schema, then resubmit the corrected preflight request. |
| `model_blocked` or `model_safety_hold` | Check the visible screen and effect. For ordinary authorized navigation, use a narrower goal with a clear stop. Route an actual permission or secure step to the owner. |
| Busy owner, unknown helper result, timeout, lost transport after possible dispatch, or unverified cleanup | Preserve the job and owner records. Reconcile before repeating, switching routes, or releasing resources. On the gateway send `{"job_id":"ACTUAL_HOST_JOB_ID"}` to `status`. For USB use the local result and resolve the owning session. USB has no `status` verb; keep the local job JSON. |
| `no_private_route_available` or `private_tunnel_start_failed` on `doctor` | Wait 20 seconds and retry the read-only check, up to three attempts. Report the exact failed route if it persists. Use USB when available after prior ownership and outcomes are clear. |
| Agent about to ask the owner to unlock by hand | Do not. Unlock is the helper's job. Re-run with `display:"main"` and follow this table. Only the section 7 cases need the owner. |
| ADB `unauthorized` | the owner approves the intended Mac on NT2. Verify the serial and `device` state again. |

Retained locks and displays represent work that requires reconciliation. Release only resources owned by the identified job, after its outcome is known. When the last known result is cached, use a fresh live observation to establish the current screen.

Example reconciliation: after a held job, run a fresh `observe` and `doctor`, confirm the lock belongs to your own dead job (job id, agent, user, display), then release it only through the installed `Ownership.release` guard.

Lock rule (reconciled 1.2.0): an agent may release a retained lock only if it is the agent's own dead job, after a fresh `observe` and `doctor` show the lock identity matches (job id, agent, user, display) and the installed `Ownership.release` guard is used (matching token, user, boot_id). Never `rm` the lock directory by hand, never release another agent's or the owner's job.

### Requests and model settings

- `run` uses `user`, `package`, `display`, and a bounded `goal`. Use `switch_user` only as needed. Typical navigation uses 4 to 8 steps. The active gateway permits up to 12 steps and 300 seconds. Use `expect_labels`, `expect_activity`, or `expect_text_keys` based on observed destination evidence. An already matched contract can return with zero model decisions. A model `DONE` without a satisfied contract returns `needs_review` for independent inspection.
- `observe` and `act` use the main display. A direct `act` includes the fresh `observation_id` and a nested `action` such as `{"op":"TAP","target_id":123}` using the real current ID. Use it on the already prepared profile. For `TYPE`, supply an explicit `text_key` and its exact non-secret value in `texts`. `expect_text_keys` checks exact editable-field text on two fresh reads. Verify the intended field and role, especially when several fields can contain the same value.
- The pinned JEV model is `typesafe/jev-1.13`, with expected returned revision `typesafe/jev-1.13-20260917`. The runtime uses `POST https://openrouter.ai/api/alpha/decisions` with one combined offered action and a safety decision. Active model proposals require a safety score of at least `0.90`, then pass fresh UI and ownership guards.
- JEV receives compact structured hierarchy and OCR-derived labels. Screenshot OCR runs on the controller host. Lamda supplies the phone UI and native action transport. `run` may call JEV when its destination contract is unmet. `capabilities`, `status`, `doctor`, `observe`, and guarded `act` require no JEV inference. Preserve the pinned model, specialized request contract, and provider binding unless an approved replacement passes comparable tests.
- Reuse fresh readiness and combine switch, unlock, and coherent navigation in one job. Use hierarchy first. The runtime can automatically use OCR when it sees fewer than three clickable or editable controls. `visual:true` forces OCR for that job, so use it for the stage that needs it. Use immediate observation-bound `act` for a known stable control and for reviewed final actions. Submit Chinese query text directly when appropriate. Put English directly below Chinese excerpts shown to the owner.
- OCR may be unavailable on the USB host. When USB hierarchy lacks a required control, inspect `vision.ocr_reason` and use a verified route with OCR support or identify the specific blocked field.
- Use `model_use` to distinguish attempts, decoded responses, accepted decisions, and acknowledged actions. Missing counters on older or incomplete receipts mean unknown. Verify the actual destination after action acknowledgment. Preserve private field evidence in the appropriate private job record.

### Shopping apps (generic)

1. Resolve the market, profile and installed regional app. Confirm product, variant, quantity, budget and delivery needs.
2. Search with exact text in a real editable field and read it back. Check results match the query.
3. Match the variant and quantity on the detail page. Read price and pack size separately.
4. Review address, delivery date, fees and the final total. Only submit an order the owner has authorized for this task, then verify the order reference and payment state.

## 6. Standing preferences
- No em dashes. English translation directly below any Chinese shown to the owner.
- WhatsApp and email: draft in TextEdit and queue it; the owner approves before anything is sent.
- Share the phone politely: one operator at a time, and if the owner is using the screen, wait or use a virtual display.
- A short outcome line beats a full audit trail. Keep job ids handy for debugging, not for ceremony.

## 7. When the owner is needed (the whole list)
- Money: paying or buying beyond what the owner already approved in this task.
- Sending: a message or email the owner has not approved.
- Secure screens: PIN, OTP, password, bank, payment confirmation, identity.
- Wrong-PIN risk: helper reports still_locked, guard_locked, rate_limited or no_pin.
- Phone reboot (one in-person unlock), or the phone is offline on BOTH routes with no cable.
- ADB `unauthorized` on a new computer (one tap, once per computer; the host Mac is done).
Everything else: handle it yourself.

## 8. Secrets policy

Never put secrets in this repo or in prompts: SSH private keys, PIN values, certificates, viewer passwords, API keys, OTPs, card data. Keep them in a password manager and reference them by item name (`<CREDENTIAL_REF>`). Agents fetch them at runtime.

Your own values go in a private file that is NOT committed (see `config.example.env`).
