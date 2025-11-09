# Cloudflare Workers PR Preview 🚀

Automatically deploy pull request previews to Cloudflare Workers - just like Cloudflare Pages does for static sites!

[![GitHub](https://img.shields.io/github/license/mertselimb/cloudflare-workers-pr-preview)](LICENSE)

## Why?

Cloudflare Pages automatically creates preview deployments for every pull request, but this feature doesn't exist for Cloudflare Workers. This action fills that gap by automatically deploying your Workers to unique preview URLs for each PR.

## Features

- ✨ Automatic PR preview deployments
- 🧹 Automatic cleanup when PR is closed or merged
- 💬 Comments preview URL directly on your PR
- 🎨 Fully customizable build process
- 📦 Supports npm, pnpm, and yarn
- 🔧 Works with any framework (Astro, Next.js, Remix, etc.)
- 🎯 Simple setup with sensible defaults

## Quick Start

### 1. Get Your Cloudflare Credentials

#### Cloudflare API Token

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens)
2. Click "Create Token"
3. Use the "Edit Cloudflare Workers" template
4. Or create a custom token with these permissions:
   - Account > Cloudflare Workers Scripts > Edit
   - Account > Account Settings > Read
5. Copy your token

#### Cloudflare Account ID

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Select your domain/account
3. Scroll down on the right sidebar - you'll see "Account ID"
4. Copy your Account ID

### 2. Add Secrets to GitHub

1. Go to your repository settings
2. Navigate to `Secrets and variables` > `Actions`
3. Add these secrets:
   - `CF_API_TOKEN` - Your Cloudflare API Token
   - `CF_ACCOUNT_ID` - Your Cloudflare Account ID

### 3. Create Workflow

Create `.github/workflows/preview.yml`:

```yaml
name: Deploy PR Preview

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

permissions:
  contents: read
  pull-requests: write
  deployments: write

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Deploy preview on PR open/update
      - name: Deploy Preview
        if: github.event.action != 'closed'
        uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
        with:
          cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
          cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}

      # Cleanup preview on PR close/merge
      - name: Cleanup Preview
        if: github.event.action == 'closed'
        uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
        with:
          cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
          cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
          mode: cleanup
```

That's it! 🎉

**What happens:**
- When a PR is opened or updated → Deploys preview to Cloudflare Workers
- When a PR is closed or merged → Automatically deletes the worker deployment

> **Example workflow:** See [`workflow-example.yml`](workflow-example.yml) for a complete working example.

## Usage

### Basic Example

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
```

### Custom Build Command

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
    build-command: npm run build:production
    install-command: npm ci
```

### Using pnpm

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
    package-manager: pnpm
    install-command: pnpm install --frozen-lockfile
```

### Monorepo Setup

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
    working-directory: ./apps/web
    build-command: pnpm build
```

### Custom Subdomain

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
    subdomain: myapp
    # Preview URL will be: https://myapp-pr-123.yourname.workers.dev
```

### Without PR Comments

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
    comment-on-pr: false
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `cloudflare-api-token` | Cloudflare API Token with Workers permissions | ✅ Yes | - |
| `cloudflare-account-id` | Cloudflare Account ID | ✅ Yes | - |
| `mode` | Action mode: `deploy` or `cleanup` | No | `deploy` |
| `build-command` | Command to build your project | No | `npm run build` |
| `install-command` | Command to install dependencies | No | `npm install` |
| `node-version` | Node.js version to use | No | `18` |
| `working-directory` | Working directory for the project | No | `.` |
| `subdomain` | Custom subdomain prefix | No | Repository name |
| `comment-on-pr` | Whether to comment preview URL on PR | No | `true` |
| `package-manager` | Package manager (npm, pnpm, yarn) | No | `npm` |
| `wrangler-version` | Specific Wrangler version to use | No | Latest |

## Outputs

| Output | Description |
|--------|-------------|
| `preview-url` | The deployed preview URL |
| `deployment-id` | The deployment ID |

### Using Outputs

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  id: preview
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}

- name: Run E2E tests
  run: npm run test:e2e
  env:
    PREVIEW_URL: ${{ steps.preview.outputs.preview-url }}
```

## Complete Example

Here's a full-featured example with testing and automatic cleanup:

```yaml
name: PR Preview & Tests

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

permissions:
  contents: read
  pull-requests: write
  deployments: write

jobs:
  deploy-and-test:
    if: github.event.action != 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
        id: preview
        with:
          cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
          cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
          build-command: npm run build
          package-manager: npm

      - name: Run E2E tests against preview
        run: npm run test:e2e
        env:
          BASE_URL: ${{ steps.preview.outputs.preview-url }}

      - name: Run Lighthouse
        uses: treosh/lighthouse-ci-action@v1.0.1
        with:
          urls: ${{ steps.preview.outputs.preview-url }}
          uploadArtifacts: true

  cleanup:
    if: github.event.action == 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
        with:
          cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
          cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
          mode: cleanup
```

## How It Works

### Deploy Mode (default)
1. **Checkout**: Gets your code
2. **Setup Node.js**: Installs specified Node version
3. **Install Dependencies**: Runs your install command
4. **Build**: Runs your build command
5. **Deploy**: Deploys to Cloudflare Workers with name `{repo}-pr-{number}`
6. **Comment**: Posts preview URL to your PR

### Cleanup Mode
1. **Checkout**: Gets your code
2. **Delete Worker**: Removes the worker deployment from Cloudflare
3. **Comment**: Updates PR comment with cleanup notification

## Preview URL Format

The preview URL follows this pattern:
```
https://{subdomain}-pr-{pr-number}.{github-username}.workers.dev
```

For example:
- Repository: `johndoe/my-app`
- PR Number: 42
- Preview URL: `https://my-app-pr-42.johndoe.workers.dev`

## Framework-Specific Examples

### Astro

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
    build-command: npm run build
```

### Remix

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
    build-command: npm run build
```

### Hono

```yaml
- uses: mertselimb/cloudflare-workers-pr-preview@v1.0.1
  with:
    cloudflare-api-token: ${{ secrets.CF_API_TOKEN }}
    cloudflare-account-id: ${{ secrets.CF_ACCOUNT_ID }}
    build-command: npm run build
```

## Troubleshooting

### Permission Denied

Make sure your workflow has these permissions:
```yaml
permissions:
  contents: read
  pull-requests: write
  deployments: write
```

### API Token Issues

Verify your token has the correct permissions:
- Account > Cloudflare Workers Scripts > Edit
- Account > Account Settings > Read

### Build Failures

Check that:
- Your `build-command` is correct
- All dependencies are in `package.json`
- You have a valid `wrangler.toml` in your project root

### wrangler.toml Not Found

Make sure you have a `wrangler.toml` file in your working directory:

```toml
name = "my-worker"
main = "dist/index.js"
compatibility_date = "2024-01-01"
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Credits

Created by [@mertselimb](https://github.com/mertselimb)

Inspired by Cloudflare Pages automatic PR previews.

---

⭐ If this action helped you, consider giving it a star!
