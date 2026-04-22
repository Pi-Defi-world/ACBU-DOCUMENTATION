# ACBU Backend – environment variables

Copy these into `backend/.env`. No mock data; all values must be real or explicitly empty.

The app uses **Prisma Accelerate** for the primary database (managed PostgreSQL; not a local PostgreSQL instance) and **MongoDB Atlas** for cache (managed; not local MongoDB).

**Prisma (single Accelerate URL):** You use one `prisma+postgres://accelerate.prisma-data.net/?api_key=...` link for both `DATABASE_URL` and `PRISMA_ACCELERATE_URL`. Set both to that same value in `backend/.env`.

**Important:** Prisma CLI loads **only** `backend/.env` (not `.env.local`). If you use `backend/.env.local`, either add `DATABASE_URL` (and `PRISMA_ACCELERATE_URL`) to `backend/.env` as well, or copy `.env.local` to `.env`. Otherwise `prisma migrate` / `prisma db push` will report "Environment variable not found: DATABASE_URL".

## Required (app fails without them)

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | Prisma Accelerate URL (`prisma+postgres://accelerate.prisma-data.net/?api_key=...`). Used for schema, migrate, db push, studio. Same value as `PRISMA_ACCELERATE_URL` when using one link. |
| `MONGODB_URI` | **MongoDB Atlas** connection string, e.g. `mongodb+srv://user:password@cluster.mongodb.net/acbu?retryWrites=true&w=majority` |
| `RABBITMQ_URL` | RabbitMQ URL, e.g. `amqp://acbu:acbu_password@localhost:5672` |
| `JWT_SECRET` | Secret for JWT signing (min 32 chars) |

## Database: Prisma Accelerate

| Variable | Description |
|----------|-------------|
| `PRISMA_ACCELERATE_URL` | Same as `DATABASE_URL` when using one link. **Runtime** connection via Accelerate adapter. Required in production. |
| `DATABASE_URL` | Same Accelerate URL. Used by Prisma schema and CLI (migrate, db push, studio). |

## Server

| Variable | Default | Description |
|----------|---------|-------------|
| `NODE_ENV` | `development` | `development` \| `test` \| `production` |
| `PORT` | `3000` | HTTP port |
| `API_VERSION` | `v1` | API path prefix |

## Fintech: Flutterwave

| Variable | Default | Description |
|----------|---------|-------------|
| `FLUTTERWAVE_PUBLIC_KEY` | — | Flutterwave public key |
| `FLUTTERWAVE_SECRET_KEY` | — | Flutterwave secret key |
| `FLUTTERWAVE_ENCRYPTION_KEY` | — | Flutterwave encryption key |
| `FLUTTERWAVE_BASE_URL` | `https://api.flutterwave.com/v3` | Flutterwave API base |

## Fintech: Paystack (NGN)

| Variable | Default | Description |
|----------|---------|-------------|
| `PAYSTACK_SECRET_KEY` | — | Paystack secret key |
| `PAYSTACK_BASE_URL` | `https://api.paystack.co` | Paystack API base |

## Fintech: MTN Mobile Money (RWF, etc.)

| Variable | Default | Description |
|----------|---------|-------------|
| `MTN_MOMO_SUBSCRIPTION_KEY` | — | MTN MoMo API subscription key |
| `MTN_MOMO_API_USER_ID` | — | API user id from MTN MoMo provisioning |
| `MTN_MOMO_API_KEY` | — | API key from MTN MoMo provisioning |
| `MTN_MOMO_BASE_URL` | sandbox/production URL by env | Override if needed |
| `MTN_MOMO_TARGET_ENVIRONMENT` | `sandbox` | `sandbox` \| `production` |

## Fintech: currency → provider (optional)

| Variable | Description |
|----------|-------------|
| `FINTECH_CURRENCY_PROVIDERS` | JSON `{"NGN":"paystack","RWF":"mtn_momo",...}` or CSV `NGN=paystack,RWF=mtn_momo,KES=flutterwave`. If unset, defaults: NGN→paystack, RWF→mtn_momo, others→flutterwave. |

## Stellar / Soroban

