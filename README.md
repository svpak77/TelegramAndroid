# Forkgram Split — resizable two-pane layout for foldables and tablets

An unofficial build of [Forkgram](https://github.com/Forkgram/TelegramAndroid) (itself a fork of
Telegram for Android) with a **draggable divider between the chat list and the chat pane**, built for
foldables such as the **OPPO Find N5** (unfolded 2248×2180, 520 dpi). It works on any device where
Forkgram itself switches to the two-pane tablet layout.

Upstream behaviour: the panes are locked to `max(320dp, 35% of the width)`. On a 6.9″ foldable that is
almost half of the screen and cannot be changed. Here you drag the boundary with your finger, keep the
position across restarts, and the chat list can be squeezed down to bare avatars so the conversation gets
the rest of the screen.

> Unofficial client. Not affiliated with Telegram Messenger LLP. No telemetry, no analytics, no extra
> network endpoints — all traffic goes to Telegram's official servers, as in Forkgram.

## What this fork adds

| Area | Change |
|---|---|
| Divider gesture | The root split container intercepts touches within ±24dp of the boundary and drags the divider anywhere in any section (Chats, Settings, Contacts, Profile). Works even with an empty right pane. |
| Limits | Left pane `80dp … (width − 300dp)`; the fraction is stored in `SharedConfig.tabletSplitFraction` and restored on launch. |
| Avatar-only rail | When the chat list is narrower than `112dp`, `DialogCell` switches to `railMode`: nothing but avatars, badges moved into the bottom-right corner of the avatar and drawn on top with an outline. Placeholders (forum/premium/verified/pin/error/timer) are hidden. Settings and Profile collapse to icons the same way. |
| Compact bottom tabs | If the width is under `tabs × 56dp`, `MainTabsLayout` shows a single button for the active tab plus an arrow; tapping it opens a `PopupWindow` with the list of sections. |
| Reflow fixes | Text no longer breaks into single-letter columns while resizing: the container defers `requestLayout()` (`pendingRelayout`) and calls `relayoutPanes()` on every drag frame and on `ACTION_UP`; the width measured in `onMeasure` is passed to `onLayout` via `splitLeftWidth`. |
| Crash fix | "Settings → Devices" no longer throws an NPE in `SessionsActivity` (`actionBar == null` when `useSimpleActionBar()` is true). |
| Build | `arm64-v8a` only, to keep the APK small and the native build fast. |

Changed files (11): `AndroidUtilities.java`, `SharedConfig.java`, `LaunchActivity.java`,
`Cells/DialogCell.java`, `ActionBar/SimpleTextView.java`, `MainTabsLayout.java`, `MainTabsActivity.java`,
`ContactsActivity.java`, `DialogsActivity.java`, `SettingsActivity.java`, `SessionsActivity.java`.
Details in [FORK_CHANGES.md](FORK_CHANGES.md) (Russian).

## Install

Grab the APK from [Releases](../../releases):

- package `org.forkclient.messenger.beta` — installs **alongside** official Telegram and Forkgram;
- `arm64-v8a` only, Android 6.0+ (minSdk 23), version 13.2.1 (Forkgram base);
- signed with the maintainer's key: updates must come from this repository.

## Build it yourself

`gradle.properties` in this repository ships with `APP_ID=0` / `APP_HASH=0` on purpose (same as upstream) —
**you need your own Telegram API credentials**, otherwise the client cannot connect:

1. Log in at [my.telegram.org](https://my.telegram.org) → *API development tools* → create an app.
2. Keep the credentials out of git — put them in the Gradle user home, which takes priority over the
   project file (`%USERPROFILE%\.gradle\gradle.properties` on Windows):

```bash
cat >> ~/.gradle/gradle.properties <<'EOF'
APP_ID=1234567
APP_HASH=0123456789abcdef0123456789abcdef
EOF
```

3. Build (see [BUILDING.md](BUILDING.md) for the full toolchain, Docker image included):

```bash
git clone --recurse-submodules -b foldable-split https://github.com/svpak77/TelegramAndroid.git
cd TelegramAndroid
gradle -Parm64-v8a :TMessagesProj_App:assembleAfatRelease
# APK: TMessagesProj_App/build/outputs/apk/afat/release/
```

App name, icons and the update channel are in `TMessagesProj/src/main/res/values/strings.xml`,
`gradle.properties` (`APP_VERSION_NAME`, `APP_VERSION_CODE`) — change them before publishing your own builds.

## Known limitations

- Very narrow chat list: the search field and the folder chips are still clipped; Settings headings wrap
  mid-word below ~200dp.
- Horizontal swipe inside Profile may scroll the content instead of dragging the divider.
- Account switching inside "Devices" is not restored yet (only the crash is fixed).
- One split fraction for all sections; a separate ratio per section is not implemented.
- No divider highlight while dragging; fold posture changes (fold/unfold mid-drag) are untested.

## License and credits

GPLv2 (with the Telegram exception) — inherited from
[Telegram for Android](https://github.com/DrKLO/Telegram) via [Forkgram](https://github.com/Forkgram/TelegramAndroid).
Source of every binary published here is this repository, at the corresponding tag. Keep the license and
credit the upstream projects if you redistribute.
