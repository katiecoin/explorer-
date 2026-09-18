# Deploying this Solana Explorer fork

This project explores Solana, not Base. This guide is a source-code configuration review, not proof of a successful build or deployment.

## Runtime and build

Use Node.js 22.x and pnpm 10.17.1, as specified in package.json. The older versions in CONTRIBUTING.md are stale.

Run in a clean checkout with network access to the dependency registries:

```bash
corepack enable
corepack prepare pnpm@10.17.1 --activate
pnpm install --frozen-lockfile
pnpm types
pnpm lint
pnpm test:ci
pnpm build
pnpm start
```

Do not bypass failures. package.json and pnpm-lock.yaml currently contain a local dependency named `postcss@3.4.17` pointing to `link:tailwindcss/postcss@3.4.17`, but that target directory is absent from the repository tree. Investigate this entry during clean-install validation; if accidental, remove it from package.json and regenerate the lockfile using the pinned pnpm version, then rerun the checks. This guide does not change dependencies.

## Environment settings

For local work, copy the template into an untracked .env.local file and fill only the settings used. The existing .gitignore excludes .env and .env*.local. For hosting, set variables in the hosting provider's environment settings rather than committing values.

| Variable | Purpose |
| --- | --- |
| NEXT_PUBLIC_MAINNET_RPC_URL | Browser mainnet Solana RPC endpoint |
| MAINNET_RPC_URL | Server mainnet Solana RPC endpoint |
| NEXT_PUBLIC_DEVNET_RPC_URL | Browser devnet Solana RPC endpoint |
| DEVNET_RPC_URL | Server devnet Solana RPC endpoint |
| NEXT_PUBLIC_TESTNET_RPC_URL | Browser testnet Solana RPC endpoint |
| TESTNET_RPC_URL | Server testnet Solana RPC endpoint |
| JUPITER_API_KEY | Server-only key required by the Jupiter verification route |
| RUGCHECK_API_KEY | Server-only key required by the premium RugCheck verification route |
| COINGECKO_API_KEY | Optional server-only CoinGecko Pro key; omit to use the public API fallback |
| SENTRY_DSN | Optional error-reporting DSN |
| SENTRY_ORG, SENTRY_PRJ, SENTRY_AUTH_TOKEN | Optional source-map upload settings |

The server RPC settings above are read by app/utils/cluster.ts but are absent from .env.example. Set both browser and server URLs explicitly for each network you support. Otherwise the code rewrites default RPC hosts to explorer-api hosts outside localhost, which may not be appropriate for this fork.

NEXT_PUBLIC variables are embedded into browser code at build time. Never put a private API key in them. Browser RPC URLs must be safe for public exposure; use provider-supported origin restrictions and quotas, or implement a secured server-side proxy before using private RPC credentials.

Without Jupiter or RugCheck keys, the corresponding verification routes return configuration errors. That does not alone establish that the whole application cannot build. CoinGecko uses the Pro API whenever COINGECKO_API_KEY is present: do not supply a Demo API key to this variable without adapting the route to the Demo API.

## Hosting configuration

Choose a Next.js-capable host with server/API-route support; this is not configured as a static GitHub Pages export.

- Repository: katiecoin/explorer-
- Branch: master (after reviewed changes are merged)
- Root directory: repository root
- Node.js: 22.x
- Install command: pnpm install --frozen-lockfile
- Build command: pnpm build
- For a traditional Node host, start command: pnpm start
- For a managed Next.js host, keep its default output handling; production output is .next

Do not create a hosting account, incur charges, or publish the site until the hosting destination and settings have been chosen.

## Launch checks

1. Resolve clean-install and build failures, including the local dependency reference above.
2. Configure your own safe browser RPC endpoints and server RPC endpoints.
3. Open the home page, an account, and a known transaction on each supported network.
4. Check verification API responses for valid and invalid mint addresses, with and without optional credentials.
5. Confirm private keys are absent from browser bundles, responses, logs, and git changes.
6. Review API rate limits and abuse protection before public launch. BotID integration exists in next.config.mjs; verify its behavior on the selected host.
7. Review inherited branding, metadata, canonical URLs, and sitemap URLs so the fork does not imply it is the official explorer.

No provider API keys have been generated or installed by this guide.
