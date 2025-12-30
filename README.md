# Ronin Marketplace

**Battle-tested skills for shipping software.** Hard-won knowledge from real deployments, packaged for Claude Code.

[![Sponsor](https://img.shields.io/badge/Sponsor-Albert_Hui-ea4aaa?logo=github-sponsors)](https://github.com/sponsors/h4x0r)

## Installation

### Claude Code

First, add the marketplace:
```
/plugin marketplace add SecurityRonin/ronin-marketplace
```

Then install the plugins you need:
```
/plugin install deployment-skills@ronin-marketplace
/plugin install packaging-skills@ronin-marketplace
/plugin install docs-skills@ronin-marketplace
/plugin install automation-skills@ronin-marketplace
```

Or install all at once:
```
/plugin install deployment-skills@ronin-marketplace packaging-skills@ronin-marketplace docs-skills@ronin-marketplace browser-skills@ronin-marketplace
```

Verify installation:
```
/skills
```

### Claude Desktop

Add to your config file:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "ronin-marketplace": {
      "command": "npx",
      "args": ["-y", "@anthropic/claude-code-mcp", "--plugin", "https://github.com/SecurityRonin/ronin-marketplace"]
    }
  }
}
```

Restart Claude Desktop after adding.

### Updates

```
/plugin update deployment-skills@ronin-marketplace
```

## Plugins

### deployment-skills

Cloud deployment patterns for Vercel, Fly.io, and Cloudflare.

```
/plugin install deployment-skills@ronin-marketplace
```

| Skill | Use When |
|-------|----------|
| **deployment** | Choosing a deployment platform. Routes to Vercel, Fly.io, or Cloudflare based on workload type. |
| **vercel-deployment** | Deploying to Vercel. vercel.json config, env var gotchas (printf!), monorepo setup, Next.js errors, GCP WIF. |
| **fly-deployment** | Deploying to Fly.io. Single volume limit, monorepo patterns, Next.js provider errors. |
| **cloudflare-r2-d1** | Using Cloudflare D1/R2/KV. Critical limits (10GB DB, 1 write/sec/key), multi-tenant patterns. |

### packaging-skills

Cross-platform installers: DMG, MSI, DEB with GitHub Actions.

```
/plugin install packaging-skills@ronin-marketplace
```

| Skill | Use When |
|-------|----------|
| **build-cross-platform-packages** | Building installers for macOS (DMG), Windows (MSI/WiX), Linux (DEB). GitHub Actions automation, SLSA attestations, Homebrew updates. |
| **robust-dependency-installation** | Bundling portable executables (FFmpeg, Tesseract, ExifTool) in MSI installers. Unified detection/execution code paths. |

### docs-skills

Documentation pipelines: MkDocs, Pandoc, GitHub Pages.

```
/plugin install docs-skills@ronin-marketplace
```

| Skill | Use When |
|-------|----------|
| **mkdocs-github-pages-deployment** | Deploying MkDocs to GitHub Pages. Python-Markdown gotchas: 4-space indentation, footnotes, grid tables. |
| **pandoc-pdf-generation** | Generating PDFs from Markdown. Blank line rules, fix scripts, visual testing workflow. |

### automation-skills

Browser automation with Playwright, Puppeteer, and CDP.

```
/plugin install automation-skills@ronin-marketplace
```

| Skill | Use When |
|-------|----------|
| **browser-automation** | Automating browsers with Playwright/Puppeteer, testing Chrome extensions, handling dynamic content, CDP techniques, AI-powered tools. |

## Philosophy

These aren't generic tutorials. Each skill exists because something broke in production.

- **Real failures** — Problems discovered shipping software, not theory
- **Working code** — Tested across multiple projects
- **Edge cases** — The gotchas that waste hours when you don't know them
- **Automation** — Scripts and workflows to prevent manual errors

> "The best documentation is written in the blood of those who came before."

The DMG race condition fix came from 6 failed releases. The Python-Markdown rules from 469 formatting issues. The bundled dependency architecture from users reporting "dependencies not found" despite successful installation.

## Contributing

Found a gotcha we missed? PRs welcome. Follow the pattern in existing skills.

## Sponsorship

If these skills save you time, consider [sponsoring](https://github.com/sponsors/h4x0r) continued development.

## License

MIT — Use freely, ship confidently.

---

*Built by [Security Ronin](https://securityronin.com)*
