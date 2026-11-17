# Build and verify on a physical iPhone

## Select the project and phone

- Start from the project's documented build and test commands. Identify the workspace or project, app scheme, configuration, target, and signing team. Use the scheme that builds the app variant the user wants to test. For a bug fix, write down the reproduction steps, expected result, and starting data so you can run the same check again.
- Follow [iPhone setup](setup.md) to pair the phone and verify screen capture and input. Match the hardware UDID used by AXe and Xcode's run destination to the identifier reported by `xcrun devicectl list devices`. Check the OS with `xcrun devicectl device info details --device "$COREDEVICE_ID"`. The iPhone's installed OS must meet the app's deployment target. [Xcode device commands](https://developer.apple.com/documentation/xcode/interacting-with-devices-using-the-command-line)
- If MCP handles input, confirm it reaches the same physical device that receives the build. Use the project's authorized install route for that phone.

## Build and locate the signed app

Use the project's configured signing team and entitlements. Diagnose a signing failure from the build log and [signing setup](setup.md#apple-account-and-signing). For a project using automatic signing, use `-allowProvisioningUpdates` when the task includes provisioning changes. Xcode can create or update signing resources in the developer account. If the iPhone also needs registration and the task covers it, add `-allowProvisioningDeviceRegistration` with that flag. [Xcode build and signing guide](https://developer.apple.com/documentation/xcode/running-your-app-on-simulated-or-physical-devices)

This Bash example uses a workspace. Set `IPHONE_UDID` to the paired phone's hardware UDID, set the workspace and scheme to the project's values, and replace `-workspace` with `-project` and the path to the `.xcodeproj` when appropriate. Check the phone appears in `-showdestinations` before building. Reuse `build_args` and its Derived Data path during the same iteration. [Xcode CLI reference](https://developer.apple.com/documentation/xcode/xcode-command-line-tool-reference)

```bash
WORKSPACE='Example.xcworkspace'
SCHEME='Example'
run_dir="$(mktemp -d "${TMPDIR:-/tmp}/iphone-use.XXXXXX")"
build_args=(
  -workspace "$WORKSPACE"
  -scheme "$SCHEME"
  -configuration Debug
  -destination "platform=iOS,id=$IPHONE_UDID"
  -derivedDataPath "$run_dir/DerivedData"
)

xcodebuild "${build_args[@]}" -showdestinations
xcodebuild "${build_args[@]}" build &&
  xcodebuild "${build_args[@]}" -showBuildSettings -json > "$run_dir/build-settings.json"
```

Continue after the build succeeds. In `build-settings.json`, find the entry for the **app target**. Combine its `TARGET_BUILD_DIR` and `FULL_PRODUCT_NAME` to get the app's full path. Check that it is the intended signed `.app` before deploying it. [Xcode build settings reference](https://developer.apple.com/documentation/xcode/build-settings-reference)

## Install and launch

Set `APP_PATH` to the just-built app's full path and `COREDEVICE_ID` to the matching phone's identifier from `xcrun devicectl list devices`. Read the bundle ID from the built app because scheme variants can use different IDs. Launch after the install succeeds and verify the foreground app with AXe or MCP. [Apple device install and launch commands](https://developer.apple.com/documentation/xcode/interacting-with-devices-using-the-command-line)

```bash
BUNDLE_ID="$(/usr/bin/plutil -extract CFBundleIdentifier raw -o - "$APP_PATH/Info.plist")" &&
  xcrun devicectl device install app --device "$COREDEVICE_ID" "$APP_PATH" &&
  xcrun devicectl device process launch --device "$COREDEVICE_ID" --terminate-existing "$BUNDLE_ID"
```

Use an in-place update for an existing install. Get authorization before uninstalling or clearing app data for a clean-install test.

## Verify and iterate

1. Check the initial screen for the expected launch state. Exercise the affected flow using [AXe or MCP](backends.md) and verify the result against the planned checks. Use safe test data for flows that create server-side state. Capture evidence that shows the outcome, such as a screenshot with the new screen or an accessibility element whose value changed.
2. If the scheme has a UI test target, run the relevant test on the same iPhone. Use the actual test target name and a fresh `.xcresult` path per run. Keep both the failure report and the rerun result when diagnosing a regression. [Apple UI automation guide](https://developer.apple.com/documentation/xcuiautomation/recording-ui-automation-for-testing)

   ```bash
   UI_TEST_TARGET='ExampleUITests'
   test_dir="$(mktemp -d "$run_dir/device-ui.XXXXXX")"
   results="$test_dir/results.xcresult"
   xcodebuild "${build_args[@]}" -only-testing:"$UI_TEST_TARGET" -resultBundlePath "$results" test
   xcrun xcresulttool get test-results summary --path "$results"
   ```

3. Use the failing build step, UI state, device log, or test report to narrow the cause. If the app quits on launch, inspect its crash report or device console in Xcode. Fix the issue, rebuild, install the new bundle, and repeat the same checks on the same phone.
4. Report the device and iOS version, the build tested, the steps that passed, and any blocked steps. Include the screenshot or test result path when useful.
