# Ronin Marketplace

**Battle-tested skills for shipping software.** Hard-won knowledge from real deployments, packaged for Claude Code.

[![Sponsor](https://img.shields.io/badge/Sponsor-Albert_Hui-ea4aaa?logo=github-sponsors)](https://github.com/sponsors/h4x0r)

```
ronin-marketplace
├── deployment-skills (4 skills)
├── packaging-skills (2 skills)
├── docs-skills (2 skills)
├── automation-skills (1 skill)
├── development-skills (1 skill)
├── data-skills (1 skill)
└── ui-skills (1 skill)
```

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
/plugin install development-skills@ronin-marketplace
/plugin install data-skills@ronin-marketplace
/plugin install ui-skills@ronin-marketplace
```

Or install all at once:
```
/plugin install deployment-skills@ronin-marketplace packaging-skills@ronin-marketplace docs-skills@ronin-marketplace automation-skills@ronin-marketplace development-skills@ronin-marketplace data-skills@ronin-marketplace ui-skills@ronin-marketplace
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
| **vercel-deployment** | Deploying to Vercel. vercel.json config, env var gotchas (printf!), monorepo setup, Next.js errors, GCP WIF, async operations (Vercel kills background tasks!). |
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
| **browser-automation** | Automating browsers with Playwright/Puppeteer, handling dynamic content, CDP techniques, AI-powered tools. |

### development-skills

Software development patterns and tooling.

```
/plugin install development-skills@ronin-marketplace
```

| Skill | Use When |
|-------|----------|
| **chrome-extension-development** | Building Chrome extensions (Manifest V3). Floating panel architecture, sidepanel API, SPA navigation detection, storage patterns, message passing, content scripts, Vitest testing, Playwright E2E. |

### data-skills

Analytics and data warehouse patterns for DuckDB, MotherDuck, and Parquet.

```
/plugin install data-skills@ronin-marketplace
```

| Skill | Use When |
|-------|----------|
| **duckdb-motherduck-parquet** | Using DuckDB/MotherDuck for analytics. Connection string auth (token in URL!), GLIBC compatibility on Vercel/Lambda, Parquet loading, BigInt serialization, CORS headers for WASM. |

### ui-skills

UI patterns for real-time updates and interactive components.

```
/plugin install ui-skills@ronin-marketplace
```

| Skill | Use When |
|-------|----------|
| **server-sent-events** | Real-time progress updates with SSE. TransformStream patterns, Next.js App Router, client consumption, Vercel timeout limits, error handling, batch processing. |

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