| Variable | Default | Description |
|----------|---------|-------------|
| `STELLAR_NETWORK` | `testnet` | `testnet` \| `mainnet` |
| `STELLAR_HORIZON_URL` | horizon-testnet URL | Horizon base URL |
| `STELLAR_SECRET_KEY` | — | Secret key for contract txs |
| `STELLAR_BASE_FEE_STROOPS` | `100` | Base transaction fee in stroops (1 stroop = 0.0000001 XLM). Used as the static fee when dynamic fees are disabled, and as the fallback when a Horizon fee fetch fails. |
| `STELLAR_USE_DYNAMIC_FEES` | `false` | Set to `true` to fetch the current recommended base fee from Horizon before each transaction. Automatically falls back to `STELLAR_BASE_FEE_STROOPS` if the Horizon request fails. Recommended for mainnet deployments under variable network load. |
| `WALLET_ACTIVATION_XLM` | `1` | XLM sent to a new Stellar address when the wallet-activation job runs. Same as `STELLAR_MIN_BALANCE` if set. Pi Network bridge variables were removed; activation is Stellar-only. |

## USDC→XLM swap (usdcConvertAndMintJob)

| Variable | Default | Description |
|----------|---------|-------------|
| `USDC_ISSUER_TESTNET` | `GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5` | Circle USDC issuer on Stellar testnet. Default is the well-known Circle testnet address; override with your issuer when using a stand-in asset (e.g. DemoUSDC). Circle testnet USDC is optional. |
| `USDC_ISSUER_MAINNET` | `GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN` | Circle USDC issuer on Stellar mainnet. |
| `USDC_ASSET_CODE_TESTNET` | `USDC` | Stellar asset code (4–12 alphanumeric) for the swap asset on testnet. Set to your issued code (e.g. `DemoUSDC`) when not using Circle testnet USDC. Must match the asset users hold and the Soroban minting contract’s configured USDC token (see below). |
| `USDC_ASSET_CODE_MAINNET` | `USDC` | Same as testnet, for mainnet. |
| `USDC_XLM_SLIPPAGE_BPS` | `50` | Slippage tolerance for the USDC→XLM `pathPaymentStrictSend` DEX swap, in basis points (50 = 0.5%). The backend queries Horizon for the expected XLM output and rejects the swap if the DEX delivers fewer XLM than `expected × (1 − slippage)`. |

**Testnet DemoUSDC (or other stand-in):** set `USDC_ISSUER_TESTNET` to your issuer and `USDC_ASSET_CODE_TESTNET` to the exact on-chain code (e.g. `DemoUSDC`). Ensure Horizon can find a **path from that asset → XLM** (liquidity pool or offers); otherwise `strictSendPaths` fails the same as with an illiquid USDC. **Soroban minting:** (re)deploy or re-`initialize` the minting contract so its stored USDC token address is the **Stellar asset contract (SAC)** for that issuer+code—not Circle’s testnet SAC—so `mint_from_usdc` matches what the backend swaps.

## Stellar contract IDs (after deploy)

| Variable | Description |
|----------|-------------|
| `CONTRACT_ORACLE` | Oracle contract id |
| `CONTRACT_RESERVE_TRACKER` | Reserve tracker contract id |
| `CONTRACT_MINTING` | Minting contract id |
| `CONTRACT_BURNING` | Burning contract id |

Or per network: `CONTRACT_ORACLE_TESTNET`, `CONTRACT_ORACLE_MAINNET`, etc.

**Custodial demo-fiat MVP:** demo basket tokens must be custodied on the **minting contract**; the backend calls `mint_from_demo_fiat` and `admin_drip_demo_fiat`. After deploy, follow [TESTNET_CUSTODIAL_BOOTSTRAP.md](./TESTNET_CUSTODIAL_BOOTSTRAP.md) to seed oracle rates, `s_token` map, reserves, and mint demo supply to the minter contract id.

## Oracle (40/40/20: central bank, fintech, forex)

| Variable | Default | Description |
|----------|---------|-------------|
| `ORACLE_UPDATE_INTERVAL_HOURS` | `6` | Hours between oracle updates |
| `ORACLE_EMERGENCY_THRESHOLD` | `0.05` | Emergency deviation threshold |
| `ORACLE_MAX_DEVIATION_PER_UPDATE` | `0.05` | Max 5% change per update; log warning if exceeded |
| `ORACLE_CIRCUIT_BREAKER_THRESHOLD` | `0.10` | If currency moves >10%, use 48h average instead |
| `EXCHANGERATE_API_BASE_URL` | `https://v6.exchangerate-api.com/v6` | Forex layer (Layer 3) base URL |
| `EXCHANGERATE_API_KEY` | — | API key for ExchangeRate-API (Layer 3); if unset, forex layer skipped |
| `CURRENCY_CENTRAL_BANK_URLS` | `{}` | JSON map of currency to central bank API URL (e.g. `{"NGN":"https://..."}`) for Layer 1 |

## Basket metrics (proposed weights job)

| Variable | Default | Description |
|----------|---------|-------------|
| `BASKET_METRICS_INTERVAL_DAYS` | `30` | Days between metrics ingestion and proposed basket weight creation |

