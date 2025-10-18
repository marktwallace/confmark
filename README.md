# confmark

Sync Confluence ↔ Markdown. Edit, use git, push back to Confluence.

This does not support all Confluence features, but it does support Mermaid diagrams and a table of contents.

This was built for a specific workflow: I need all technical docs to be in git and in a format that is equally meaningful to human and AI readers. I use Mermaid diagrams, since they render fine in Confluence and VSCode, and AI has no trouble understanding the meaning. They say "a picture is worth a thousand words", but for tech docs, I have found a picture is generally worth a few dozen words, and Mermaid is a fine way to express it. I want tech docs to be available as prompts for downstream work like detailed design and coding and the constraints of this project work well for that. 

`test-doc.md` in this repo shows how Mermaid and ToC are converted into markdown. Mermaid is not free in Confluence, but in VSCode, bierner.markdown-mermaid will render it in markdown preview.

## Install

```bash
# 1. Install pandoc
brew install pandoc  # or: apt-get install pandoc

# 2. Download confmark (one file)
curl -O https://raw.githubusercontent.com/marktwallace/confmark/main/confmark
chmod +x confmark
```

## Configure

```bash
# Add to ~/.zshrc or ~/.bashrc
export CONFLUENCE_DOMAIN="your-domain.atlassian.net"
export CONFLUENCE_EMAIL="your.email@company.com"
export CONFLUENCE_TOKEN="your_api_token"
```

Get token: https://id.atlassian.com/manage-profile/security/api-tokens

## Use

### Basic Usage

```bash
# Pull from Confluence
./confmark pull 123456789 document.md    # to file
./confmark pull 123456789                # auto-generates filename

# Edit with your tools
vim document.md
git add document.md && git commit -m "updates"

# Push back to Confluence
./confmark push 123456789 document.md    # from file

# Verbose mode (show progress details)
./confmark -v push 123456789 document.md
```

### Unix-style Pipes

```bash
# Pull to stdout, pipe to tools
./confmark pull 123456789 | grep -i "TODO"
./confmark pull 123456789 > document.md

# Push from stdin
cat document.md | ./confmark push 123456789
echo "# Quick Update" | ./confmark push 123456789
```

Find PAGE_ID in URL: `https://.../pages/123456789/Title` → `123456789`

## Table of Contents

Just add a heading - confmark auto-converts to/from Confluence TOC macro:

```markdown
## Table of Contents
```

(The macro auto-generates the actual table of contents based on page headings)

## How It Works

**Pull**: Confluence HTML → pandoc → Markdown → Insert Mermaid from ADF

**Push**: Markdown → Convert Mermaid/TOC to macros → pandoc → HTML → Confluence

`pandoc` is non-trivial tool, and it is used to support this specific workflow. The is no general way of moving from Confluence to markdown, so this tool limits it to a set of practical conventions.

I made it one file so there is no install, sacrificing some readablity for convenience. It imports urllib rather than requests so there are no python dependencies.

## Troubleshooting

**Missing env vars**: `echo $CONFLUENCE_TOKEN` should show your token

**Version conflict (409)**: Someone edited the page, pull first

**Pandoc not found**: Install with brew/apt (see Install section)

