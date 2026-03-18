---
nav_exclude: true
search_exclude: true
---

# Local Development Pipeline Options for JTD-RMW

## The Core Problem

JTD-RMW is a **theme**, not a site. It has layouts, includes, and assets but no content pages. To test it, you need a **consuming site** (like `RMWinslow.github.io` or `posts`) that references the theme, pointed at your local checkout instead of the remote GitHub copy.

There is no way to preview a Jekyll site without some kind of Ruby environment — Jekyll's Liquid templating, frontmatter processing, and layout inheritance are Ruby-specific. The only alternatives are pushing to GitHub and waiting 30–90 seconds per change (too slow for development) or porting the entire theme to a different static site generator (not practical).

## Where to Stage the Dev Environment

**Option A: Add dev config to an existing consuming site.**
Add a `_config_dev.yml` to (say) `RMWinslow.github.io` that overrides `remote_theme` to point to the local JTD-RMW checkout. No new repo needed.

```yaml
# _config_dev.yml (in the consuming site, git-ignored)
remote_theme: ../JTD-RMW
url: http://localhost:4000
baseurl: ""
```

Serve with:
```bash
bundle exec jekyll serve --config "_config.yml,_config_dev.yml" --livereload
```

**Option B: Create a minimal test site in this repo.**
Add a `test/` or `_test_site/` directory with a handful of pages exercising each layout and frontmatter option. The theme repo's `_config.yml` already exists and could be adapted. Keeps everything self-contained. Downside: you'd be maintaining test content that duplicates what the real sites already exercise.

**Option C: Create a separate `jtd-rmw-testbed` repo.**
A dedicated repo with pages that stress-test the theme. Can be deployed to GitHub Pages as its own site for visual comparison. Cleanest separation but adds another repo to maintain.

**Recommendation:** Option A is simplest. You already have five consuming sites checked out locally. A one-line `_config_dev.yml` (git-ignored) is all that's needed. If you later want a dedicated test page for things like the disclosure triangle CSS experiments, a single `test.html` in Option B is fine without a whole separate repo.

---

## Getting Ruby/Jekyll Running

### Option 1: Native Windows Ruby

Install RubyInstaller+DevKit from rubyinstaller.org, then `gem install jekyll bundler`.

| Pros | Cons |
|------|------|
| Direct, no virtualization layer | Windows is not officially supported by Jekyll |
| Fastest file access | Needs extra gems: `tzinfo-data`, `wdm` for file watching |
| Nothing else to install | MSYS2/DevKit can be finicky with native extensions |
| | Encoding issues (UTF-8 BOM, console codepage) |

### Option 2: WSL2 + Native Ruby

Enable WSL2, install Ubuntu, install Ruby inside Linux. Jekyll runs on its native platform.

| Pros | Cons |
|------|------|
| All Windows-specific gem issues vanish | Files must live on the Linux filesystem for good performance |
| Jekyll's officially supported platform | Cross-filesystem `--watch` doesn't work (needs `--force_polling`) |
| Order-of-magnitude faster builds than native Windows | Workflow change: repos under `~/projects/` not `/mnt/c/...` |
| | May need a manual WSL kernel update on your Windows 10 build |

### Option 3: Docker (`starefossen/github-pages`)

A single `docker run` command, no Ruby installation at all.

```bash
docker run --rm -v "$PWD":/usr/src/app -p 4000:4000 starefossen/github-pages
```

| Pros | Cons |
|------|------|
| Zero local Ruby infrastructure | The official `jekyll/jekyll` Docker image is abandoned (3+ years stale) |
| Reproducible environment | Community images (`starefossen`) may also go stale |
| Works on any OS with Docker | Docker Desktop has resource overhead |
| | Volume mount performance on Windows can be slow |
| | Debugging gem issues inside a container is harder |

### Option 4: Dev Container (VS Code + Docker)

Microsoft maintains a Jekyll dev container image at `mcr.microsoft.com/devcontainers/jekyll`. Add a single JSON file to a repo and open it in VS Code.

```json
{
  "image": "mcr.microsoft.com/devcontainers/jekyll:2-bookworm",
  "postCreateCommand": "bundle install",
  "forwardPorts": [4000]
}
```