## Limits (Deposit/Withdrawal & Circuit Breakers)

| Variable | Default (USD) | Description |
|----------|---------|-------------|
| `LIMIT_RETAIL_DEPOSIT_DAILY_USD` | `5000` | Daily deposit limit for retail users |
| `LIMIT_RETAIL_DEPOSIT_MONTHLY_USD` | `50000` | Monthly deposit limit for retail users |
| `LIMIT_RETAIL_WITHDRAWAL_DAILY_USD` | `10000` | Daily withdrawal limit for retail users |
| `LIMIT_RETAIL_WITHDRAWAL_MONTHLY_USD` | `80000` | Monthly withdrawal limit for retail users |
| `LIMIT_BUSINESS_DEPOSIT_DAILY_USD` | `50000` | Daily deposit limit for business users |
| `LIMIT_BUSINESS_DEPOSIT_MONTHLY_USD` | `500000` | Monthly deposit limit for business users |
| `LIMIT_BUSINESS_WITHDRAWAL_DAILY_USD` | `100000` | Daily withdrawal limit for business users |
| `LIMIT_BUSINESS_WITHDRAWAL_MONTHLY_USD` | `800000` | Monthly withdrawal limit for business users |
| `LIMIT_GOV_DEPOSIT_DAILY_USD` | `500000` | Daily deposit limit for government users |
| `LIMIT_GOV_DEPOSIT_MONTHLY_USD` | `5000000` | Monthly deposit limit for government users |
| `LIMIT_GOV_WITHDRAWAL_DAILY_USD` | `500000` | Daily withdrawal limit for government users |
| `LIMIT_GOV_WITHDRAWAL_MONTHLY_USD` | `4000000` | Monthly withdrawal limit for government users |
| `LIMIT_CIRCUIT_BREAKER_RESERVE_WEIGHT_PCT`| `10` | Reserve % weight threshold circuit breaker |
| `LIMIT_CIRCUIT_BREAKER_MIN_RATIO` | `1.02` | Minimum reserve ratio threshold circuit breaker |

## Reserve

| Variable | Default | Description |
|----------|---------|-------------|
| `RESERVE_MIN_RATIO` | `1.02` | Min overcollateralization (102%) |
| `RESERVE_TARGET_RATIO` | `1.05` | Target ratio (105%) |
| `RESERVE_ALERT_THRESHOLD` | `1.02` | Alert when below this |

## Notifications (email / SMS)

| Variable | Default | Description |
|----------|---------|-------------|
| `NOTIFICATION_EMAIL_PROVIDER` | `log` | `sendgrid` \| `ses` \| `log` |
| `NOTIFICATION_FROM_EMAIL` | `noreply@acbu.example.com` | Verified sender email for SendGrid/SES |
| `SENDGRID_API_KEY` | — | SendGrid API key when email provider is sendgrid |
| `NOTIFICATION_SMS_PROVIDER` | `log` | `twilio` \| `africas_talking` \| `log` |
| `TWILIO_ACCOUNT_SID` | — | Twilio account SID |
| `TWILIO_AUTH_TOKEN` | — | Twilio auth token |
| `TWILIO_FROM_NUMBER` | — | Twilio from number for SMS |
| `AFRICAS_TALKING_API_KEY` | — | Africas Talking API key |
| `AFRICAS_TALKING_USERNAME` | — | Africas Talking username |
| `NOTIFICATION_ALERT_EMAIL` | — | Email for reserve/ops alerts when no user context |

## Outbound webhooks

| Variable | Default | Description |
|----------|---------|-------------|
| `WEBHOOK_URL` | — | URL to POST outbound webhook payloads (HMAC-SHA256 signed) |
| `WEBHOOK_SECRET` | — | Secret for HMAC-SHA256 signature |

## Monitoring

| Variable | Default | Description |
|----------|---------|-------------|
| `SENTRY_DSN` | — | Sentry DSN for error tracking (integrate in app when set) |
| `SENTRY_ENVIRONMENT` | — | Sentry environment (e.g. production, staging) |

## Other

| Variable | Default | Description |
|----------|---------|-------------|
| `API_KEY_SALT` | — | Salt for API key hashing |
| `JWT_EXPIRES_IN` | `7d` | JWT expiry |
| `RATE_LIMIT_WINDOW_MS` | `60000` | Rate limit window (ms) |
| `RATE_LIMIT_MAX_REQUESTS` | `100` | Max requests per window |
| `LOG_LEVEL` | `info` | Log level |
| `LOG_FILE` | `logs/app.log` | Log file path |
| `CORS_ORIGIN` | `http://localhost:3000` | Allowed origins (comma-separated) |
