## Android Skills for Claude Code

This is a fork of [android/skills](https://github.com/android/skills) converted to **Claude Code** custom slash commands.

The original skills target the Android CLI (Gemini / Antigravity agent). This repo re-writes each skill as a Claude Code command — same technical content, adapted to Claude's tool-use model (`Read`, `Edit`, `Glob`, `Grep`, etc.).

---

## Available Skills (Claude Code Commands)

| Command | Description |
|---|---|
| `/android-agp-9-upgrade` | Migrate an Android project to AGP 9 |
| `/android-compose-xml-migration` | Incrementally migrate one XML layout to Jetpack Compose |
| `/android-navigation-3` | Set up or migrate to Jetpack Navigation 3 |
| `/android-r8-analyzer` | Audit R8/ProGuard keep rules and recommend size optimisations |
| `/android-play-billing-upgrade` | Upgrade Google Play Billing Library to the latest version |
| `/android-edge-to-edge` | Add adaptive edge-to-edge and IME inset support to Compose screens |

---

## Installation

Copy the `.claude/commands/` folder into your Android project root (or your global `~/.claude/commands/`):

```bash
# Project-level (recommended)
cp -r .claude/commands/ /path/to/your/android/project/.claude/commands/

# Global (available in all projects)
cp -r .claude/commands/ ~/.claude/commands/
```

Then in any Claude Code session within that project, invoke a skill with:

```
/android-agp-9-upgrade
/android-edge-to-edge
/android-r8-analyzer
```

Some commands accept an optional argument:

```
/android-compose-xml-migration app/src/main/res/layout/fragment_home.xml
/android-navigation-3 bottom-nav
/android-play-billing-upgrade 7.0.0
```

---

## Source

Original skills: [android/skills](https://github.com/android/skills) — Apache License 2.0

Claude Code conversion maintained in this fork. The original `SKILL.md` files for each topic remain in their original directories for reference.

## Disclaimer

AI can make mistakes. Always review and test changes before committing to production.

## License

Android Skills is licensed under the [Apache License 2.0](LICENSE.txt).