| Pros | Cons |
|------|------|
| Zero local Ruby setup | Requires Docker Desktop |
| Microsoft-maintained image (actively updated, unlike Docker Hub's) | First container build takes a few minutes |
| Includes `github-pages` gem for build parity | Editing files outside VS Code is awkward |
| Works identically locally and in GitHub Codespaces | `--livereload` doesn't work in Codespaces |
| Single JSON file to set up | |

### Option 5: GitHub Codespaces (Cloud)

Open the repo in a browser-based VS Code instance with Ruby pre-installed. Uses the same dev container config as Option 4.

| Pros | Cons |
|------|------|
| Nothing to install locally | Free tier: 120 core-hours/month |
| Works from any machine | Network latency for file operations |
| Pre-configured Ruby environment | `--livereload` doesn't work |
| | Slower feedback loop than local |

### Recommendation

**For simplicity: Option 3 (Docker one-liner)** if you already have Docker Desktop. Zero setup, zero maintenance.

**For best developer experience: Option 2 (WSL2)** if you're willing to keep repos on the Linux filesystem. Native-speed Jekyll with no Windows quirks.

**For lowest friction with VS Code: Option 4 (Dev Container).** One JSON file, Microsoft maintains the image, includes the `github-pages` gem.

**Avoid: Option 1 (Native Windows Ruby)** unless you specifically want to. The accumulation of Windows workarounds is not worth it when WSL2 and Docker exist.

---

## Automated Visual Inspection

### Claude Code Chrome Integration (Simplest)

Claude Code can control a real Chrome browser directly.

```bash
claude --chrome
```

- Opens real Chrome tabs, shares your login state
- Can navigate to `localhost:4000`, take screenshots, read console errors
- Can inspect DOM, fill forms, verify layout
- **Requires:** Chrome or Edge + the "Claude in Chrome" extension (v1.0.36+)
- **Limitation:** Beta feature. Not available through third-party API providers.

**Best for:** Quick visual checks during development. "Navigate to localhost:4000, does the nav look right? Any console errors?"

### Playwright MCP (Most Powerful)

Full browser automation via Microsoft's Playwright, connected as an MCP server.

```bash
claude mcp add playwright -- cmd /c npx -y @playwright/mcp@latest --headless
```

(Note: Windows needs the `cmd /c` wrapper around `npx`.)

- Full Chromium/Firefox/WebKit automation
- Screenshots, DOM inspection, accessibility tree
- Can generate reusable TypeScript test scripts
- **Limitation:** Token-heavy (~114,000 tokens per page interaction). Complex pages can exceed context limits.

**Best for:** Systematic visual regression testing. "Check every layout on mobile and desktop widths. Compare screenshots before and after the CSS change."

### Claude Code's Native Image Reading (Verification Only)

Claude Code can read PNG/JPG files natively via the Read tool. If you take a screenshot (manually or via Playwright), Claude can analyze it visually.

**Best for:** Spot-checking. "Here's a screenshot of the nav on Safari — does the disclosure triangle look right?"

### Recommendation

**Start with Chrome integration.** It's the simplest path: install the Chrome extension, run `claude --chrome`, and ask Claude to check `localhost:4000`. No Playwright setup, no token-heavy page scraping. If you later need systematic cross-browser testing or visual regression checks, add Playwright.

---

## Build Parity with GitHub Pages

GitHub Pages uses the `github-pages` gem (currently v232), which pins Jekyll 3.10.0 and specific versions of all plugins. Your theme targets this environment. For local builds to match GitHub's output exactly, the consuming site's Gemfile should use:

```ruby
source "https://rubygems.org"
gem "github-pages", "~> 232", group: :jekyll_plugins
```

The theme repo's own Gemfile uses `gemspec` (which pulls in `jekyll >= 3.8.5`). This is fine for the theme repo itself — the `github-pages` gem belongs in the consuming site's Gemfile, not the theme's.

---

## Proposed Minimal Setup

The simplest path from where you are today:

1. **Install Docker Desktop** (if not already installed).
2. **In a consuming site** (e.g., `RMWinslow.github.io`), create a git-ignored `_config_dev.yml`:
   ```yaml
   remote_theme: ../JTD-RMW
   url: http://localhost:4000
   baseurl: ""
   ```
3. **Serve with Docker:**
   ```bash
   docker run --rm -v "$PWD":/usr/src/app -p 4000:4000 starefossen/github-pages
   ```
   Or with a Gemfile + native/WSL Ruby:
   ```bash
   bundle exec jekyll serve --config "_config.yml,_config_dev.yml" --livereload
   ```
4. **Install the Chrome extension** and run `claude --chrome` for visual inspection.
5. Edit theme files in JTD-RMW. The consuming site rebuilds and the browser refreshes.

This gets you a working local dev loop with automated visual inspection, no Ruby installation on your Windows host, and minimal configuration.

---

## What We Actually Set Up (2026-03-18)

After evaluating the options above, here is what we ended up installing and how the workflow actually works.

**Ruby and Jekyll** are installed inside WSL2 Ubuntu. A shared `Gemfile` in the parent `Documents/GitHub/` directory uses the `github-pages` gem (v232), which pins Jekyll to the exact version that GitHub Pages uses. The gems are installed into a local `.bundles_cache/` folder in that same directory. A shared `_config_dev.yml` overlay sits alongside the Gemfile and sets `url: http://localhost:4000`.

**To serve a consuming site locally,** open a WSL terminal and run:

```bash
cd /mnt/c/Users/RMW/Documents/GitHub/circe
bundle exec jekyll serve --config _config.yaml,../_config_dev.yml
```

The site will be available at `http://localhost:4000` in any Windows browser. Each consuming site needs `jekyll-remote-theme` listed in its `plugins:` config — GitHub Pages injects this automatically, but local Jekyll does not. Adding it explicitly is harmless on GitHub's side.

**Playwright** is installed as a Python package (`pip install playwright`) with a standalone Chromium binary cached in `AppData\Local\ms-playwright\`. It can take headless screenshots of any URL:

```bash
python -m playwright screenshot --full-page http://localhost:4000 screenshot.png
```

It also supports `--device "iPhone 11"` for mobile viewports and `--color-scheme dark` for dark mode testing.

**The Claude Code agent's role** in this pipeline is limited by the fact that it cannot start or restart the Jekyll server. WSL commands that require a password or that run as long-lived processes do not work from Claude's non-interactive shell. So the workflow is:

1. Claude edits theme or site files.
2. The user starts or restarts the Jekyll server in their own WSL terminal.
3. Claude takes a screenshot of `localhost:4000` via Playwright and reads the image to check the results.

For standalone HTML/CSS prototypes that don't involve Jekyll at all (like a test page for CSS experiments), Claude can serve them independently using Python's built-in `python -m http.server` on the Windows side, screenshot them with Playwright, and iterate without needing the user's help.

We decided against the Chrome integration approach because it shares the user's real browser session, which means Claude could accidentally interact with logged-in sites. The Playwright approach is isolated and read-only — Claude can look but cannot click or type.
