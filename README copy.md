# PR Code Reviewer — VS Code Extension

Automated code review system that triggers on GitHub Pull Requests via webhooks and APIs. Reviews feature branches, shows results inline in VS Code, and prompts re-checking before merging.

## Features

- **Webhook Listener** — Local HTTP server receives GitHub PR events in real-time
- **Automated Code Review** — Triggered automatically when a PR is opened or updated
- **Rich Review Panel** — WebView-based detailed report with file-by-file breakdown
- **Tree Views** — Sidebar views for PRs, review results, and webhook status
- **GitHub Integration** — Posts review comments and commit statuses on PRs
- **Re-check Before Merge** — Prompts to re-verify the feature branch before merging
- **Configurable Checks** — Toggle individual review rules (security, debug statements, complexity, etc.)

## Review Checks

| Check | Description |
|-------|-------------|
| Debug Statements | `console.log`, `print()`, `debugger`, etc. |
| Security Patterns | Hardcoded secrets, `eval()`, SQL injection, XSS |
| TODO/FIXME | Unresolved TODO and FIXME comments |
| Merge Conflicts | Leftover conflict markers |
| Code Complexity | Deep nesting, long functions |
| Naming Conventions | Single-letter variables, incorrect casing |
| Large Files | Files with excessive changes |
| Best Practices | Empty catch blocks, magic numbers, `any` type |

## Setup

### 1. Install Dependencies

```bash
cd pr-code-reviewer
npm install
```

### 2. Compile

```bash
npm run compile
```

### 3. Configure GitHub Token

1. Create a [GitHub Personal Access Token](https://github.com/settings/tokens) with `repo` scope
2. Run command: **PR Reviewer: Configure GitHub Token**
3. Paste your token

### 4. Set Up Webhook (for real-time PR events)

1. Run command: **PR Reviewer: Start Webhook Server**
2. Expose local port (default `7890`) via [ngrok](https://ngrok.com/) or similar:
   ```bash
   ngrok http 7890
   ```
3. In your GitHub repo → Settings → Webhooks → Add webhook:
   - **Payload URL**: `https://your-ngrok-url.ngrok.io/webhook`
   - **Content type**: `application/json`
   - **Secret**: (match your `prCodeReviewer.webhookSecret` setting)
   - **Events**: Select "Pull requests"

### 5. Manual Review (without webhook)

1. Run command: **PR Reviewer: List Open Pull Requests**
2. Click any PR in the sidebar to trigger a review
3. Or run: **PR Reviewer: Review Current Pull Request** and enter a PR number

## Commands

| Command | Description |
|---------|-------------|
| `PR Reviewer: Start Webhook Server` | Start local webhook listener |
| `PR Reviewer: Stop Webhook Server` | Stop the webhook listener |
| `PR Reviewer: List Open Pull Requests` | Fetch and display open PRs |
| `PR Reviewer: Review Current Pull Request` | Run review on a specific PR |
| `PR Reviewer: Re-check Feature Branch` | Re-run review on last reviewed PR |
| `PR Reviewer: Configure GitHub Token` | Set GitHub PAT |
| `PR Reviewer: Open Review Panel` | Show last review results |

## Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `prCodeReviewer.githubToken` | `""` | GitHub PAT |
| `prCodeReviewer.webhookPort` | `7890` | Webhook server port |
| `prCodeReviewer.webhookSecret` | `""` | Webhook HMAC secret |
| `prCodeReviewer.autoReviewOnPR` | `true` | Auto-review on PR webhook |
| `prCodeReviewer.owner` | `""` | GitHub repo owner |
| `prCodeReviewer.repo` | `""` | GitHub repo name |
| `prCodeReviewer.reviewChecks` | `{...}` | Toggle individual checks |
| `prCodeReviewer.maxFileSizeKB` | `500` | Large file threshold |

## Development

```bash
# Watch mode for development
npm run watch

# Press F5 in VS Code to launch Extension Development Host
```

## Architecture

```
src/
├── extension.ts      # Extension entry point, command registration
├── types.ts          # TypeScript type definitions
├── config.ts         # Configuration reader, repo detection
├── githubService.ts  # GitHub REST API client (Octokit)
├── webhookServer.ts  # Local HTTP webhook listener
├── reviewEngine.ts   # Code review analysis engine
├── reviewPanel.ts    # WebView panel for review reports
└── treeViews.ts      # Sidebar tree view providers
```

## License

MIT
