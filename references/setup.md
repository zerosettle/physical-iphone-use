# iPhone setup

Run the local tools on the Mac paired with the physical iPhone. Check the [minimum iOS version](../SKILL.md#minimum-ios-version) and make sure the Mac and phone are supported by your Xcode version. For an iPhone already connected to your agent through MCP, use the [connector setup](backends.md#existing-mcp-connector). [Xcode compatibility matrix](https://developer.apple.com/xcode/system-requirements)

## Apple Account and signing

| Task | Account needed for the workflow |
| --- | --- |
| Inspect or interact with a paired, unlocked iPhone through AXe | Xcode device pairing and Developer Mode. No developer membership for AXe itself. |
| Install a development build or run Xcode UI tests on your iPhone | An Apple Account in Xcode. Use your free Personal Team for personal testing, or the project’s existing team if its signing or entitlements require it. |
| Distribute your own build through TestFlight or the App Store | Membership in the Apple Developer Program or access to the project’s enrolled team. |

For a free Personal Team, use an existing Apple Account or create one with Apple. Open **Xcode > Settings > Apple Accounts** (**Accounts** in some versions) and add the account. Apple limits a Personal Team to 10 active App IDs, 3 registered devices, and 3 installed apps per device. The provisioning profile expires seven days after issuance. Rebuild and reinstall to renew it. Enter passwords and verification codes yourself in Apple’s prompts. [Create an Apple Account](https://support.apple.com/en-us/108647) · [Personal Team limits](https://developer.apple.com/help/account/basics/about-your-developer-account)

If you need a paid team and your project already has one, ask its administrator for access. For your own team, begin enrollment with your Apple Account and complete Apple’s identity verification, agreement, and payment steps. Apple lists the program at US$99 per membership year, with local pricing shown during enrollment. After enrollment, add that account in Xcode and select the enrolled team when signing your build. [Apple Developer Program enrollment](https://developer.apple.com/programs/enroll/)

## Install Xcode on the Mac

1. Download full Xcode from Apple. Check its supported macOS versions and its device support for the iOS version already installed on your iPhone. [Get Xcode](https://developer.apple.com/xcode/) · [Xcode compatibility matrix](https://developer.apple.com/xcode/system-requirements)
2. Open Xcode once and complete its license and initial component prompts. In **Xcode > Settings > Locations**, select that Xcode release under **Command Line Tools**. For a one-shell override, set `DEVELOPER_DIR` to the desired Xcode app’s `Contents/Developer` directory. [Xcode command-line tool reference](https://developer.apple.com/documentation/xcode/xcode-command-line-tool-reference)
3. Verify the selected toolchain in Terminal. If the path points to `/Library/Developer/CommandLineTools` or the tools cannot be found, select full Xcode in step 2. [Xcode command-line tool reference](https://developer.apple.com/documentation/xcode/xcode-command-line-tool-reference)

```sh
xcode-select --print-path
xcodebuild -version
xcrun --find devicectl
```

## Pair a physical iPhone for developer tools

1. Unlock the iPhone and connect it using a USB data cable. In Xcode 26, open the device manager from Xcode’s **Window** menu and select its **Devices** tab. In Xcode 27, select the phone in Device Hub and click **Pair** if prompted. Confirm **Trust This Computer** on the iPhone and enter its passcode on the device. [Device pairing in Xcode](https://developer.apple.com/documentation/xcode/managing-your-simulated-and-physical-devices-in-device-hub)
2. Open **Settings > Privacy & Security > Developer Mode** on the iPhone. Turn it on, restart when prompted, then unlock the iPhone and confirm **Turn On**. If the setting is missing, initiate pairing from Xcode first. [Enable Developer Mode](https://developer.apple.com/documentation/xcode/enabling-developer-mode-on-a-device)
3. Keep the phone connected and unlocked while Xcode prepares it. Use `xcrun devicectl list devices` to check that Xcode sees the intended device. For AXe, use the hardware UDID from `axe list-devices` or the Xcode device inspector. The CoreDevice identifier and hardware UDID can differ. [Device identifiers for `devicectl`](https://developer.apple.com/documentation/xcode/interacting-with-devices-using-the-command-line) · [AXe physical device setup](https://www.axe-cli.com/docs/physical-devices)

If a development build is part of the task, open the project, select its app target, and set the team under **Signing & Capabilities**. Keep the existing project’s bundle identifier and signing settings when they work. For a new local project, use a unique bundle identifier and enable **Automatically manage signing**. Select the paired iPhone as the run destination and choose **Product > Run**. Follow Xcode’s guidance if it requests device registration. For UI tests, use the project’s UI test target and **Product > Test**. [Run and sign a development app](https://developer.apple.com/documentation/xcode/running-your-app-on-simulated-or-physical-devices) · [Record and run UI automation](https://developer.apple.com/documentation/xcuiautomation/recording-ui-automation-for-testing)

If you already have a device `.app` signed and provisioned for that iPhone, you can install it without reopening the project. Use the identifier from `xcrun devicectl list devices`. [Install and launch from the command line](https://developer.apple.com/documentation/xcode/interacting-with-devices-using-the-command-line)

```sh
COREDEVICE_ID='replace-with-the-identifier-from-devicectl'
xcrun devicectl device install app --device "$COREDEVICE_ID" ./Example.app
```

## Install AXe for command-line interaction

Install AXe if you want to inspect accessibility elements or send input from the command line. Its 2.x documentation covers Xcode 26 and 27. Use an AXe release that supports your target, and check the installed command help because features vary by release. Install Homebrew first if `brew` is missing. [Homebrew installation](https://brew.sh/), [AXe installation](https://www.axe-cli.com/docs/installation)

```sh
brew install cameroncooke/axe/axe
axe --help
```

If `axe --help` does not list `list-devices` and `doctor`, update to a compatible AXe release with physical-device support. Set `IPHONE_UDID` to the hardware UDID shown for the paired phone. [AXe releases](https://github.com/cameroncooke/AXe/releases) · [AXe physical device setup](https://www.axe-cli.com/docs/physical-devices)

```sh
axe list-devices
IPHONE_UDID='replace-with-the-iphone-hardware-udid'
axe doctor --udid "$IPHONE_UDID"
axe screenshot --udid "$IPHONE_UDID" --output iphone.png
```

`doctor` checks connection, trust, Developer Mode, the developer disk image, lock state, and access to the screenshot service. If the image or service is missing, let Xcode finish preparing the device. If it is still missing, check whether the selected Xcode version provides the required device service. Unlock a locked device on the phone. After capturing a screenshot, use the [AXe commands](backends.md#local-axe) to try a reversible input, such as opening an app from the Home Screen, and read the resulting screen. AXe's upstream release notes document physical input testing on iOS 27.2. [AXe readiness checks](https://www.axe-cli.com/docs/physical-devices) · [AXe physical test evidence](https://github.com/cameroncooke/AXe/blob/v2.0.4/docs/release-readiness.md#L466-L478)
