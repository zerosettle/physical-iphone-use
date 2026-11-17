# iphone-use

An agent skill for setting up and operating a physical iPhone for app QA, screenshots, and interactive UI tasks. Use a USB-paired iPhone with AXe on a Mac or an existing device-control MCP connection.

## Install

For Codex, clone the skill into your personal skills directory so it can be used from any project.

```sh
mkdir -p ~/.agents/skills
git clone https://github.com/zerosettle/iphone-use.git ~/.agents/skills/iphone-use
```

Start a new session and invoke `$iphone-use`. In Claude Code, use `~/.claude/skills/iphone-use` as the clone destination and invoke `/iphone-use`.

The [setup guide](references/setup.md) covers USB pairing, Developer Mode, and when you need an Apple Developer account. [SKILL.md](SKILL.md) describes how the agent selects and drives the phone.

[MIT-0 license](LICENSE)
