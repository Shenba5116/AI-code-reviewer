# PR Code Reviewer - API Endpoints Summary

## Overview
This document describes all API endpoints used by the PR Code Reviewer VS Code extension, including AI LLM, GitHub, and Webhook endpoints.

---

## 1. AI Review Service (Llama 3.3 via Groq)

### File Location
📄 **[src/aiReviewService.ts](src/aiReviewService.ts)**

### LLM Details
- **Provider**: Groq (Ultra-fast LLM Inference)
- **Model**: Llama 3.3 70B Versatile
- **Model ID**: `llama-3.3-70b-versatile`

### API Endpoint
```
POST https://api.groq.com/openai/v1/chat/completions
```

### Authentication
- **Type**: Bearer Token
- **Header**: `Authorization: Bearer {API_KEY}`

### Request Format
```json
{
  "model": "llama-3.3-70b-versatile",
  "messages": [
    {
      "role": "system",
      "content": "You are a code review assistant..."
    },
    {
      "role": "user",
      "content": "{PR_DIFF_PROMPT}"
    }
  ],
  "temperature": 0.15,
  "top_p": 0.8,
  "max_tokens": 8192,
  "stream": false
}
```

### Key Methods
| Method | Location | Purpose |
|--------|----------|---------|
| `reviewPullRequest()` | [Line 60](src/aiReviewService.ts#L60) | Main entry point - reviews entire PR |
| `callGroqWithRetry()` | [Line 221](src/aiReviewService.ts#L221) | Calls Groq API with retry logic (3 attempts) |
| `callGroq()` | [Line 253](src/aiReviewService.ts#L253) | Direct Groq API call via HTTPS |
| `buildReviewPrompt()` | [Line 161](src/aiReviewService.ts#L161) | Constructs review prompt from PR files |

### Features
- Batch processing (max 25,000 chars per batch)
- Rate limiting: 30 req/min, 1000 req/day (free tier)
- 90-second timeout per request
- Automatic retry with exponential backoff (10s, 20s delays)

### Response Handling
- Parses JSON array of review issues
- Returns structured `ReviewResult` with issues, file results, and summary

---

## 2. GitHub API Service

### File Location
📄 **[src/githubService.ts](src/githubService.ts)**

### API Endpoint
```
https://api.github.com
```

### Authentication
- **Type**: Octokit (GitHub REST API client library)
- **Auth Method**: Personal Access Token

### Core API Calls Made

| Endpoint | Method | Purpose | Location |
|----------|--------|---------|----------|
| `/user` | GET | Verify token & connectivity | [Line 30](src/githubService.ts#L30) |
| `/repos/{owner}/{repo}/pulls` | GET | List open PRs | [Line 40](src/githubService.ts#L40) |
| `/repos/{owner}/{repo}/pulls/{pr_number}` | GET | Get specific PR details | [Line 54](src/githubService.ts#L54) |
| `/repos/{owner}/{repo}/pulls/{pr_number}/files` | GET | Get PR file changes | [Line 65](src/githubService.ts#L65) |
| `/repos/{owner}/{repo}/contents/{path}` | GET | Get file content at reference | [Line 89](src/githubService.ts#L89) |
| `/repos/{owner}/{repo}/compare/{base}...{head}` | GET | Compare branches | [Line 107](src/githubService.ts#L107) |
| `/repos/{owner}/{repo}/pulls/{pr_number}/reviews` | POST | Create review (approve/request changes/comment) | [Line 140](src/githubService.ts#L140) |
| `/repos/{owner}/{repo}/pulls/{pr_number}/comments` | POST | Post inline review comment | [Line 123](src/githubService.ts#L123) |
| `/repos/{owner}/{repo}/issues/{pr_number}/comments` | POST | Post PR comment (general) | [Line 152](src/githubService.ts#L152) |
| `/repos/{owner}/{repo}/statuses/{sha}` | POST | Create commit status check | [Line 161](src/githubService.ts#L161) |
| `/repos/{owner}/{repo}/pulls/{pr_number}/commits` | GET | Get PR commits | [Line 175](src/githubService.ts#L175) |
| `/repos/{owner}/{repo}/branches/{branch}/protection` | GET | Get branch protection rules | [Line 187](src/githubService.ts#L187) |

### Key Methods
| Method | Purpose |
|--------|---------|
| `verifyConnection()` | Validate token validity |
| `listOpenPullRequests()` | Fetch all open PRs (up to 30) |
| `getPullRequest()` | Get single PR by number |
| `getPullRequestFiles()` | Get all files changed in PR (paginated) |
| `getFileContent()` | Get file content from specific branch |
| `compareBranches()` | Get diff between branches |
| `createReviewComment()` | Post inline code comment |
| `submitReview()` | Submit PR review decision |
| `postPRComment()` | Post general PR comment |
| `createCommitStatus()` | Set commit status (pending/success/failure/error) |

---

## 3. Webhook Server (Local)

### File Location
📄 **[src/webhookServer.ts](src/webhookServer.ts)**

### Local Server Endpoint
```
HTTP POST http://localhost:{port}/webhook
```

### Configuration
- **Port**: Configurable via settings (default typically 3000 or configurable)
- **Protocol**: HTTP (local)
- **Path**: `/webhook` or `/` (both accepted)

### GitHub Webhook Configuration
To receive events, configure in GitHub repository settings:
```
Payload URL: http://localhost:{port}/webhook
Content type: application/json
Secret: {configured_secret}
Events: Pull Request
```

### Webhook Handler Details

| Endpoint | Method | Purpose | Location |
|----------|--------|---------|----------|
| `/webhook` | POST | Main webhook receiver | [Line 80](src/webhookServer.ts#L80) |
| `/health` | POST | Health check endpoint | [Line 73](src/webhookServer.ts#L73) |

### Supported PR Events
- `opened` - New PR created
- `synchronize` - PR updated with new commits
- `reopened` - PR reopened
- `ready_for_review` - Draft PR marked ready

### Security Features
- **Signature Verification**: HMAC SHA-256 validation
- **Header**: `X-Hub-Signature-256`
- **Event Type Filter**: Only `pull_request` events processed

### Key Methods
| Method | Purpose | Location |
|--------|---------|----------|
| `start()` | Start webhook server | [Line 26](src/webhookServer.ts#L26) |
| `stop()` | Stop webhook server | [Line 52](src/webhookServer.ts#L52) |
| `handleRequest()` | Request handler | [Line 68](src/webhookServer.ts#L68) |
| `verifySignature()` | Validate GitHub signature | [Line 167](src/webhookServer.ts#L167) |

---

## API Flow Summary

```
┌─────────────────────────────────────────────────────────────┐
│                   PR Code Reviewer Extension                │
└─────────────────────────────────────────────────────────────┘
           │                          │                    │
           ▼                          ▼                    ▼
    ┌─────────────┐          ┌──────────────┐      ┌──────────────┐
    │  Groq API   │          │ GitHub API   │      │    Webhook   │
    └─────────────┘          └──────────────┘      └──────────────┘
    
    POST /openai/v1/        REST API calls:         HTTP POST
    chat/completions        - List PRs              /webhook
                             - Get PR files
    llama-3.3-70b          - Create reviews        Listens for
    versatile              - Post comments         PR events
                             - Set commit status
```

---

## Summary Table

| Service | Endpoint | Type | File | Auth |
|---------|----------|------|------|------|
| **Groq (Llama 3.3)** | `api.groq.com/openai/v1/chat/completions` | HTTPS POST | [aiReviewService.ts](src/aiReviewService.ts) | Bearer Token |
| **GitHub API** | `api.github.com` | REST API | [githubService.ts](src/githubService.ts) | Personal Access Token |
| **Webhook** | `localhost:{port}/webhook` | HTTP POST | [webhookServer.ts](src/webhookServer.ts) | HMAC SHA-256 |

---

## Configuration References

- **GitHub Token**: Stored securely via VS Code secrets
- **Groq API Key**: Required for AI review functionality
- **Webhook Port**: Configurable (default: 3000)
- **Webhook Secret**: Optional, for signature verification

For setup instructions, see [README.md](README.md)
