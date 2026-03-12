# Skip Skills

Official agent skills for building, deploying, and testing cross-platform iOS and Android apps with Swift and SwiftUI using [Skip](https://skip.dev).

## Installation

[Claude Code](https://claude.com/claude-code) is the recommended agent to use with Skip, and so the skills are optimized for Claude's conventions and models, but these skills should work with other agents as well (Gemini, Codex, etc.)

## Claude Code

Add the marketplace:

```
/plugin marketplace add skiptools/skills
```

Install a plugin:

```
/plugin install skip-app-design
/plugin install skip-testing-deployment
```

## Cursor

**Install from GitHub**

1. Open Cursor Settings (Cmd+Shift+J / Ctrl+Shift+J)
2. Navigate to `Rules & Command` > `Project Rules` > Add Rule > Remote Rule (GitHub)
3. Enter: `https://github.com/skiptools/skills.git`

**How it works:** Skills are automatically discovered and used by the agent based on context. When you ask questions about Skip development, the agent will automatically use the relevant skills based on the skill descriptions.

**Note:** Skills won't appear in the `/` slash command menu. Skills work via auto-discovery — the agent uses them automatically when your questions match their descriptions.

**Verify installation:** After adding the Remote Rule, try asking the agent Skip-related questions like:
- "Create a new Skip Lite app with a tab-based navigation"
- "Add Firebase authentication to my Skip project"
- "How do I write tests that run on both iOS and Android?"

If the skills are working, the agent will use the relevant skill content to answer your questions.

## Any agent

Point your agent at the `AGENTS.md` file or individual `SKILL.md` files in the `plugins/` directory. The skills are plain markdown and work with any agent that supports file-based context.

## License

Apache 2.0
