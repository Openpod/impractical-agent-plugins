# Impractical Claude plugins

Public Claude Code marketplace for `impractical-motion@impractical`.

## First video

Paste this into Claude Code:

> Set up Impractical Motion and make my first launch video for: **[product and value proposition]**. Run `npx --yes @impractical-ai/motion setup --claude`, stay with me through browser signup and device approval, then tell me when to run `/reload-plugins`. After I say “continue,” use Impractical’s MCP tools to create the video, inspect its frames, fix visible problems, export it, and give me the studio link.

The plugin bundles the workflow skill and starts the MCP server through the public npm package. Motion-library capabilities, templates, references, engine types, and personalized guidance remain live MCP data, so a plugin update is not required when the library changes.
