# Phone controls

## Existing MCP connector

Use the server already attached to the session if it exposes phone controls for the intended physical iPhone. Inspect its tool schemas first. A working connector for that phone handles its own device setup. Use the provider's documented URL or local command if a connection still needs to be added.

For example, to register an authenticated Streamable HTTP endpoint with the Codex CLI, use the server URL and a bearer token supplied to Codex through a secret environment variable. An OAuth endpoint uses the provider's sign-in flow. [Codex MCP configuration](https://developers.openai.com/codex/mcp/)

```sh
codex mcp add iphone --url "$IPHONE_MCP_URL" --bearer-token-env-var IPHONE_MCP_TOKEN
codex mcp list
```

The following tool names and shapes cover the screenshot-relative interface. Follow the schemas your connector actually advertises. If it requires session initialization, complete the advertised step before input. A connector that gates input on the `driving-the-phone` guide may ask for `load_skill({"slug":"driving-the-phone"})` once per MCP session.

| Tool | Arguments | Behavior |
| --- | --- | --- |
| `screenshot` | `{}` | Capture the current screen. |
| `tap` | `{"x":123,"y":456}` | Tap in the latest image's pixel space. |
| `swipe` | `{"x0":123,"y0":456,"x1":123,"y1":250}` | Swipe between two points in the same image. |
| `type` | `{"text":"Example"}` | Type into the focused text field. |
| `home` | `{}` | Go to the Home Screen. |
| `launch_app` | `{"bundle_id":"com.apple.Preferences"}` | Foreground an installed app by its exact bundle identifier. |
| `swipe_back` | `{}` | Try the app's back navigation. |
| `long_press` | `{"x":123,"y":456}` | Hold at a point when the server offers this tool. |

The screenshot origin is the upper-left corner. Pass coordinates in the returned image's original dimensions, even if a viewer displays it smaller. These connectors typically return a screenshot after a successful input. Inspect that image and capture a fresh one when the result has no image, the screen changes independently, or the state is still settling.

When the connector offers a `keyboard` argument for `type`, match the focused field's actual layout. Use `email` for an email field, `url` for a URL field, `phone` or `number` for a numeric pad, `decimal` for a decimal pad, and `punctuation` when the keyboard opens on its punctuation plane. Omit it for the regular layout.

## Local AXe

Use the installed AXe 2 CLI with the `IPHONE_UDID` identified in the [setup guide](setup.md). Start with an accessibility tree and a screenshot. To capture UI, use a fresh local directory, then open the returned PNG with the available image viewer.

```sh
shot_dir="$(mktemp -d "${TMPDIR:-/tmp}/iphone-use.XXXXXX")"
axe describe-ui --udid "$IPHONE_UDID"
axe screenshot --udid "$IPHONE_UDID" --output "$shot_dir/before.png"
```

The commands below are alternatives. Choose one after inspecting the target, then read the UI again. Replace sample coordinates and labels with observations from the current screen.

| Action | AXe 2 command |
| --- | --- |
| Tap a unique accessible element | `axe tap --id ContinueButton --udid "$IPHONE_UDID"` |
| Tap a unique visible label | `axe tap --label 'Done' --udid "$IPHONE_UDID"` |
| Tap a point | `axe tap -x 120 -y 280 --udid "$IPHONE_UDID"` |
| Swipe up | `axe swipe --start-x 150 --start-y 550 --end-x 150 --end-y 280 --udid "$IPHONE_UDID"` |
| Type in a focused field | `axe type 'Example' --udid "$IPHONE_UDID"` |
| Go Home | `axe button home --udid "$IPHONE_UDID"` |
| Try back navigation | `axe gesture swipe-from-left-edge --edge-inset 5 --udid "$IPHONE_UDID"` |
| Long-press | `axe tap -x 120 -y 280 --hold 0.8 --udid "$IPHONE_UDID"` |

If AXe finds multiple matches for a label, inspect the tree for a unique ID or narrow the selector with `--element-type`. If the view is in transition, read again once it settles. [AXe selectors](https://www.axe-cli.com/docs/accessibility-targeting)

AXe coordinates and accessibility frames are in logical points. PNG dimensions are in pixels. To map a screenshot point, multiply each axis by the corresponding logical display dimension divided by the original image dimension. Match orientation and capture area before mapping. If the accessibility read does not give reliable logical bounds, use a unique selector or hand off to the user on the iPhone. If the app ignores the edge swipe, inspect the screen for a visible Back or Done control. Keep each `type` call within 256 UTF-8 bytes. [AXe input documentation](https://www.axe-cli.com/docs/gestures), [AXe physical device capabilities](https://www.axe-cli.com/docs/physical-devices)

Launch an installed app with Xcode's CLI and verify the foreground screen after launch. Set `COREDEVICE_ID` to the identifier reported by `xcrun devicectl list devices` and `BUNDLE_ID` to the installed app's bundle identifier. If the CLI cannot launch the app, open it from the Home Screen. [Xcode command-line device guide](https://developer.apple.com/documentation/xcode/interacting-with-devices-using-the-command-line)

```sh
xcrun devicectl device info apps --device "$COREDEVICE_ID" --include-all-apps
xcrun devicectl device process launch --device "$COREDEVICE_ID" "$BUNDLE_ID"
```
